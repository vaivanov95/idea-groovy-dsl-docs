# Named Argument Provider Extension Point

The `namedArgumentProvider` extension point enables plugin developers to add support for named arguments in Groovy method calls. Named arguments are key-value pairs passed as method arguments, such as `foo(bar: 42)`. This extension point is crucial for enhancing code completion, validation, and navigation in DSLs and APIs that heavily utilize named arguments.

---

## Extension Point Declaration

```xml
<extensionPoint name="namedArgumentProvider" 
                dynamic="true" 
                interface="org.jetbrains.plugins.groovy.extensions.GroovyNamedArgumentProvider"/>
```

---

## Purpose and Capabilities

### **1. Enabling Named Argument Support**
- **Code Completion**: By implementing this extension point, plugins can provide context-aware completion for named arguments.
- **Type Validation**: Ensure that the provided named arguments adhere to the expected types.
- **Navigation and Documentation**: Developers can navigate to the definitions of named arguments and view their associated documentation.

### **2. Context-Sensitive Argument Resolution**
The extension point allows computing named arguments dynamically, depending on the method or constructor being called. This is particularly useful when a method overload accepts different sets of named arguments.

### **3. Integration with Custom DSLs**
Many Groovy-based DSLs rely on named arguments for configuration. Examples include:
- **Gradle Build Scripts**: Configure tasks using named arguments.
- **SwingBuilder**: Define UI components with property-style named arguments.

---

## Key Interface and Methods

### `GroovyNamedArgumentProvider`
This abstract class defines methods to compute named arguments for a call or a map literal.

```java
public abstract class GroovyNamedArgumentProvider {

  public void getNamedArguments(@NotNull GrCall call,
                                @NotNull GroovyResolveResult resolveResult,
                                @Nullable String argumentName,
                                boolean forCompletion,
                                @NotNull Map<String, NamedArgumentDescriptor> result) {
  }

  public @NotNull Map<String, NamedArgumentDescriptor> getNamedArguments(@NotNull GrListOrMap literal) {
    return Collections.emptyMap();
  }
}
```

#### Key Methods:
- **`getNamedArguments(GrCall)`**: Computes named arguments for a method call.
- **`getNamedArguments(GrListOrMap)`**: Computes named arguments for a map literal.

---

## Example Implementations and Their Capabilities

### **1. Source Code Named Argument Provider**

#### Implementation Overview:
This provider retrieves named arguments defined as parameters or fields in a Groovy class.

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
      ((GrMethod)resolve).getNamedParameters().forEach((key, value) -> result.putIfAbsent(key, value));
    }
  }
}
```

#### Capabilities:
- **Dynamic Resolution**: Dynamically retrieves named arguments defined in method signatures or fields.
- **Integration with Completion**: Enhances the IDE’s autocompletion capabilities by suggesting named arguments relevant to the method.
- **Validation**: Ensures that the named arguments are valid for the method being called.

### **2. SwingBuilder Named Argument Provider**

#### Implementation Overview:
This provider adds support for named arguments in SwingBuilder, which uses property-style arguments to configure UI components.

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
      if (propertyName != null) {
        result.put(propertyName, NamedArgumentDescriptor.SIMPLE_ON_TOP);
      }
    }
  }
}
```

#### Capabilities:
- **Domain-Specific Enhancements**: Focuses on the SwingBuilder DSL, enhancing the developer experience for UI definitions.
- **Optimized for Completion**: Provides quick suggestions for property-based named arguments.
- **Extends Functionality**: Supports dynamically added event listeners by analyzing method names and parameter types.

### **3. Gradle Named Argument Provider**

#### Discussion:
The Gradle DSL relies heavily on named arguments to define tasks and configurations. A custom implementation of this extension point can:
- Suggest task-specific named arguments.
- Validate argument types against Gradle API specifications.
- Improve navigation to argument definitions.

---

## Use Cases and Benefits

### **1. Enhancing DSLs**
Plugins can leverage this extension point to:
- Provide better support for custom DSLs built on Groovy.
- Enhance autocompletion, validation, and navigation for named arguments.

### **2. Improved Developer Productivity**
By dynamically resolving and validating named arguments, the extension point reduces errors and improves the usability of Groovy APIs.

### **3. Flexible and Extensible**
This extension point can adapt to any Groovy-based framework or library, making it a powerful tool for plugin developers targeting Groovy ecosystems.

---

## Conclusion

The `namedArgumentProvider` extension point is a cornerstone for supporting named arguments in Groovy. By enabling dynamic computation and validation of named arguments, it enhances IDE support for Groovy-based DSLs and APIs. From improving code completion to ensuring type safety, this extension point plays a critical role in creating a seamless development experience.

