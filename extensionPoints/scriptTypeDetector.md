# Script Type Detector Extension Point

The `scriptTypeDetector` extension point in the IntelliJ Platform allows developers to define and detect specific types of Groovy scripts. This enables enhanced support for domain-specific script types, such as custom icons, resolution scopes, and behavior adjustments.

## Definition

```xml
<extensionPoint name="scriptTypeDetector" dynamic="true" interface="org.jetbrains.plugins.groovy.extensions.GroovyScriptTypeDetector"/>
```

## Key Classes

### `GroovyScriptTypeDetector`
The `GroovyScriptTypeDetector` abstract class is responsible for identifying specific Groovy script types.

#### Key Methods:
- `@NotNull GroovyScriptType getScriptType()`
  - Returns the script type associated with the detector.

- `boolean isSpecificScriptFile(@NotNull GroovyFile script)`
  - Determines if a given Groovy file matches the script type.

- `@Nullable static GroovyScriptType getScriptType(@NotNull GroovyFile file)`
  - Iterates over all registered detectors to find the script type for a file.

### `GroovyScriptType`
The `GroovyScriptType` abstract class represents a specific type of Groovy script.

#### Key Methods:
- `@NotNull String getId()`
  - Returns the unique identifier for the script type.

- `@NotNull Icon getScriptIcon()`
  - Provides a custom icon for the script type.

- `GlobalSearchScope patchResolveScope(@NotNull GroovyFile file, @NotNull GlobalSearchScope baseScope)`
  - Allows modifications to the resolve scope for the script type.

### Example Script Type: `GradleScriptType`
This script type supports Gradle build scripts (`.gradle` files). It:
- Provides a custom icon for Gradle scripts.
- Adjusts the resolution scope to include Gradle-specific dependencies and settings.

#### Highlights:
- Overrides `patchResolveScope` to include:
  - JDK dependencies.
  - Gradle-specific classpath entries.

## Example Instances

Below is an example registration for a script type detector:

```xml
<scriptTypeDetector implementation="com.example.MyCustomScriptTypeDetector"/>
```

## Purpose of the Extension

The `scriptTypeDetector` extension point enables IntelliJ IDEA to:
- Identify and differentiate between different Groovy script types.
- Provide custom behavior, icons, and resolution logic for specific script types.
- Improve support for domain-specific languages or frameworks.

## How it Works

1. **File Detection**:
   - The `isSpecificScriptFile` method determines if a Groovy file belongs to a specific script type.
   - If a matching detector is found, the corresponding `GroovyScriptType` is returned.

2. **Icon Customization**:
   - The `getScriptIcon` method provides a custom icon for scripts of the detected type.

3. **Scope Adjustment**:
   - The `patchResolveScope` method modifies the resolution scope to include additional dependencies or settings relevant to the script type.

## Implementation Guidelines

To implement a custom `scriptTypeDetector`:

1. Extend the `org.jetbrains.plugins.groovy.extensions.GroovyScriptTypeDetector` class.
2. Provide a custom `GroovyScriptType` implementation with:
   - A unique identifier.
   - Optional custom icon.
   - Scope adjustments (if needed).
3. Override the `isSpecificScriptFile` method to detect relevant files.
4. Register your implementation in the plugin's `plugin.xml` file.

Example:

```xml
<scriptTypeDetector implementation="com.example.MyCustomScriptTypeDetector"/>
```

## Notes

- Ensure that script type detection is efficient to avoid performance overhead.
- Use the `patchResolveScope` method to include additional dependencies only when necessary.
- Provide meaningful icons and identifiers to enhance the user experience.

