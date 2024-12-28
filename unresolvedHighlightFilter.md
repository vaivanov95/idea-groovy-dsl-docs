# Unresolved Highlight Filter Extension Point

The `unresolvedHighlightFilter` extension point enables plugin developers to suppress IDE highlights for unresolved references in Groovy code. This capability is particularly valuable for handling dynamic constructs or custom Abstract Syntax Tree (AST) transformations, where standard resolution rules may not apply, and highlighting unresolved references might be misleading.

---

## Extension Point Declaration

```xml
<extensionPoint name="unresolvedHighlightFilter"
                dynamic="true"
                interface="org.jetbrains.plugins.groovy.extensions.GroovyUnresolvedHighlightFilter"/>
```

---

## Purpose and Capabilities

### **1. Suppressing Irrelevant Warnings**

Dynamic languages like Groovy often use runtime or custom transformations that are not visible to the IDE’s static analysis. Unresolved reference warnings in such cases can:
- Highlight valid references incorrectly.
- Add unnecessary noise to the development experience.

This extension point helps developers filter out these incorrect warnings, improving clarity.

### **2. Supporting Custom AST Transformations**

Dynamic AST transformations (e.g., those implemented via Groovy macros or runtime extensions) may introduce elements that standard resolution processes cannot handle. The filter ensures that references related to these transformations are not unnecessarily flagged.

### **3. Flexible Configuration**

Plugin developers can implement custom logic to:
- Identify when and where unresolved references should be suppressed.
- Integrate seamlessly with other dynamic or domain-specific enhancements.

---

## Key Interface

### `GroovyUnresolvedHighlightFilter`

This abstract class defines a method for determining whether a specific reference should be excluded from unresolved highlighting.

```java
public abstract class GroovyUnresolvedHighlightFilter {

  public static final ExtensionPointName<GroovyUnresolvedHighlightFilter> EP_NAME =
      ExtensionPointName.create("org.intellij.groovy.unresolvedHighlightFilter");

  public abstract boolean isReject(@NotNull GrReferenceExpression expression);

  public static boolean shouldHighlight(@NotNull GrReferenceExpression expression) {
    for (GroovyUnresolvedHighlightFilter filter : EP_NAME.getExtensions()) {
      if (filter.isReject(expression)) return false;
    }
    return true;
  }
}
```

#### Key Methods:
- **`isReject(GrReferenceExpression expression)`**: Determines whether a given reference should be suppressed from unresolved highlighting.
- **`shouldHighlight(GrReferenceExpression expression)`**: Iterates through all registered filters to decide whether a reference should be highlighted.

---

## Example Implementation: Gradle Unresolved Reference Filter

### Overview

The `GradleUnresolvedReferenceFilter` suppresses unresolved warnings for certain Gradle-specific dynamic elements. Gradle dynamically adds properties like `task` or `sourceSet` at runtime, which might otherwise trigger unresolved reference warnings.

```java
public final class GradleUnresolvedReferenceFilter extends GroovyUnresolvedHighlightFilter {

  private static final Set<String> IGNORE_SET = ContainerUtil.newHashSet(
    GradleCommonClassNames.GRADLE_API_TASK,
    GradleCommonClassNames.GRADLE_API_SOURCE_SET,
    GradleCommonClassNames.GRADLE_API_CONFIGURATION,
    GradleCommonClassNames.GRADLE_API_DISTRIBUTION
  );

  @Override
  public boolean isReject(@NotNull GrReferenceExpression expression) {
    final PsiType psiType = GradleResolverUtil.getTypeOf(expression);
    if (psiType == null) {
      PsiElement child = expression.getFirstChild();
      if (child == null) return false;

      PsiReference reference = child.getReference();
      if (reference instanceof GrReferenceExpression) {
        PsiType type = ((GrReferenceExpression) reference).getType();
        if (type != null) {
          return InheritanceUtil.isInheritor(type, GRADLE_API_EXTRA_PROPERTIES_EXTENSION);
        }
      }
      return false;
    }

    return false;
  }
}
```

### Capabilities

1. **Suppress Dynamic Gradle Properties**:
   - Gradle adds runtime properties like `task` and `sourceSet`.
   - These are dynamically resolved, so this filter prevents them from being flagged as unresolved.

2. **Integration with Gradle DSL**:
   - Enhances the experience for developers working with Gradle by reducing false positives in highlighting.

3. **Custom Logic**:
   - Implements specific checks to determine if a reference is part of Gradle’s dynamically added properties.

---

## Use Cases and Benefits

### **1. Improving IDE Support for Dynamic Languages**
This extension point bridges the gap between Groovy’s dynamic nature and the IDE’s static analysis capabilities. It ensures that developers are not overwhelmed by irrelevant warnings.

### **2. Domain-Specific Enhancements**
Filters can be tailored for specific domains or frameworks, such as:
- **Gradle**: Suppressing unresolved highlights for dynamically added Gradle properties.
- **Custom DSLs**: Handling dynamic constructs in domain-specific languages built on Groovy.

### **3. Enhancing Developer Productivity**
By reducing noise from false positives, this extension point allows developers to focus on actual issues, improving productivity and the overall development experience.

---

## Conclusion

The `unresolvedHighlightFilter` extension point is a powerful tool for tailoring IDE behavior to the unique characteristics of Groovy. By enabling suppression of unnecessary unresolved highlights, it enhances support for dynamic constructs, custom DSLs, and runtime transformations, creating a more intuitive and developer-friendly experience.

