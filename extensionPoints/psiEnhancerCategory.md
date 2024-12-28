todo: add examples


# Psi Enhancer Category Extension Point

The `psiEnhancerCategory` extension point enables the dynamic addition of custom methods and properties to PSI (Program Structure Interface) classes in IntelliJ IDEA. This functionality is particularly useful for extending Groovy's dynamic capabilities, allowing custom logic or properties to be seamlessly integrated into the IDE's PSI model.

---

## Extension Point Declaration

```xml
<extensionPoint name="psiEnhancerCategory"
                dynamic="false"
                interface="org.jetbrains.plugins.groovy.dsl.psi.PsiEnhancerCategory"/>
```

### Purpose
This extension point is designed to:
- Add new methods and properties to existing PSI classes such as `PsiClass`, `PsiMethod`, and `PsiElement`.
- Provide utility functions for navigating and manipulating PSI elements in the context of Groovy projects.
- Enable DSL (Domain-Specific Language) support by enriching PSI elements with Groovy-specific behavior.

---

## Key Interface: `PsiEnhancerCategory`

```java
package org.jetbrains.plugins.groovy.dsl.psi;

import com.intellij.openapi.extensions.ExtensionPointName;

public interface PsiEnhancerCategory {
  ExtensionPointName<PsiEnhancerCategory> EP_NAME =
    ExtensionPointName.create("org.intellij.groovy.psiEnhancerCategory");
}
```

Implementing classes of `PsiEnhancerCategory` provide custom enhancements to specific PSI classes, which are dynamically integrated into IntelliJ IDEA's ecosystem. These enhancements are accessed and invoked transparently within the IDE through Groovy DSLs or programmatically by tools leveraging IntelliJ's PSI APIs. For example, enhanced methods such as `getMethods` or `bind` are directly callable on their respective PSI elements, enabling dynamic querying and manipulation without modifying the original PSI structure. This seamless integration ensures that the added methods are available for use in features like code completion, navigation, and refactoring, making these enhancements invaluable for Groovy-specific workflows.

---

## Example Implementations

### **1. `PsiClassCategory`**
This implementation adds methods to `PsiClass` for querying its properties and annotations.

#### **Key Methods:**
- **`getMethods(PsiClass clazz):`** Returns all methods in the class.
- **`getQualName(PsiClass clazz):`** Retrieves the qualified name of the class.
- **`hasAnnotation(PsiMember clazz, String annotName):`** Checks if the class has a specific annotation.
- **`getAnnotations(PsiMember clazz, String annotName):`** Retrieves all annotations matching a given name.

```java
public final class PsiClassCategory implements PsiEnhancerCategory {

  public static Collection<PsiMethod> getMethods(PsiClass clazz) {
    return Arrays.asList(clazz.getAllMethods());
  }

  public static @Nullable String getQualName(PsiClass clazz) {
    return clazz.getQualifiedName();
  }

  public static boolean hasAnnotation(PsiMember clazz, String annotName) {
    final PsiModifierList list = clazz.getModifierList();
    if (list == null) return false;
    for (PsiAnnotation annotation : list.getAnnotations()) {
      if (annotName.equals(annotation.getQualifiedName())) return true;
    }
    return false;
  }
}
```

#### **Use Cases:**
- Accessing methods or annotations dynamically for Groovy scripts or frameworks.
- Providing utilities for Groovy-based annotation processing.

---

### **2. `PsiElementCategory`**
This implementation provides utility methods for `PsiElement` objects.

#### **Key Methods:**
- **`bind(PsiElement element):`** Resolves the reference associated with the element.
- **`getQualifier(PsiElement elem):`** Retrieves the qualifier expression for a given element.
- **`asList(PsiElement elem):`** Converts elements into a list, useful for handling collections or array-like structures.
- **`eval(PsiElement elem):`** Evaluates and returns the value of a literal element.

```java
public final class PsiElementCategory implements PsiEnhancerCategory {

  public static @Nullable PsiElement bind(PsiElement element) {
    PsiReference ref = element.getReference();
    return ref == null ? null : ref.resolve();
  }

  public static @Nullable PsiElement getQualifier(PsiElement elem) {
    if (elem instanceof GrReferenceExpression) {
      return ((GrReferenceExpression) elem).getQualifierExpression();
    }
    return null;
  }

  public static @NotNull Collection<? extends PsiElement> asList(@Nullable PsiElement elem) {
    if (elem instanceof GrListOrMap) {
      return Arrays.asList(((GrListOrMap) elem).getInitializers());
    } else {
      return Collections.singleton(elem);
    }
  }

  public static Object eval(PsiElement elem) {
    if (elem instanceof GrLiteral literal) {
      return literal.getValue();
    }
    return elem;
  }
}
```

#### **Use Cases:**
- Simplifying reference resolution for Groovy PSI elements.
- Handling list or map structures dynamically in Groovy scripts.

---

### **3. `PsiMethodCategory`**
This implementation focuses on methods and their associated metadata.

#### **Key Methods:**
- **`getClassType(PsiField field):`** Retrieves the class type for a field.
- **`getParamStringVector(PsiMethod method):`** Returns a map of parameter types for a method.

```java
public final class PsiMethodCategory implements PsiEnhancerCategory {

  public static @Nullable PsiClass getClassType(PsiField field) {
    final PsiType type = field.getType();
    return PsiCategoryUtil.getClassType(type, field);
  }

  public static Map<String, String> getParamStringVector(PsiMethod method) {
    Map<String, String> result = new LinkedHashMap<>();
    int idx = 1;
    for (PsiParameter p : method.getParameterList().getParameters()) {
      result.put("value" + idx, p.getType().getCanonicalText());
      idx++;
    }
    return result;
  }
}
```

#### **Use Cases:**
- Inspecting method parameters and types dynamically.
- Supporting Groovy DSLs that rely on method metadata.

---

### **4. `PsiExpressionCategory`**
This implementation extends `GrExpression` with methods for working with arguments and class types.

#### **Key Methods:**
- **`getClassType(GrExpression expr):`** Retrieves the class type of an expression.
- **`getArguments(GrCall call):`** Returns the arguments of a method call as a collection.

```java
public final class PsiExpressionCategory implements PsiEnhancerCategory {

  public static @Nullable PsiClass getClassType(GrExpression expr) {
    final PsiType type = expr.getType();
    return PsiCategoryUtil.getClassType(type, expr);
  }

  public static Collection<GrExpression> getArguments(GrCall call) {
    final GrArgumentList argumentList = call.getArgumentList();
    if (argumentList != null) {
      return Arrays.asList(argumentList.getExpressionArguments());
    }
    return Collections.emptyList();
  }
}
```

#### **Use Cases:**
- Providing utilities for Groovy expressions in DSLs.
- Extracting arguments from method calls dynamically.

---

## Use Cases and Benefits

### **1. Groovy-Specific Enhancements**
The `psiEnhancerCategory` extension point allows developers to extend IntelliJ's PSI model with Groovy-specific methods and utilities. This is particularly useful for DSLs and frameworks that heavily rely on dynamic behavior.

### **2. Improved IDE Support**
By enriching the PSI with additional methods, this extension point improves code insight features like completion, navigation, and refactoring.

### **3. Simplified DSL Implementation**
For Groovy-based DSLs, these categories provide essential utilities for handling custom structures, annotations, and expressions seamlessly.

---

## Conclusion
The `psiEnhancerCategory` extension point is a powerful tool for extending IntelliJ IDEA's PSI model to support Groovy-specific use cases. By defining custom enhancements for PSI classes, developers can build robust plugins that cater to Groovy's dynamic and DSL-oriented nature.

