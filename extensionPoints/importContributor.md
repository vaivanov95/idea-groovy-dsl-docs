# Import Contributor Extension Point

The `importContributor` extension point in the IntelliJ Platform allows developers to dynamically add imports to Groovy files. This is particularly useful for frameworks or plugins that want to provide default imports for specific use cases, such as Gradle build scripts or domain-specific languages.

## Definition

```xml
<extensionPoint name="importContributor" dynamic="true" interface="org.jetbrains.plugins.groovy.lang.resolve.imports.GrImportContributor"/>
```

## Example Instances

Below are example implementations of the `importContributor` extension point:

```xml
<importContributor implementation="org.jetbrains.plugins.gradle.config.GradleDefaultImportContributor"/>
```

## Key Method

Implementations of this extension point must provide the following method:

```java
@NotNull
List<GroovyImport> getFileImports(@NotNull GroovyFile file);
```

### Parameters:
- **`file`**: The Groovy file for which imports are being resolved.

### Return Value:
A list of `GroovyImport` objects representing the additional imports to be injected into the file.

## Purpose of the Extension

The `importContributor` extension point is used to enhance the resolution of classes and methods in Groovy files by dynamically adding imports. This enables:
- **Framework Support**: Automatically adding default imports for frameworks such as Gradle.
- **Improved Code Completion**: Enhancing the IDE experience by preloading commonly used classes and packages.
- **Custom Language Enhancements**: Supporting domain-specific languages (DSLs) that require specific imports.

## Built-in Implementations

### `org.jetbrains.plugins.gradle.config.GradleDefaultImportContributor`
This contributor adds default imports to Gradle build scripts, as defined in the [Gradle User Guide](https://docs.gradle.org/current/userguide/writing_build_scripts.html#script-default-imports).

#### Highlights:
- Injects a wide range of Gradle-related packages such as:
  - `org.gradle.api`
  - `org.gradle.api.tasks`
  - `org.gradle.api.plugins`
- Ensures that these packages are available by default in Gradle script files, improving the developer experience.

#### Logic:
- Checks if the Groovy file is a Gradle script by evaluating its script type.
- Adds a predefined list of `StarImport` objects, each representing a package to be imported.

## How it Works

The `importContributor` extension point integrates into IntelliJ IDEA’s Groovy resolution process. It works as follows:

1. **File Evaluation**:
   - The `getFileImports` method is called with a Groovy file.
   - Implementations determine if they should provide imports based on the file's context (e.g., script type).

2. **Dynamic Import Injection**:
   - If applicable, the contributor provides a list of imports (e.g., star imports or specific class imports) to be injected into the file.

3. **Improved Resolution**:
   - The injected imports enhance class and method resolution, code completion, and error analysis.

## Implementation Guidelines

To implement a custom `importContributor`:

1. Extend the `org.jetbrains.plugins.groovy.lang.resolve.imports.GrImportContributor` interface.
2. Implement the `getFileImports` method to provide the required imports based on the file's context.
3. Register your implementation in the plugin's `plugin.xml` file.

Example:

```xml
<importContributor implementation="com.example.MyCustomImportContributor"/>
```

## Notes

- Ensure that contributors only provide imports for relevant files to avoid unnecessary overhead.
- Use `StarImport` for package-level imports and `StaticImport` for static members to be imported.
- Contributors should align their behavior with the specific needs of their target frameworks or DSLs to provide meaningful enhancements.

