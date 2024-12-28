# Custom Annotation Checker Extension Point

The `customAnnotationChecker` extension point provides a mechanism to implement custom validation logic for Groovy annotations. It enables plugin developers to add additional checks and validations on Groovy annotations, enhancing their usability and correctness in various contexts.

---

## Extension Point Declaration

```xml
<extensionPoint name="customAnnotationChecker" dynamic="true" interface="org.jetbrains.plugins.groovy.annotator.checkers.CustomAnnotationChecker"/>
```

---

## Key Interface: `CustomAnnotationChecker`

The `CustomAnnotationChecker` interface defines methods for implementing custom checks on Groovy annotations. These checks can validate argument lists, applicability, and other characteristics of annotations.

### Interface Definition

```java
public abstract class CustomAnnotationChecker {
  public static final ExtensionPointName<CustomAnnotationChecker> EP_NAME =
    ExtensionPointName.create("org.intellij.groovy.customAnnotationChecker");

  public boolean checkArgumentList(@NotNull AnnotationHolder holder, @NotNull GrAnnotation annotation) {
    return false;
  }

  public boolean checkApplicability(@NotNull AnnotationHolder holder, @NotNull GrAnnotation annotation) {
    return false;
  }
}
```

---

## Example Implementations

### 1. **`BuilderAnnotationChecker`**

This implementation validates the `@Builder` annotation in Groovy, ensuring specific attributes are not misused in certain configurations.

#### Capabilities
- **Error Highlighting:** Detects and highlights invalid combinations of annotation attributes, such as using `includeSuperProperties` with the `SimpleStrategy`.
- **Custom Error Messages:** Provides meaningful feedback to the user via error messages.

#### Code Example

```java
public final class BuilderAnnotationChecker extends CustomAnnotationChecker {
  @Override
  public boolean checkApplicability(@NotNull AnnotationHolder holder, @NotNull GrAnnotation annotation) {
    if (!BuilderAnnotationContributor.BUILDER_FQN.equals(annotation.getQualifiedName())) return false;

    if (BuilderAnnotationContributor.isApplicable(annotation, SimpleBuilderStrategySupport.SIMPLE_STRATEGY_NAME) &&
        BuilderAnnotationContributor.isIncludeSuperProperties(annotation)) {
      holder.newAnnotation(HighlightSeverity.ERROR,
        GroovyBundle.message("builder.annotation.not.support.super.for.simple.strategy"))
        .range(annotation)
        .create();
      return true;
    }

    return false;
  }
}
```

#### DSL Use Cases
- Ensures consistent and valid usage of builder annotations in Groovy DSLs.
- Simplifies debugging and validation of builder configurations.

---

### 2. **`DelegatesToAnnotationChecker`**

This checker validates the `@DelegatesTo` annotation, ensuring it is correctly used in conjunction with other annotations such as `@DelegatesTo.Target`.

#### Capabilities
- **Argument Validation:** Ensures the `value` attribute of the `@DelegatesTo` annotation is correctly specified or its default behavior is valid.
- **Context-Aware Validation:** Checks the surrounding context of the annotation to infer and validate its usage.

#### Code Example

```java
public final class DelegatesToAnnotationChecker extends CustomAnnotationChecker {
  @Override
  public boolean checkArgumentList(@NotNull AnnotationHolder holder, @NotNull GrAnnotation annotation) {
    if (!GroovyCommonClassNames.GROOVY_LANG_DELEGATES_TO.equals(annotation.getQualifiedName())) return false;

    final PsiAnnotationMemberValue valueAttribute = annotation.findAttributeValue("value");

    if (valueAttribute == null) {
      final PsiAnnotationOwner owner = annotation.getOwner();
      if (owner instanceof GrModifierList) {
        final PsiElement parent = ((GrModifierList)owner).getParent();
        if (parent instanceof GrParameter) {
          final PsiElement grandParent = parent.getParent();
          if (grandParent instanceof GrParameterList) {
            for (GrParameter parameter : ((GrParameterList)grandParent).getParameters()) {
              if (parameter.getModifierList().hasAnnotation(GroovyCommonClassNames.GROOVY_LANG_DELEGATES_TO_TARGET)) {
                return true;
              }
            }
          }
        }
      }
    }

    return false;
  }
}
```

#### DSL Use Cases
- Supports type-safe delegation in Groovy closures and DSLs.
- Enhances the usability of Groovy's `@DelegatesTo` annotation by ensuring correctness in complex configurations.

---

## General Capabilities Added by Implementations

### **Error Detection**
- Validates annotation arguments and contexts, reducing runtime errors and misconfigurations.

### **Custom Contextual Validation**
- Allows plugin developers to introduce highly specific checks tailored to their DSLs or frameworks.

### **Enhanced User Feedback**
- Provides users with actionable error messages, helping them resolve configuration issues quickly.

---

## Conclusion

The `customAnnotationChecker` extension point is a versatile tool for enhancing the correctness and usability of Groovy annotations. By implementing custom checks, developers can ensure better code quality and provide an improved development experience for Groovy DSL and framework users. Whether it's enforcing specific rules for annotations like `@Builder` or validating complex setups involving `@DelegatesTo`, this extension point unlocks a wide range of possibilities for Groovy plugin development.

