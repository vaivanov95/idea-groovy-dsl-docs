# Groovy Element Filter Extension Point

The `GroovyElementFilter` extension point allows developers to define filters for Groovy PSI (Program Structure Interface) elements. These filters determine whether an element is "real" or should be treated as a "fake" representation, often as part of a DSL or specialized language feature. This functionality is particularly useful for Groovy DSLs like Spock and Gradle, where certain elements may represent higher-level abstractions or constructs.

---

## Extension Point Declaration

```xml
<extensionPoint name="elementFilter" dynamic="true" interface="org.jetbrains.plugins.groovy.lang.GroovyElementFilter"/>
```

## Key Interface: `GroovyElementFilter`

The `GroovyElementFilter` interface defines a single method to evaluate whether a given PSI element is "fake" and thus not directly representative of Groovy code.

### Interface Definition

```kotlin
@Experimental
interface GroovyElementFilter {

  /**
   * @return `true` if [element] is not really an element,
   * meaning it would be transformed into something else,
   * otherwise `false`
   */
  fun isFake(element: GroovyPsiElement): Boolean
}
```

---

## Example Implementations

### 1. **Spock Element Filter**

The `SpockElementFilter` is designed to handle elements within the Spock testing framework. Spock introduces constructs like feature methods and data tables, which may require special handling as "fake" elements to align with Spock's DSL semantics.

#### Code Example

```kotlin
class SpockElementFilter : GroovyElementFilter {

  override fun isFake(element: GroovyPsiElement): Boolean {
    return (
             isFakeExpression(element)
             || isFakeMethod(element)
           )
           && element.isInsideSpecification()
  }

  private fun isFakeExpression(element: GroovyPsiElement): Boolean {
    return element is GrExpression && (element.isInteractionPart() || element.isTableColumnSeparator())
  }

  private fun isFakeMethod(element: GroovyPsiElement): Boolean {
    return element is GrMethod && isFeatureMethod(element)
  }
}
```

#### Key Features
- **Fake Expressions:** Identifies Spock-specific constructs such as interaction parts or data table column separators.
- **Fake Methods:** Marks methods as fake if they are Spock feature methods.

#### Use Cases
- **Spock Testing Framework:** Provides IDE support for recognizing Spock's DSL constructs as distinct from regular Groovy code.

---

### 2. **Gradle Task Declaration Element Filter**

The `GradleTaskDeclarationElementFilter` is specific to Gradle's DSL. Gradle scripts often define tasks and other elements that may not directly correspond to standard Groovy constructs.

#### Code Example

```kotlin
class GradleTaskDeclarationElementFilter : GroovyElementFilter {

  override fun isFake(element: GroovyPsiElement): Boolean {
    return isFakeInner(element) && element.containingFile.isGradleScript()
  }
}
```

#### Key Features
- **Gradle Script Context:** Filters elements only within files identified as Gradle scripts.
- **Task-Specific Filtering:** Marks elements as fake if they represent Gradle-specific constructs.

#### Use Cases
- **Gradle Build Scripts:** Enhances IDE behavior for handling Gradle's custom DSL constructs.

---

## Integration with the IntelliJ Platform

### **How It Works**
1. **Registration:** Implementations of `GroovyElementFilter` are registered via the `elementFilter` extension point in a plugin's XML configuration.
2. **Evaluation:** The IDE queries registered filters to determine whether a given element is fake.
3. **Transformation:** Fake elements are typically transformed or ignored during code analysis, completion, and other IDE features.

### **Benefits**
- **DSL Customization:** Provides tailored support for Groovy-based DSLs.
- **Improved Code Insight:** Ensures accurate code navigation and completion by distinguishing real and fake elements.
- **Seamless Integration:** Works transparently within the existing IntelliJ Platform infrastructure.

---

## Conclusion

The `GroovyElementFilter` extension point is a powerful mechanism for adapting IntelliJ IDEA's behavior to Groovy DSLs and frameworks. By defining custom filters, developers can ensure that the IDE accurately represents DSL constructs, providing a more intuitive and productive experience for Groovy developers.

