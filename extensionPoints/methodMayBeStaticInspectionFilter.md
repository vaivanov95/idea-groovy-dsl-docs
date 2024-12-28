# Method May Be Static Inspection Filter

The `methodMayBeStaticInspectionFilter` extension point allows plugin developers to customize and refine IntelliJ IDEA's inspection behavior for detecting methods that can be declared `static` in Groovy classes. By implementing this extension, you can ignore specific methods from being flagged by the `Method May Be Static` inspection based on domain-specific rules or project requirements.

---

## Extension Point Declaration

```xml
<extensionPoint name="methodMayBeStaticInspectionFilter" dynamic="true"
                interface="org.jetbrains.plugins.groovy.codeInspection.declaration.GrMethodMayBeStaticInspectionFilter"/>
```

---

## Key Interface: `GrMethodMayBeStaticInspectionFilter`

The `GrMethodMayBeStaticInspectionFilter` interface defines the contract for excluding specific methods from being flagged by the inspection.

### Interface Definition

```java
public abstract class GrMethodMayBeStaticInspectionFilter {

  public static final ExtensionPointName<GrMethodMayBeStaticInspectionFilter> EP_NAME =
    new ExtensionPointName<>("org.intellij.groovy.methodMayBeStaticInspectionFilter");

  public abstract boolean isIgnored(@NotNull GrMethod method);
}
```

---

## Example Implementation: Excluding DSL-Specific Methods

### Implementation Details
The following implementation filters out methods that are part of a Groovy-based Domain-Specific Language (DSL) where `static` methods may not align with the DSL's semantics or design.

```java
package org.jetbrains.plugins.groovy.custom;

import org.jetbrains.annotations.NotNull;
import org.jetbrains.plugins.groovy.codeInspection.declaration.GrMethodMayBeStaticInspectionFilter;
import org.jetbrains.plugins.groovy.lang.psi.api.statements.typedef.members.GrMethod;

public class DslMethodInspectionFilter extends GrMethodMayBeStaticInspectionFilter {

  @Override
  public boolean isIgnored(@NotNull GrMethod method) {
    // Example: Ignore methods annotated with @DslMarker
    return method.getModifierList().findAnnotation("my.dsl.DslMarker") != null;
  }
}
```

### Behavior
- **Annotation-Based Filtering:** Methods annotated with `@DslMarker` are excluded from the `Method May Be Static` inspection.
- **Use Case:** Ensures that DSL methods remain non-static to maintain the DSL’s expected behavior.

---

## Use Cases for Custom Filters

### 1. **Framework-Specific Methods**
Certain methods in Groovy frameworks (e.g., Grails or Gradle) are designed to be non-static for lifecycle or contextual reasons. For instance:
- Grails service methods depend on dependency injection and should not be static.
- Gradle task definitions rely on dynamic properties.

### Example
```java
@Override
public boolean isIgnored(@NotNull GrMethod method) {
  PsiClass containingClass = method.getContainingClass();
  if (containingClass != null && containingClass.getQualifiedName().startsWith("grails.")) {
    return true;
  }
  return false;
}
```

---

### 2. **Dynamic Behavior in Scripting**
Groovy scripts often define methods that rely on runtime context. Declaring such methods as `static` could break their intended functionality.

### Example
```java
@Override
public boolean isIgnored(@NotNull GrMethod method) {
  return method.getContainingClass().isScript();
}
```

---

### 3. **Excluding Specific Classes or Packages**
Custom filters can target specific classes or packages to suppress the inspection.

### Example
```java
@Override
public boolean isIgnored(@NotNull GrMethod method) {
  PsiClass containingClass = method.getContainingClass();
  return containingClass != null && containingClass.getQualifiedName().startsWith("com.example.generated");
}
```

---

## Integration with IntelliJ IDEA

- **Inspection Behavior:** When the `Method May Be Static` inspection runs, it invokes all registered `GrMethodMayBeStaticInspectionFilter` extensions. If any filter returns `true` for a method, that method is excluded from inspection.
- **Dynamic Extension:** Developers can define custom logic dynamically, adapting the inspection behavior to specific frameworks, projects, or use cases.

---

## Conclusion

The `methodMayBeStaticInspectionFilter` extension point provides a flexible way to refine IntelliJ IDEA’s static method inspection for Groovy. By leveraging this extension, developers can:
- Align inspections with domain-specific requirements.
- Prevent false positives in DSLs and framework-based projects.
- Ensure that Groovy methods adhere to the intended design patterns and behaviors.

