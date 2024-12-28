# Type Calculator Extension Point

The `typeCalculator` extension point provides plugin developers with the ability to influence and customize the type inference mechanism for Groovy expressions within the IDE. By implementing this extension point, plugins can contribute custom type calculation logic for specific Groovy constructs, enhancing the IDE’s static analysis capabilities.

---

## Extension Point Declaration

```xml
<extensionPoint name="typeCalculator"
                dynamic="true"
                beanClass="com.intellij.openapi.util.ClassExtensionPoint">
  <with attribute="implementationClass" implements="org.jetbrains.plugins.groovy.lang.typing.GrTypeCalculator"/>
</extensionPoint>
```

---

## Purpose and Capabilities

### **1. Custom Type Inference**
Groovy’s dynamic nature makes it challenging for static analysis tools to infer types accurately. The `typeCalculator` extension point enables plugin developers to implement custom logic for specific constructs, improving the IDE’s understanding of:
- Custom DSLs
- Dynamic method calls
- Framework-specific behavior

### **2. Resolving Ambiguities**
Default type inference mechanisms may struggle with ambiguous or complex scenarios. Custom type calculators can:
- Refine type calculations for particular Groovy expressions.
- Handle scenarios where default inference falls short.

### **3. Seamless Integration with Existing Infrastructure**
Type calculators operate alongside default inference mechanisms. The custom logic is executed first, and if it returns `null`, the default implementation takes over. This ensures backward compatibility and minimizes disruption.

---

## Key Interface

### `GrTypeCalculator`
This interface defines the contract for type calculators. Implementations register themselves to handle specific expression types and provide custom type inference logic.

```java
public interface GrTypeCalculator<T extends GrExpression> {

  ClassExtension<GrTypeCalculator<?>> EP = new ClassExtension<>("org.intellij.groovy.typeCalculator");

  @Nullable
  PsiType getType(@NotNull T expression);

  @SuppressWarnings("unchecked")
  static @Nullable PsiType getTypeFromCalculators(@NotNull GrExpression expression) {
    for (GrTypeCalculator<?> calculator : EP.forKey(expression.getClass())) {
      PsiType type = ((GrTypeCalculator<GrExpression>)calculator).getType(expression);
      if (type != null) return type;
    }
    return null;
  }
}
```

#### Key Methods:
- **`getType(T expression)`**:
  Computes the type of the given expression. If the calculator cannot determine the type, it returns `null`.
- **`getTypeFromCalculators(GrExpression expression)`**:
  Iterates over all registered calculators for the expression’s class, returning the first non-`null` type.

---

## Example Implementations

### 1. **Gradle Task Declaration Type Calculator**
Handles type inference for Gradle-specific task declarations.

```kotlin
class GradleTaskDeclarationTypeCalculator : GrTypeCalculator<GrReferenceExpression> {

  override fun getType(expression: GrReferenceExpression): PsiType? = when {
    isTaskIdExpression(expression) -> createType(JAVA_LANG_STRING, expression.containingFile)
    isTaskIdInOperator(expression) -> createType(GRADLE_API_TASK, expression.containingFile)
    else -> null
  }
}
```

#### Capabilities:
- **Identifies Gradle Task IDs**:
  Returns `String` type for task identifiers.
- **Handles `in` Operator**:
  Infers type as `Task` when using Gradle’s `in` operator.

---

### 2. **Default Method Reference Type Calculator**
Handles type inference for method references.

```kotlin
class DefaultMethodReferenceTypeCalculator : GrTypeCalculator<GrReferenceExpression> {

  override fun getType(expression: GrReferenceExpression): PsiType? {
    if (!expression.hasMemberPointer()) {
      return null
    }
    return GroovyMethodReferenceType(expression)
  }
}
```

#### Capabilities:
- **Method References**:
  Resolves method references to their appropriate types.
- **Fallback Logic**:
  Returns `null` if the expression does not involve a method reference.

---

### 3. **Closure Delegate Type Calculator**
Infers types for Groovy closure delegate or owner references.

```kotlin
class GrClosureOwnerDelegateTypeCalculator : GrTypeCalculator<GrReferenceExpression> {

  override fun getType(expression: GrReferenceExpression): PsiType? {
    val method = expression.rValueReference?.resolve() as? PsiMethod ?: return null

    val methodName = method.name
    val delegate = "getDelegate" == methodName
    if (!delegate && "getOwner" != methodName) return null

    if (method.parameterList.parametersCount != 0) return null

    val closureClass = JavaPsiFacade.getInstance(expression.project)
      .findClass(GROOVY_LANG_CLOSURE, expression.resolveScope)
    if (closureClass == null || closureClass != method.containingClass) return null

    val functionalExpression = PsiTreeUtil.getParentOfType(expression, GrFunctionalExpression::class.java) ?: return null
    return if (delegate) getDelegatesToInfo(functionalExpression)?.typeToDelegate else functionalExpression.ownerType
  }
}
```

#### Capabilities:
- **Handles `getDelegate` and `getOwner`**:
  Returns the type of a closure’s delegate or owner based on the method being resolved.
- **Closure Context Awareness**:
  Examines the closure’s parent context to infer the correct type.

---

## Use Cases and Benefits

### **1. Framework-Specific Enhancements**
- **Gradle**:
  Improves type inference for Gradle DSL constructs, such as task declarations and configurations.
- **Grails**:
  Handles types for dynamic methods and properties in Grails domain classes.

### **2. DSL Support**
Provides accurate type inference for custom DSLs, enabling better code completion, navigation, and validation.

### **3. Resolving Ambiguities**
- Handles edge cases where default type inference fails.
- Ensures smoother developer experience by reducing false positives and negatives in type-related inspections.

---

## Conclusion

The `typeCalculator` extension point is an essential tool for enhancing Groovy’s type inference capabilities within the IDE. By allowing custom logic for specific expressions, it bridges the gap between Groovy’s dynamic nature and the needs of static analysis, making the development process more intuitive and efficient.

