# Groovy Framework Configuration Notification Extension Point

The `groovyFrameworkConfigNotification` extension point allows IntelliJ IDEA plugins to define notifications that help developers configure the Groovy framework in their projects. This extension point identifies whether a module conforms to a Groovy framework's expected structure and whether it has the necessary library dependencies.

---

## Extension Point Declaration

```xml
<extensionPoint name="groovyFrameworkConfigNotification"
                dynamic="true"
                interface="org.jetbrains.plugins.groovy.annotator.GroovyFrameworkConfigNotification"/>
```

### Purpose
This extension point serves to:

- Detect if a module adheres to a specific Groovy framework's structure.
- Verify if the required libraries for the Groovy framework are present in the module.
- Trigger framework-specific setup notifications or guidance for users.

---

## Key Interface: `GroovyFrameworkConfigNotification`

The `GroovyFrameworkConfigNotification` interface is implemented by extensions to define custom logic for framework detection and library verification.

```java
package org.jetbrains.plugins.groovy.annotator;

import com.intellij.openapi.extensions.ExtensionPointName;
import com.intellij.openapi.module.Module;
import org.jetbrains.annotations.NotNull;

public abstract class GroovyFrameworkConfigNotification {

  public static final ExtensionPointName<GroovyFrameworkConfigNotification> EP_NAME =
    ExtensionPointName.create("org.intellij.groovy.groovyFrameworkConfigNotification");

  public abstract boolean hasFrameworkStructure(@NotNull Module module);

  public abstract boolean hasFrameworkLibrary(@NotNull Module module);
}
```

### Overview of Methods

1. **`hasFrameworkStructure(Module module):`**
   - Checks if the given module conforms to the expected directory or file structure of the framework.
   - Example: Verifying if the module contains a `src/main/groovy` directory.

2. **`hasFrameworkLibrary(Module module):`**
   - Checks if the required libraries for the framework are present in the module.
   - Example: Verifying if the `groovy-all` library is available in the module's classpath.

---

## Example Implementation

### Default Implementation

The following implementation provides basic support for detecting the Groovy framework by:
- Always assuming the framework structure is valid.
- Checking if the `groovy-all` library is available in the module.

```java
package org.jetbrains.plugins.groovy.config;

import com.intellij.openapi.module.Module;
import com.intellij.psi.JavaPsiFacade;
import org.jetbrains.annotations.NotNull;
import org.jetbrains.plugins.groovy.annotator.GroovyFrameworkConfigNotification;
import org.jetbrains.plugins.groovy.lang.psi.util.GroovyCommonClassNames;

final class DefaultGroovyFrameworkConfigNotification extends GroovyFrameworkConfigNotification {

  @Override
  public boolean hasFrameworkStructure(@NotNull Module module) {
    return true; // Assume all modules conform to the framework structure.
  }

  @Override
  public boolean hasFrameworkLibrary(@NotNull Module module) {
    return JavaPsiFacade.getInstance(module.getProject()).findClass(
      GroovyCommonClassNames.GROOVY_OBJECT, module.getModuleWithDependenciesAndLibrariesScope(true)
    ) != null;
  }
}
```

### Key Features of the Example

1. **Framework Structure Validation:**
   - This implementation assumes that any module could potentially conform to the framework's structure. Developers can override this method to enforce stricter checks, such as ensuring specific directories or configuration files exist.

2. **Library Presence Validation:**
   - The implementation checks if the `GroovyObject` class is resolvable in the module's dependencies. This class is part of the `groovy-all` library, which is essential for any Groovy framework.

---

## Use Cases

### 1. Framework-Specific Detection
Custom implementations of this extension point can:
- Detect frameworks such as Grails or Gradle by inspecting the module's file structure.
- Ensure specific configurations or files (e.g., `application.yml` for Grails) are present.

### 2. Missing Library Notifications
If a required library is missing, the plugin can:
- Notify the user to add the necessary library.
- Offer quick fixes or actions to resolve the issue, such as adding dependencies via Maven or Gradle.

### 3. Enhanced IDE Support
By identifying framework modules correctly, plugins can:
- Provide targeted code assistance, such as framework-specific inspections and code completions.
- Enable specialized run configurations or build tools for the framework.

---

## Integration with IntelliJ IDEA

### 1. Registering the Extension
To use this extension point, register your implementation in the plugin's `plugin.xml`:

```xml
<extensions defaultExtensionNs="org.intellij.groovy">
  <groovyFrameworkConfigNotification implementation="com.example.MyGroovyFrameworkNotification"/>
</extensions>
```

### 2. Accessing Notifications
Framework configuration notifications can be accessed programmatically using the `EP_NAME` constant:

```java
for (GroovyFrameworkConfigNotification notification : GroovyFrameworkConfigNotification.EP_NAME.getExtensions()) {
  if (notification.hasFrameworkStructure(module) && notification.hasFrameworkLibrary(module)) {
    // Perform framework-specific actions
  }
}
```

---

## Conclusion

The `groovyFrameworkConfigNotification` extension point provides a flexible mechanism for detecting and validating Groovy framework configurations within IntelliJ IDEA. By implementing this extension, plugin developers can:

- Ensure modules conform to specific frameworks.
- Assist users in resolving misconfigurations or missing dependencies.
- Enhance the IDE's support for Groovy frameworks through tailored insights and notifications.

This extension point is an essential tool for improving the developer experience in Groovy-based projects.

