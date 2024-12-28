# Call Type Calculator Extension Point

The `callTypeCalculator` extension point enables plugin developers to define custom logic for determining the return type of method calls in Groovy code. By extending this functionality, plugins can enhance type inference for dynamic and context-dependent method invocations.

---

## Extension Point Declaration

```xml
<extensionPoint name="callTypeCalculator"
                dynamic="true"
                interface="org.jetbrains.plugins.groovy.lang.typing.GrCallTypeCalculator"/>
```

---

## Purpose and Capabilities

### **1. Contextual Type Inference for Method Calls**
Groovy's dynamic nature often requires additional logic to infer the types of method calls accurately. This extension point allows developers to:

- Provide custom type inference for specific method calls.
- Handle context-sensitive methods whose return types depend on the receiver type or method arguments.

### **2. Improved Code Insight**
Plugins can enhance IDE features by using this extension point to provide more precise type information for method calls. This results in:

- Better code completion.
- Accurate highlighting and validation of method calls.
- Improved navigation and refactoring support.

---

## Key Interface: `GrCallTypeCalculator`

### Overview
The `GrCallTypeCalculator` interface defines a single method for calculating the return type of a method call:

```kotlin
@Experimental
interface GrCallTypeCalculator {
  fun getType(receiver: PsiType?,
              method: PsiMethod,
              arguments: Arguments?,
              context: PsiElement): PsiType?
}
```

### Parameters
- **`receiver`**: The type of the receiver (e.g., `object` in `object.method()`).
- **`method`**: The `PsiMethod` representing the method being invoked.
- **`arguments`**: The arguments passed to the method (can be `null` if unavailable).
- **`context`**: The context in which the method is being called.

### Return Value
- Returns the inferred type of the method call or `null` if no type can be determined.

---

## Example Implementations

### **1. Closure Methods Call Type Calculator**
This implementation determines the types of Groovy closure methods such as `curry`, `ncurry`, `rcurry`, and `memoize`:

```kotlin
class ClosureMethodsCallTypeCalculator : GrCallTypeCalculator {

  override fun getType(receiver: PsiType?, method: PsiMethod, arguments: Arguments?, context: PsiElement): PsiType? {
    if (receiver !is GroovyClosureType) {
      return null
    }
    val methodName = method.name
    if (methodName !in interestingNames) {
      return null
    }
    if (method.containingClass?.qualifiedName != GROOVY_LANG_CLOSURE) return null

    return when (methodName) {
      "memoize" -> receiver
      "call" -> receiver.returnType(arguments)
      "rcurry" -> receiver.curry(-arguments.size, arguments, context)
      "curry", "trampoline" -> receiver.curry(0, arguments, context)
      "ncurry" -> handleNcurry(receiver, arguments, context)
      else -> null
    }
  }

  private fun handleNcurry(receiver: GroovyClosureType, arguments: Arguments, context: PsiElement): PsiType? {
    val literal = arguments.firstOrNull()?.expression as? GrLiteral
    val value = literal?.value as? Int ?: return receiver
    return receiver.curry(value, arguments.drop(1), context)
  }

  companion object {
    private val interestingNames = setOf("call", "curry", "ncurry", "rcurry", "memoize", "trampoline")
  }
}
```

#### Key Capabilities
- **Closure Method Support**: Provides specific type inference for closure operations.
- **Context Awareness**: Determines types based on method names and argument types.

---

### **2. Default Groovy Methods (DGM) Call Type Calculator**
This example enhances type inference for Groovy's `DefaultGroovyMethods` (DGM), such as `find` and `flatten`:

```kotlin
class DgmCallTypeCalculator : GrCallTypeCalculator {

  override fun getType(receiver: PsiType?, method: PsiMethod, arguments: Arguments?, context: PsiElement): PsiType? {
    if (method.containingClass?.qualifiedName != DEFAULT_GROOVY_METHODS) {
      return null
    }

    val methodName = method.name
    return when {
      methodName == "find" -> (arguments?.first()?.type as? PsiArrayType)?.componentType
      methodName == "flatten" -> handleFlatten(receiver, context)
      methodName in interestingNames && isSimilarCollectionReturner(method) -> createSimilarCollection(receiver, context.project, getItemType(receiver))
      else -> null
    }
  }

  private fun handleFlatten(receiver: PsiType?, context: PsiElement): PsiType? {
    var itemType = getItemType(receiver)
    while (itemType != null) {
      itemType = getItemType(itemType)
    }
    return createSimilarCollection(receiver, context.project, itemType)
  }

  companion object {
    private val interestingNames = setOf("unique", "findAll", "grep", "collectMany", "split", "plus", "intersect", "leftShift")

    private fun getItemType(type: PsiType?): PsiType? {
      return (type as? PsiArrayType)?.componentType ?: extractIterableTypeParameter(type, true)
    }
  }
}
```

#### Key Capabilities
- **DGM Enhancements**: Provides type inference for collection-returning methods like `find` and `flatten`.
- **Dynamic Collection Support**: Dynamically creates types for collections with modified contents (e.g., filtered or transformed).

---

## Use Cases and Benefits

### **1. Enhancing Framework-Specific Method Calls**
Plugins for Groovy-based frameworks like Gradle or Grails can use `callTypeCalculator` to provide accurate type inference for domain-specific methods, improving code insight.

### **2. Closure and DSL Operations**
Custom Groovy DSLs can leverage this extension point to enhance support for dynamically typed operations, ensuring better developer productivity.

### **3. Improved Completion and Validation**
By providing accurate types for method calls, this extension point reduces ambiguity in code completion and ensures that errors are caught early during validation.

---

## Conclusion
The `callTypeCalculator` extension point is essential for enhancing Groovy's type inference in dynamic and context-dependent scenarios. It enables plugins to provide targeted, framework-aware type calculations, significantly improving the IDE experience for Groovy developers.

