# Groovy Named Argument Provider Extension Point

The `namedArgumentProvider` extension point allows plugin developers to enhance Groovy method calls by providing named arguments. Named arguments are arguments passed to methods in the form `methodName(arg1: value1, arg2: value2)`. This feature is particularly useful for supporting frameworks or libraries that rely on specific named parameters.

---

## Extension Point Declaration

```xml
<extensionPoint name="namedArgumentProvider"
                dynamic="true"
                interface="org.jetbrains.plugins.groovy.extensions.GroovyNamedArgumentProvider"/>
```

---

## Purpose

- **Enhanced Code Completion**: Add meaningful named argument suggestions in Groovy DSLs and libraries.
- **Dynamic Argument Validation**: Provide runtime or compile-time validation for named arguments.
- **Improved Developer Experience**: Help developers use APIs more effectively by surfacing valid named parameters.

---

## Key Interfaces and Classes

### 1. `GroovyNamedArgumentProvider`

This is the main interface for contributing named arguments to method calls.

```java
public abstract class GroovyNamedArgumentProvider {

  public static final ExtensionPointName<GroovyNamedArgumentProvider> EP_NAME =
    ExtensionPointName.create("org.intellij.groovy.namedArgumentProvider");

  /**
   * Computes named arguments for a call.
   * @param call The method call expression.
   * @param resolveResult The resolved target method or element.
   * @param argumentName The name of the argument to be resolved (optional).
   * @param forCompletion Whether the call is for code completion.
   * @param result A map to accumulate the results.
   */
  public void getNamedArguments(@NotNull GrCall call,
                                @NotNull GroovyResolveResult resolveResult,
                                @Nullable String argumentName,
                                boolean forCompletion,
                                @NotNull Map<String, NamedArgumentDescriptor> result) {
  }

  /**
   * Provides named arguments for Groovy literal maps.
   * @param literal The Groovy map literal.
   * @return A map of argument names to their descriptors.
   */
  public @NotNull Map<String, NamedArgumentDescriptor> getNamedArguments(@NotNull GrListOrMap literal) {
    return Collections.emptyMap();
  }
}
```

### 2. `NamedArgumentDescriptor`

Represents metadata for a named argument, including its type, navigation target, and documentation.

---

## Example Implementations

### 1. **Source Code-Based Named Arguments**

This implementation provides named arguments based on source code definitions.

```java
public final class GroovySourceCodeNamedArgumentProvider extends GroovyNamedArgumentProvider {

  @Override
  public void getNamedArguments(@NotNull GrCall call,
                                @NotNull GroovyResolveResult resolveResult,
                                @Nullable String argumentName,
                                boolean forCompletion,
                                @NotNull Map<String, NamedArgumentDescriptor> result) {
    PsiElement resolve = resolveResult.getElement();
    if (resolve instanceof GrMethod) {
      ((GrMethod)resolve).getNamedParameters().forEach(result::putIfAbsent);
    } else if (resolve instanceof GrField) {
      ((GrField)resolve).getNamedParameters().forEach(result::putIfAbsent);
    }
  }
}
```

### 2. **Named Variant Arguments**

This implementation supports named arguments for annotated methods using `@NamedParams` or similar annotations.

```kotlin
class GroovyNamedVariantArgumentProvider : GroovyNamedArgumentProvider() {
  override fun getNamedArguments(call: GrCall,
                                 resolveResult: GroovyResolveResult,
                                 argumentName: String?,
                                 forCompletion: Boolean,
                                 result: MutableMap<String, NamedArgumentDescriptor>) {
    val method = resolveResult.element as? PsiMethod ?: return

    val parameters = method.parameterList.parameters
    val mapParameter = parameters.firstOrNull() ?: return

    collectNamedParams(mapParameter).forEach {
      val type = it.type
      result[it.name] = if (type != null) TypeCondition(type, it.navigationElement) else NamedArgumentDescriptorImpl(it.navigationElement)
    }
  }
}
```

### 3. **SwingBuilder Support**

This implementation provides named arguments for SwingBuilder DSL.

```java
public class SwingBuilderNamedArgumentProvider extends GroovyNamedArgumentProvider {

  @Override
  public void getNamedArguments(@NotNull GrCall call,
                                @NotNull GroovyResolveResult resolveResult,
                                @Nullable String argumentName,
                                boolean forCompletion,
                                @NotNull Map<String, NamedArgumentDescriptor> result) {
    PsiElement resolve = resolveResult.getElement();
    PsiType returnType = resolve == null ? null : ((PsiMethod)resolve).getReturnType();
    PsiClass aClass = PsiTypesUtil.getPsiClass(returnType);
    if (aClass == null) return;

    for (PsiMethod method : aClass.getAllMethods()) {
      String propertyName = GroovyPropertyUtils.getPropertyNameBySetterName(method.getName());
      if (propertyName != null && (argumentName == null || argumentName.equals(propertyName))) {
        result.put(propertyName, NamedArgumentDescriptor.SIMPLE_ON_TOP);
      }
    }
  }
}
```

---

## Use Cases

### 1. DSL Frameworks
Named arguments are common in DSLs like SwingBuilder, Gant, or Gradle. This extension point helps provide meaningful argument suggestions in such cases.

### 2. Enhanced IntelliJ Completion
Integrating this extension ensures users see only valid named arguments for methods, improving the completion experience.

### 3. Validation of Arguments
Plugin developers can validate named arguments dynamically, catching errors at design time.

---

## Registration

To register a `GroovyNamedArgumentProvider`, add the following to your `plugin.xml`:

```xml
<namedArgumentProvider implementation="com.example.MyNamedArgumentProvider"/>
```

---

## Conclusion

The `namedArgumentProvider` extension point enables plugin developers to enhance the Groovy editing experience by dynamically contributing named arguments. By leveraging this feature, plugins can support domain-specific languages, improve code completion, and validate arguments effectively.

