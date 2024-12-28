# Inspection Disabler Extension Point

The `inspectionDisabler` extension point allows plugin developers to define file types where specific inspections are disabled by default. This is particularly useful for controlling inspections in file types where certain checks are not applicable or may lead to false positives.

---

## Extension Point Declaration

```xml
<extensionPoint name="inspectionDisabler"
                beanClass="com.intellij.openapi.fileTypes.FileTypeExtensionPoint"
                dynamic="true">
    <with attribute="implementationClass" implements="org.jetbrains.plugins.groovy.codeInspection.FileTypeInspectionDisabler"/>
</extensionPoint>
```

---

## Purpose

- **Selective Disabling of Inspections**: Control which inspections should be ignored for specific file types.
- **Improved Developer Experience**: Minimize irrelevant warnings or errors in files where they don't apply.
- **Support for Domain-Specific File Types**: Tailor inspections to only relevant file types in DSLs or specialized languages.

---

## Key Interfaces and Classes

### 1. `FileTypeExtensionPoint<T>`

Defines a relationship between a file type and its associated `FileTypeInspectionDisabler` implementation.

```java
public final class FileTypeExtensionPoint<T> extends BaseKeyedLazyInstance<T> implements KeyedLazyInstance<T> {

  @Attribute("filetype")
  @RequiredElement
  public String filetype;

  @Attribute("implementationClass")
  @RequiredElement
  public String implementationClass;

  @Override
  public @NotNull String getKey() {
    return filetype;
  }

  @Override
  protected @Nullable String getImplementationClassName() {
    return implementationClass;
  }
}
```

#### Attributes:
- `filetype`: The file type for which the inspection disabler is registered.
- `implementationClass`: Fully qualified name of the disabler class.

---

### 2. `FileTypeInspectionDisabler`

Defines the logic for disabling inspections for a specific file type.

#### Interface Methods

```kotlin
interface FileTypeInspectionDisabler {
  fun getDisableableInspections(): Set<Class<out LocalInspectionTool>>

  fun isTypecheckingDisabled(file: PsiFile): Boolean
}
```

- `getDisableableInspections()`: Returns a set of `LocalInspectionTool` classes that should be disabled.
- `isTypecheckingDisabled(PsiFile)`: Determines if type checking is disabled for the given file.

---

### 3. `FileTypeInspectionDisablers`

A service managing all registered `FileTypeInspectionDisabler` instances.

```kotlin
@Service(Service.Level.APP)
class FileTypeInspectionDisablers : FileTypeExtension<FileTypeInspectionDisabler>("org.intellij.groovy.inspectionDisabler")

fun getDisableableFileTypes(clazz : Class<out LocalInspectionTool>) : Set<FileType> {
  val allDisablers : Map<FileType, FileTypeInspectionDisabler> =
    ApplicationManager.getApplication().getService(FileTypeInspectionDisablers::class.java).allRegisteredExtensions
  return allDisablers.filterValues { disabler -> disabler.getDisableableInspections().contains(clazz) }.keys
}

internal fun isTypecheckingDisabled(file: PsiFile?) : Boolean {
  file ?: return false
  val disablers = ApplicationManager.getApplication().getService(FileTypeInspectionDisablers::class.java).allRegisteredExtensions
  return disablers.values.any { it.isTypecheckingDisabled(file) }
}
```

---

## Implementation Example

### Example: Disabling Inspections for Groovy File Types

The following implementation disables specific inspections for Groovy-related file types:

```kotlin
package org.jetbrains.plugins.groovy.codeInspection

import com.intellij.codeInspection.LocalInspectionTool
import com.intellij.psi.PsiFile

class GroovyFileInspectionDisabler : FileTypeInspectionDisabler {

  override fun getDisableableInspections(): Set<Class<out LocalInspectionTool>> {
    return setOf(GroovySpecificInspection::class.java)
  }

  override fun isTypecheckingDisabled(file: PsiFile): Boolean {
    return file.fileType.name == "Groovy"
  }
}
```

In this example:
- `getDisableableInspections()` returns a set of inspections specific to Groovy files.
- `isTypecheckingDisabled()` disables type checking for files with the `Groovy` file type.

---

## Registration

To register a `FileTypeInspectionDisabler` implementation, add the following to your plugin’s `plugin.xml`:

```xml
<inspectionDisabler filetype="Groovy" implementationClass="org.jetbrains.plugins.groovy.codeInspection.GroovyFileInspectionDisabler"/>
```

---

## Use Cases

### 1. DSL-Specific Inspections
For DSLs built on Groovy or other languages, certain inspections may not apply. For example:
- Suppress type-related inspections in configuration files.
- Disable unused variable inspections for templates.

### 2. Custom File Types
For custom file types introduced by a plugin, you can disable irrelevant inspections to avoid noise for users.

### 3. Performance Optimization
For large files or files with complex structures, disabling specific inspections can improve IDE performance.

---

## Conclusion

The `inspectionDisabler` extension point provides a flexible way to control inspections for specific file types. By tailoring inspection behavior, plugin developers can enhance the IDE experience for users working with domain-specific languages or custom file types.

