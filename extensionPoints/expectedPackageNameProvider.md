# Expected Package Name Provider Extension Point

The `expectedPackageNameProvider` extension point is designed to infer the package name of a Groovy file. This can enhance features such as refactoring, file creation, and code generation, where an accurate package name is necessary.

---

## Extension Point Declaration

```xml
<extensionPoint name="expectedPackageNameProvider" dynamic="true"
                interface="org.jetbrains.plugins.groovy.lang.resolve.ExpectedPackageNameProvider"/>
```

---

## Key Interface: `ExpectedPackageNameProvider`

This interface defines the contract for extensions to infer the package name of a given Groovy file. Implementations can provide custom logic to derive the package name based on specific criteria.

### Interface Definition

```kotlin
interface ExpectedPackageNameProvider {
  fun inferPackageName(file: GroovyFile): String?
}
```

### Default Implementation: `DefaultExpectedPackageNameProvider`

The default implementation uses the `JavaDirectoryService` to infer the package name based on the file's containing directory.

#### Code Example

```kotlin
class DefaultExpectedPackageNameProvider : ExpectedPackageNameProvider {

  override fun inferPackageName(file: GroovyFile): String? = file.containingDirectory?.let {
    JavaDirectoryService.getInstance().getPackage(it)?.qualifiedName
  }
}
```

#### Behavior
- If the file is located in a directory that corresponds to a Java package, the package name is derived using `JavaDirectoryService`.
- Returns `null` if the directory does not map to a package.

---

## Example Usage

The utility function `inferExpectedPackageName` combines all registered `ExpectedPackageNameProvider` implementations to infer the package name.

### Code Example

```kotlin
private val EP_NAME = ExtensionPointName.create<ExpectedPackageNameProvider>("org.intellij.groovy.expectedPackageNameProvider")

fun inferExpectedPackageName(file: GroovyFile): @NlsSafe String {
  for (ext in EP_NAME.extensions) {
    val name = ext.inferPackageName(file) ?: continue
    return name
  }
  return ""
}
```

### Behavior
1. Iterates over all registered `ExpectedPackageNameProvider` extensions.
2. Calls `inferPackageName` on each provider.
3. Returns the first non-null package name inferred by the providers.
4. Defaults to an empty string if no providers infer a package name.

---

## Potential Use Cases

### 1. **Custom Directory-to-Package Mapping**
Extensions can implement custom mappings from directories to package names, enabling support for non-standard project structures or specialized frameworks.

### 2. **Integration with Build Tools**
Providers can integrate with build tools like Gradle to derive package names based on source sets or other build configuration metadata.

### 3. **Enhanced Refactoring Support**
During refactoring operations, accurate package inference ensures that files are placed correctly in the new structure.

### 4. **Code Generation**
Tools that generate Groovy code can use this extension point to determine the appropriate package for newly created files.

---

## Example Custom Implementation

An implementation that derives the package name based on a project-specific convention:

```kotlin
class CustomConventionPackageNameProvider : ExpectedPackageNameProvider {
  override fun inferPackageName(file: GroovyFile): String? {
    val projectRoot = file.project.baseDir
    val relativePath = file.virtualFile?.path?.removePrefix(projectRoot.path ?: "") ?: return null

    return relativePath
      .removePrefix("/src/main/groovy/")
      .replace("/", ".")
      .takeIf { it.isNotEmpty() }
  }
}
```

### Behavior
- Assumes Groovy files are located under `/src/main/groovy/`.
- Derives the package name by removing the base path and converting directory separators to dots.

---

## Conclusion

The `expectedPackageNameProvider` extension point provides a flexible mechanism to infer package names for Groovy files. By leveraging custom implementations, developers can tailor package name inference to suit project-specific needs, enhancing the IDE's intelligence and usability.

