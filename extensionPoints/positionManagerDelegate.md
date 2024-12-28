# Script Position Manager Helper Extension Point

The `positionManagerDelegate` extension point is designed to extend the debugger functionality for handling various Groovy scripts. This extension point allows plugins to enhance debugging capabilities by customizing the mapping between runtime script representations and source files or classes.

---

## Extension Point Declaration

```xml
<extensionPoint name="positionManagerDelegate"
                dynamic="true"
                interface="org.jetbrains.plugins.groovy.extensions.debugger.ScriptPositionManagerHelper"/>
```

---

## Purpose

- **Mapping Runtime Names to Source**: Resolve runtime script names to their corresponding source files or classes.
- **Custom Debugging Support**: Enable debugging for specialized Groovy scripts, such as DSLs or build scripts.
- **File Type Flexibility**: Extend debugger support to non-Groovy file types if needed.

---

## Key Interface: `ScriptPositionManagerHelper`

### Overview

The `ScriptPositionManagerHelper` abstract class provides methods to:

1. Determine if a runtime name or script file is appropriate for a particular script type.
2. Map runtime script names to source files.
3. Customize the fully qualified runtime class name for debugging.

### Key Methods

#### `isAppropriateRuntimeName`

Determines if the helper can handle a given runtime name.

```java
public boolean isAppropriateRuntimeName(@NotNull String runtimeName) {
    return false;
}
```

- **Input**: `runtimeName` - The runtime name of the class.
- **Returns**: `true` if the helper can handle the runtime name.

---

#### `getOriginalScriptName`

Maps a runtime name to the fully qualified script name.

```java
public @Nullable String getOriginalScriptName(@NotNull ReferenceType refType, @NotNull String runtimeName) {
    return null;
}
```

- **Input**:
  - `refType`: The `ReferenceType` from the debugger.
  - `runtimeName`: The runtime name of the class.
- **Returns**: The fully qualified script name or `null` if not applicable.

---

#### `isAppropriateScriptFile`

Checks if a given script file is appropriate for the helper.

```java
public boolean isAppropriateScriptFile(@NotNull GroovyFile scriptFile) {
    return false;
}
```

- **Input**: `scriptFile` - The script file to evaluate.
- **Returns**: `true` if the file matches the criteria for the helper.

---

#### `getRuntimeScriptName`

Retrieves the runtime name for a given Groovy script file.

```java
public @Nullable String getRuntimeScriptName(@NotNull GroovyFile groovyFile) {
    return null;
}
```

- **Input**: `groovyFile` - The Groovy script file.
- **Returns**: The runtime name or `null` if not applicable.

---

#### `getExtraScriptIfNotFound`

Provides a fallback script file if the debugger cannot find one using standard methods.

```java
public @Nullable PsiFile getExtraScriptIfNotFound(@NotNull ReferenceType refType,
                                                  @NotNull String runtimeName,
                                                  @NotNull Project project,
                                                  @NotNull GlobalSearchScope scope) {
    return null;
}
```

- **Input**:
  - `refType`: The `ReferenceType` from the debugger.
  - `runtimeName`: The runtime name of the class.
  - `project`: The current project.
  - `scope`: The search scope.
- **Returns**: The fallback script file or `null` if none is found.

---

#### `customizeClassName`

Customizes the fully qualified class name for a given `PsiClass`.

```java
public @Nullable String customizeClassName(@NotNull PsiClass psiClass) {
    return null;
}
```

- **Input**: `psiClass` - The class to customize.
- **Returns**: The customized class name or `null` if no customization is required.

---

#### `getAcceptedFileTypes`

Specifies additional file types supported by this helper.

```java
public Collection<? extends FileType> getAcceptedFileTypes() {
    return Collections.emptySet();
}
```

- **Returns**: A collection of supported file types.

---

## Implementation Examples

### Gradle Scripts Debugging

This implementation enables debugging for Gradle scripts by mapping their runtime names to corresponding source files.

```java
public final class GradlePositionManager extends ScriptPositionManagerHelper {

  @Override
  public boolean isAppropriateRuntimeName(final @NotNull String runtimeName) {
    return true;
  }

  @Override
  public boolean isAppropriateScriptFile(final @NotNull GroovyFile scriptFile) {
    return GroovyScriptUtil.isSpecificScriptFile(scriptFile, GradleScriptType.INSTANCE);
  }

  @Override
  public @NotNull String getRuntimeScriptName(@NotNull GroovyFile groovyFile) {
    // Implementation for Gradle-specific script runtime name resolution.
  }

  @Override
  public Collection<? extends FileType> getAcceptedFileTypes() {
    return Collections.singleton(GradleFileType.INSTANCE);
  }
}
```

---

### Gant Scripts Debugging

An implementation for handling Gant scripts, another type of Groovy-based scripting language.

```java
public final class GantPositionManagerHelper extends ScriptPositionManagerHelper {

  @Override
  public boolean isAppropriateRuntimeName(final @NotNull String runtimeName) {
    return true;
  }

  @Override
  public boolean isAppropriateScriptFile(final @NotNull GroovyFile scriptFile) {
    return GroovyScriptUtil.isSpecificScriptFile(scriptFile, GantScriptType.INSTANCE);
  }

  @Override
  public PsiFile getExtraScriptIfNotFound(@NotNull ReferenceType refType,
                                          final @NotNull String runtimeName,
                                          final @NotNull Project project,
                                          @NotNull GlobalSearchScope scope) {
    // Fallback logic to find Gant scripts by name.
  }
}
```

---

## Use Cases

### 1. Debugging Domain-Specific Groovy Scripts

For custom DSLs or scripting languages built on Groovy, plugins can implement `ScriptPositionManagerHelper` to ensure seamless debugging experiences.

### 2. Multi-Environment Script Mapping

Handle scripts across different environments (e.g., local files, remote WSL paths) by customizing runtime-to-source mapping.

### 3. File Type Extension

Support debugging for non-Groovy file types that use Groovy-based runtime environments, such as Gradle or Gant.

---

## Registration

To register a `ScriptPositionManagerHelper` implementation, add the following to your `plugin.xml`:

```xml
<positionManagerDelegate implementation="com.example.MyScriptPositionManagerHelper"/>
```

---
