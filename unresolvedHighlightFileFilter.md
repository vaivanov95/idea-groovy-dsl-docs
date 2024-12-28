# Unresolved Highlight File Filter Extension Point

The `unresolvedHighlightFileFilter` extension point allows plugin developers to suppress IDE highlights for unresolved references at the file level in Groovy. This capability is particularly useful for managing dynamic constructs or domain-specific languages (DSLs) where unresolved references might be expected or desirable.

---

## Extension Point Declaration

```xml
<extensionPoint name="unresolvedHighlightFileFilter"
                dynamic="true"
                interface="org.jetbrains.plugins.groovy.extensions.GroovyUnresolvedHighlightFileFilter"/>
```

---

## Purpose and Capabilities

### **1. File-Level Suppression of Highlighting**
This extension point enables plugin developers to:
- Suppress unresolved reference warnings for specific file types.
- Implement custom logic to determine whether unresolved highlights should be displayed for an entire file.

### **2. Tailored IDE Experience for Custom DSLs**
Dynamic or custom DSLs often introduce constructs that are not resolved by static analysis. Highlighting these as unresolved could:
- Confuse developers.
- Diminish the usability of the IDE for these languages.

This extension point allows developers to suppress such warnings on a file-by-file basis.

---

## Key Interface

### `GroovyUnresolvedHighlightFileFilter`

This abstract class defines the contract for determining whether unresolved reference highlighting should be suppressed for a given file.

```java
public abstract class GroovyUnresolvedHighlightFileFilter {

  public static final ExtensionPointName<GroovyUnresolvedHighlightFileFilter> EP_NAME =
      ExtensionPointName.create("org.intellij.groovy.unresolvedHighlightFileFilter");

  public abstract boolean isReject(@NotNull PsiFile file);
}
```

#### Key Method:
- **`isReject(PsiFile file)`**: Determines whether unresolved references in the specified file should be suppressed.

---

## Example Use Case: DSL File Suppression

Although no direct usages are available, here’s an example of how this extension point might be implemented to suppress unresolved highlights for custom DSL files.

### Hypothetical Implementation: Suppressing for DSL Files

Consider a custom DSL where unresolved references are expected as part of the syntax:

```java
public final class CustomDSLHighlightFilter extends GroovyUnresolvedHighlightFileFilter {

  @Override
  public boolean isReject(@NotNull PsiFile file) {
    // Check if the file belongs to the custom DSL file type
    return file.getFileType().getName().equals("CustomDSL");
  }
}
```

### Capabilities:
1. **Custom DSL Support**:
   - Prevents unresolved references in DSL files from being flagged unnecessarily.
   - Allows developers to work with DSLs without being distracted by irrelevant warnings.

2. **Flexible Logic**:
   - Can be extended to include additional checks, such as file content or specific project configurations.

---

## Use Cases and Benefits

### **1. Simplifying Development for Custom DSLs**
For DSLs or frameworks built on Groovy, unresolved references might be expected or even intentional. This extension point ensures that the IDE does not flag such references, improving usability.

### **2. Enhanced Developer Productivity**
By suppressing irrelevant warnings at the file level, developers can:
- Focus on actual issues.
- Avoid unnecessary distractions from false positives.

### **3. Broad Application Across Groovy Ecosystems**
This extension point is valuable for:
- Frameworks like Gradle or custom Groovy DSLs.
- Any project where unresolved references in certain file types are acceptable.

---

## Conclusion

The `unresolvedHighlightFileFilter` extension point provides a mechanism for tailoring IDE behavior to Groovy’s dynamic and flexible nature. By suppressing unresolved reference highlights at the file level, it supports custom DSLs, runtime constructs, and domain-specific frameworks, creating a more intuitive and productive development environment.

