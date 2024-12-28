# Closure Completer Extension Point

The `closureCompleter` extension point allows plugin developers to dynamically define and enhance the auto-completion of Groovy closures. This enables developers to enrich Groovy scripting and DSLs with customized completion capabilities, ensuring users have an intuitive and contextually aware coding experience.

---

## Extension Point Declaration

```xml
<extensionPoint name="closureCompleter" dynamic="true" interface="org.jetbrains.plugins.groovy.lang.completion.ClosureCompleter"/>
```

---

## Key Interface: `ClosureCompleter`

The `ClosureCompleter` interface defines the contract for implementing closure completion functionality. Implementations specify parameter names, types, and behaviors of Groovy closures in various contexts.

### Interface Definition

```java
public abstract class ClosureCompleter {
  private static final ExtensionPointName<ClosureCompleter> EP_NAME = ExtensionPointName.create("org.intellij.groovy.closureCompleter");

  protected abstract @Nullable List<ClosureParameterInfo> getParameterInfos(
    InsertionContext context,
    PsiMethod method,
    PsiSubstitutor substitutor,
    PsiElement place
  );

  public static boolean runClosureCompletion(
    InsertionContext context,
    PsiMethod method,
    PsiSubstitutor substitutor,
    Document document,
    int offset,
    PsiElement parent
  ) {
    for (ClosureCompleter completer : EP_NAME.getExtensions()) {
      final List<ClosureParameterInfo> parameterInfos = completer.getParameterInfos(context, method, substitutor, parent);
      if (parameterInfos != null) {
        runClosureTemplate(context, document, offset, substitutor, method, parameterInfos);
        return true;
      }
    }
    return false;
  }
}
```

---

## Example Implementations

### 1. **`EachWithIndexClosureCompleter`**

This implementation supports Groovy’s `eachWithIndex` method, enhancing completion for closure parameters.

#### Capabilities
- **Parameter Types:** Infers the types of closure parameters (`entry` and `index`) based on the collection being iterated.
- **Dynamic Context Handling:** Determines parameter types from the method signature and context.

#### Code Example

```java
public final class EachWithIndexClosureCompleter extends ClosureCompleter {

  @Override
  protected @Nullable List<ClosureParameterInfo> getParameterInfos(
    InsertionContext context,
    PsiMethod method,
    PsiSubstitutor substitutor,
    PsiElement place
  ) {
    if (!"eachWithIndex".equals(method.getName())) return null;

    PsiParameter[] parameters = method.getParameterList().getParameters();
    PsiType collectionType = substitutor.substitute(parameters[0].getType());
    PsiType iteratedType = PsiUtil.extractIterableTypeParameter(collectionType, true);

    if (iteratedType != null) {
      return List.of(
        new ClosureParameterInfo(iteratedType.getCanonicalText(), "entry"),
        new ClosureParameterInfo("int", "i")
      );
    }

    return null;
  }
}
```

#### DSL Use Cases
- **Iterating Collections:** Provides intuitive closure parameter suggestions when using methods like `eachWithIndex`.
- **Custom Iterators:** Can be extended to handle custom collection-like objects or data structures.

---

### 2. **`GdslClosureCompleter`**

Enhances Groovy DSLs by adding context-aware completion for closures based on GDSL descriptors.

#### Capabilities
- **DSL Awareness:** Leverages GDSL descriptors to infer parameter types and names dynamically.
- **Contextual Completion:** Determines applicable methods and parameters based on the current editing context.

#### Code Example

```java
public final class GdslClosureCompleter extends ClosureCompleter {

  @Override
  protected List<ClosureParameterInfo> getParameterInfos(
    InsertionContext context,
    PsiMethod method,
    PsiSubstitutor substitutor,
    PsiElement place
  ) {
    GrReferenceExpression ref = (GrReferenceExpression) place;
    PsiType qualifierType = PsiImplUtil.getQualifierType(ref);
    if (qualifierType == null) return null;

    List<ClosureDescriptor> descriptors = GroovyDslFileIndex.getDescriptors(qualifierType, ref);

    for (ClosureDescriptor descriptor : descriptors) {
      if (isMethodApplicable(descriptor, method)) {
        return descriptor.getParameters().stream()
          .map(param -> new ClosureParameterInfo(param.getType(), param.getName()))
          .toList();
      }
    }

    return null;
  }
}
```

#### DSL Use Cases
- **Custom DSL Constructs:** Adds method-specific closure parameters for domain-specific constructs.
- **Enhanced GDSL Support:** Complements GDSL extensions by providing runtime-aware parameter suggestions.

---

### 3. **General Capabilities Added by Implementations**

#### **Dynamic Closure Templates**
- Auto-inserts closure templates with parameter placeholders.
- Supports both simple and complex closures, enabling rapid prototyping.

#### **Integration with IDE Features**
- Provides real-time parameter suggestions.
- Integrates with IntelliJ’s refactoring and formatting tools to maintain code consistency.

#### **Parameter Inference**
- Automatically infers parameter types from:
  - Method signatures.
  - Contextual information (e.g., collections, maps, or DSL descriptors).

---

## Conclusion

The `closureCompleter` extension point is a powerful tool for enriching Groovy DSLs and scripting workflows. By enabling dynamic and context-aware closure completion, it enhances the developer experience, reduces errors, and accelerates the development of domain-specific applications. From iterating collections to supporting custom DSLs, this extension point opens the door to creating intelligent and intuitive coding environments tailored to the needs of Groovy developers.

