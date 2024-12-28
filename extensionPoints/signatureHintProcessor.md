# Signature Hint Processor Extension Point

The `signatureHintProcessor` extension point in IntelliJ IDEA enables custom logic for inferring method signatures in Groovy-based development. This allows plugin developers to dynamically determine expected method parameter types and structures based on annotations and method metadata. Such capabilities are especially valuable when working with type inference in dynamic languages like Groovy.

---

## Extension Point Declaration

```xml
<extensionPoint name="signatureHintProcessor" dynamic="true"
                interface="org.jetbrains.plugins.groovy.lang.psi.typeEnhancers.SignatureHintProcessor"/>
```

This extension point is dynamic, allowing implementations to be loaded without restarting the IDE.

---

## Key Interface: `SignatureHintProcessor`

The `SignatureHintProcessor` interface defines how to analyze Groovy method signatures and infer expected parameter types based on provided hints.

### Interface Definition

```java
public abstract class SignatureHintProcessor {
  private static final ExtensionPointName<SignatureHintProcessor> EP_NAME =
      ExtensionPointName.create("org.intellij.groovy.signatureHintProcessor");

  public abstract String getHintName();

  public abstract @NotNull List<PsiType[]> inferExpectedSignatures(@NotNull PsiMethod method,
                                                                   @NotNull PsiSubstitutor substitutor,
                                                                   String @NotNull [] options);
}
```

### Core Responsibilities

- **Hint Identification:** Each processor identifies itself with a unique hint name via `getHintName()`.
- **Signature Inference:** The `inferExpectedSignatures` method analyzes method metadata and hint options to return potential parameter types.

### How It Works

1. Groovy annotations like `@groovy.transform.stc.*` provide hints.
2. The processor identifies the appropriate hint and processes it.
3. Expected parameter types are returned for code assistance and validation.

---

## Example Implementations

### 1. **`MapEntryOrKeyValueHintProcessor`**

This implementation processes the `@groovy.transform.stc.MapEntryOrKeyValue` hint, inferring signatures involving `Map.Entry` or key-value pairs.

#### Capabilities
- Infers types for methods dealing with `Map` entries or key-value structures.
- Supports additional options like argument position (`argNum`) and index inclusion.

#### Code Example

```java
public final class MapEntryOrKeyValueHintProcessor extends SignatureHintProcessor {

  @Override
  public String getHintName() {
    return "groovy.transform.stc.MapEntryOrKeyValue";
  }

  @Override
  public @NotNull List<PsiType[]> inferExpectedSignatures(@NotNull PsiMethod method,
                                                          @NotNull PsiSubstitutor substitutor,
                                                          String @NotNull [] options) {
    int argNum = extractArgNum(options);
    boolean index = extractIndex(options);

    PsiParameter[] parameters = method.getParameterList().getParameters();

    if (argNum >= parameters.length) return ContainerUtil.emptyList();

    PsiParameter parameter = parameters[argNum];
    PsiType type = substitutor.substitute(parameter.getType());

    if (!InheritanceUtil.isInheritor(type, CommonClassNames.JAVA_UTIL_MAP)) return ContainerUtil.emptyList();

    PsiType key = PsiUtil.substituteTypeParameter(type, CommonClassNames.JAVA_UTIL_MAP, 0, true);
    PsiType value = PsiUtil.substituteTypeParameter(type, CommonClassNames.JAVA_UTIL_MAP, 1, true);

    PsiClass mapEntry = JavaPsiFacade.getInstance(method.getProject()).findClass(CommonClassNames.JAVA_UTIL_MAP_ENTRY, method.getResolveScope());
    if (mapEntry == null) return ContainerUtil.emptyList();

    PsiClassType mapEntryType = JavaPsiFacade.getElementFactory(method.getProject()).createType(mapEntry, key, value);

    PsiType[] keyValueSignature = index ? new PsiType[]{key, value, PsiTypes.intType()} : new PsiType[]{key, value};
    PsiType[] mapEntrySignature = index ? new PsiType[]{mapEntryType, PsiTypes.intType()} : new PsiType[]{mapEntryType};

    return List.of(keyValueSignature, mapEntrySignature);
  }

  private static int extractArgNum(String[] options) {
    for (String value : options) {
      if (value.startsWith("argNum=")) {
        return Integer.parseInt(value.substring("argNum=".length()));
      }
    }
    return 0;
  }

  private static boolean extractIndex(String[] options) {
    for (String value : options) {
      if (value.startsWith("index=")) {
        return Boolean.parseBoolean(value.substring("index=".length()));
      }
    }
    return false;
  }
}
```

#### DSL Use Cases
- **Maps with Closures:** Simplifies handling `Map.Entry` types in Groovy closures.
- **Custom Map Iterations:** Supports custom DSL constructs requiring key-value or entry processing.

---

### 2. **`SimpleTypeHintProcessor`**

Processes the `@groovy.transform.stc.SimpleType` hint, providing straightforward type inference from string options.

#### Capabilities
- Directly converts string options to `PsiType` instances.
- Handles simple type declarations without complex metadata.

#### Code Example

```java
public final class SimpleTypeHintProcessor extends SignatureHintProcessor {

  @Override
  public String getHintName() {
    return "groovy.transform.stc.SimpleType";
  }

  @Override
  public @NotNull List<PsiType[]> inferExpectedSignatures(@NotNull PsiMethod method,
                                                          @NotNull PsiSubstitutor substitutor,
                                                          String @NotNull [] options) {
    return List.of(ContainerUtil.map(options, option -> {
      try {
        return JavaPsiFacade.getElementFactory(method.getProject()).createTypeFromText(option, method);
      } catch (IncorrectOperationException e) {
        return PsiTypes.nullType();
      }
    }, new PsiType[options.length]));
  }
}
```

#### DSL Use Cases
- **Simple Type Constraints:** Applies direct type restrictions to parameters in custom DSLs.
- **Annotation-Based Inference:** Facilitates lightweight annotations for parameter typing.

---

## Potential Use Cases

### 1. **Groovy DSLs**
Signature hint processors allow for domain-specific annotations to influence parameter types dynamically. For example, a custom `@DataModel` annotation could guide type inference for data-binding methods.

### 2. **Framework-Specific Enhancements**
Frameworks like Spock or Grails can use signature hint processors to simplify method signatures and improve code assistance.

### 3. **Enhanced Refactoring**
By providing accurate type hints, these processors improve refactoring capabilities, ensuring method calls are updated correctly.

---

## Conclusion

The `signatureHintProcessor` extension point provides a flexible mechanism to enhance type inference in Groovy. By implementing this extension point, plugin developers can create powerful tools to support dynamic method signatures, enabling better code completion, validation, and refactoring for Groovy DSLs and frameworks.

