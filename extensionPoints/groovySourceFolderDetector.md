# Groovy Source Folder Detector Extension Point

The `groovySourceFolderDetector` extension point allows IntelliJ IDEA plugins to identify directories that serve as Groovy source folders in a project. This capability is useful for automating project setup, configuration, or providing enhanced IDE features for Groovy projects.

---

## Extension Point Declaration

```xml
<extensionPoint name="groovySourceFolderDetector"
                dynamic="true"
                interface="org.jetbrains.plugins.groovy.actions.GroovySourceFolderDetector"/>
```

---

## Key Interface: `GroovySourceFolderDetector`

The `GroovySourceFolderDetector` is an abstract class that needs to be extended to implement custom logic for identifying Groovy source folders.

### Interface Definition

```java
package org.jetbrains.plugins.groovy.actions;

import com.intellij.openapi.extensions.ExtensionPointName;
import com.intellij.psi.PsiDirectory;

public abstract class GroovySourceFolderDetector {

  public static final ExtensionPointName<GroovySourceFolderDetector> EP_NAME =
    ExtensionPointName.create("org.intellij.groovy.groovySourceFolderDetector");

  /**
   * Determines if the specified directory is a Groovy source folder.
   *
   * @param directory The directory to check.
   * @return true if the directory is identified as a Groovy source folder; false otherwise.
   */
  public abstract boolean isGroovySourceFolder(PsiDirectory directory);
}
```

---

## Purpose

This extension point serves multiple purposes:

- **Project Configuration Automation:** Automatically detect and configure Groovy source folders in projects.
- **IDE Enhancements:** Provide specific features such as Groovy-specific inspections, code generation, or module configuration for identified source folders.
- **Custom Rules:** Allow plugin developers to define custom rules for identifying Groovy source folders based on project structure, naming conventions, or other heuristics.

---

## Example Implementation

Below is an example of how to implement the `GroovySourceFolderDetector` to identify Groovy source folders based on naming conventions or directory structure.

```java
package org.jetbrains.plugins.groovy.actions;

import com.intellij.psi.PsiDirectory;
import com.intellij.psi.PsiFile;

public class NamingConventionGroovySourceFolderDetector extends GroovySourceFolderDetector {

  @Override
  public boolean isGroovySourceFolder(PsiDirectory directory) {
    // Check if the directory name matches a common Groovy source folder pattern
    String name = directory.getName();
    if ("groovy".equalsIgnoreCase(name)) {
      return true;
    }

    // Check if the directory contains any Groovy files
    for (PsiFile file : directory.getFiles()) {
      if (file.getName().endsWith(".groovy")) {
        return true;
      }
    }

    return false;
  }
}
```

### Explanation

1. **Directory Naming:** The implementation checks if the directory name is `groovy`, a common naming convention for Groovy source folders.
2. **File Inspection:** The implementation checks if the directory contains any `.groovy` files, further validating it as a Groovy source folder.

---

## Use Cases

### 1. **Build Tools Integration**
Automatically detect Groovy source folders in Gradle or Maven projects and configure them as part of the source set.

### 2. **Custom Project Structures**
Handle non-standard project structures where Groovy source folders may not follow conventional naming patterns, allowing plugins to add tailored support.

### 3. **Enhanced Developer Productivity**
By detecting Groovy source folders, the IDE can:
- Enable Groovy-specific inspections and code completion.
- Configure module dependencies automatically.
- Provide context-aware suggestions and navigation.

---

## Conclusion

The `groovySourceFolderDetector` extension point empowers plugin developers to create smarter tools that can identify and leverage Groovy source folders in projects. By implementing custom logic through this extension point, plugins can provide a seamless and enriched development experience for Groovy developers in IntelliJ IDEA.

