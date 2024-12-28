# DelegatesTo Provider Extension Point

The `delegatesToProvider` extension point in the IntelliJ IDEA platform allows developers to customize and enhance how Groovy closures resolve delegate types. This mechanism is particularly useful in DSL (Domain-Specific Language) contexts, enabling developers to provide precise typing information and improve code insight, navigation, and auto-completion in Groovy-based DSLs.

---

## Extension Point Declaration

```xml
<extensionPoint name="delegatesToProvider" dynamic="true"
                interface="org.jetbrains.plugins.groovy.lang.resolve.delegatesTo.GrDelegatesToProvider"/>
```

---

## Interface: `GrDelegatesToProvider`

The `GrDelegatesToProvider` interface defines a contract for determining the delegate type of a Groovy closure dynamically. Implementations of this interface can register custom logic for specific Groovy closures.

### Interface Definition

```java
public interface GrDelegatesToProvider {

  ExtensionPointName<GrDelegatesToProvider> EP_NAME = ExtensionPointName.create("org.intellij.groovy.delegatesToProvider");

  @Nullable
  DelegatesToInfo getDelegatesToInfo(@NotNull GrFunctionalExpression expression);
}
```

### Key Method
- **`getDelegatesToInfo`**:
  - Takes a `GrFunctionalExpression` (Groovy closure) as input.
  - Returns a `DelegatesToInfo` object, encapsulating the delegate type and strategy.
  - Returns `null` if the closure is not supported by the implementation.

---

## Example Implementations

### 1. **DefaultDelegatesToProvider**

This implementation provides delegate type resolution for common Groovy idioms, such as `with` or `identity` methods.

#### Key Features
- Resolves delegate types for closures used with `with` or `identity` calls.
- Supports advanced argument mapping logic to infer delegate types dynamically.

#### Code Example

```kotlin
class DefaultDelegatesToProvider : GrDelegatesToProvider {

  override fun getDelegatesToInfo(expression: GrFunctionalExpression): DelegatesToInfo? {
    val call = getContainingCall(expression) ?: return null
    val result = call.advancedResolve()
    val method = result.element as? PsiMethod ?: return null

    if (GdkMethodUtil.isWithOrIdentity(method)) {
      val qualifier = inferCallQualifier(call as GrMethodCall) ?: return null
      return DelegatesToInfo(qualifier.type, Closure.DELEGATE_FIRST)
    }

    // Handle `@DelegatesTo` annotations
    val parameter = ... // Obtain parameter and annotations logic
    return DelegatesToInfo(delegateType, strategyValue)
  }
}
```

#### DSL Use Cases
- Provides delegate types for Groovy's `with` blocks, improving type inference and code completion.
- Extends support for Groovy's `@DelegatesTo` annotation.

---

### 2. **GebBrowserDelegatesToProvider**

This implementation integrates with the Geb framework, commonly used for browser automation in Groovy.

#### Key Features
- Resolves delegate types for closures passed to Geb's `Browser.drive` method.
- Uses pattern matching to detect relevant closures.

#### Code Example

```kotlin
class GebBrowserDelegatesToProvider : GrDelegatesToProvider {

  override fun getDelegatesToInfo(expression: GrFunctionalExpression): DelegatesToInfo? {
    return if (pattern.accepts(expression)) {
      DelegatesToInfo(TypesUtil.createType("geb.Browser", expression))
    } else {
      null
    }
  }
}
```

#### DSL Use Cases
- Enables precise type inference for closures in Geb's `Browser.drive` DSL, improving navigation and auto-completion.

---

### 3. **LogbackDelegatesToProvider**

This implementation adds support for Logback configuration DSLs, helping developers configure logging behavior.

#### Key Features
- Supports closures in Logback's configuration methods like `appender`, `receiver`, and `turboFilter`.
- Provides context-specific delegate types for each closure.

#### Code Example

```kotlin
class LogbackDelegatesToProvider : GrDelegatesToProvider {

  override fun getDelegatesToInfo(expression: GrFunctionalExpression): DelegatesToInfo? {
    if (appendClosure.accepts(expression)) {
      return DelegatesToInfo(TypesUtil.createType("ch.qos.logback.classic.gaffer.AppenderDelegate", expression), Closure.DELEGATE_FIRST)
    }
    return null
  }
}
```

#### DSL Use Cases
- Enhances type inference for Logback's Groovy DSL, ensuring better navigation and code assistance when configuring loggers.

---

## General Capabilities Added by Implementations

1. **Dynamic Delegate Type Resolution**
   - Dynamically determines delegate types for closures based on the surrounding context.
   - Supports both built-in Groovy methods and custom DSL constructs.

2. **Integration with Annotations**
   - Leverages Groovy's `@DelegatesTo` and `@DelegatesTo.Target` annotations to infer delegate types.

3. **Pattern-Based Matching**
   - Supports fine-grained closure matching using IntelliJ's pattern-based matching utilities.

4. **Improved Developer Experience**
   - Enhances code completion and type inference for Groovy DSLs.
   - Simplifies navigation and refactoring in complex Groovy projects.

---

## Conclusion

The `delegatesToProvider` extension point empowers developers to enrich Groovy DSLs and frameworks with precise type inference and enhanced developer tooling. By leveraging this extension point, plugin authors can ensure that Groovy closures are contextually aware, improving the overall coding experience for IntelliJ IDEA users. From browser automation with Geb to logging configuration with Logback, the possibilities for enhancing Groovy DSLs are vast and impactful.

