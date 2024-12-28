# Rename Helper Extension Point

The `renameHelper` extension point in the IntelliJ Platform allows developers to customize and enhance the behavior of the rename refactoring operation for Groovy elements. It provides mechanisms to handle specific naming conventions, such as those introduced by traits or generated methods (e.g., accessors), ensuring that renaming is consistent and meaningful within the Groovy language context.

## Definition

```xml
<extensionPoint name="renameHelper" dynamic="true" interface="org.jetbrains.plugins.groovy.refactoring.rename.GrRenameHelper"/>
```

## Overview

This extension point is particularly useful for:

- Handling renaming of generated members (e.g., getters, setters, or trait-related fields).
- Determining whether a qualified name is necessary after a rename.
- Customizing rename behavior for specific constructs in Groovy.

The extension point defines methods to adjust the behavior of renaming operations, providing hooks to:

1. Suggest new member names for generated elements.
2. Decide whether a qualification is needed after renaming.

## Key Interface

### `GrRenameHelper`

The `GrRenameHelper` interface provides two main methods:

#### 1. `getNewMemberName`

Determines the new name of a generated member (e.g., accessor methods) after the original member is renamed.

```java
default @Nullable String getNewMemberName(@NotNull PsiMember member, @NotNull String newOriginalName) {
    return null;
}
```

- **Parameters**:
  - `member`: The member being referenced (e.g., a getter method).
  - `newOriginalName`: The new name for the original element (e.g., the field).
- **Returns**: The new name of the generated member (e.g., `getBar` for a field renamed to `bar`).

#### 2. `isQualificationNeeded`

Checks whether a qualified name is required after renaming.

```java
default boolean isQualificationNeeded(@NotNull PsiManager manager, @NotNull PsiElement before, @NotNull PsiElement after) {
    return false;
}
```

- **Parameters**:
  - `manager`: The `PsiManager` instance.
  - `before`: The element referenced before renaming.
  - `after`: The element referenced after renaming.
- **Returns**: `true` if qualification is necessary, `false` otherwise.

## Example Implementations

### 1. Default Rename Helper

Handles renaming of standard generated members, such as accessors (getters and setters).

#### Use Case

When renaming a field in Groovy, its associated accessor methods (`getFoo`, `setFoo`) must also update accordingly.

#### Implementation

```kotlin
class DefaultRenameHelper : GrRenameHelper {

  override fun getNewMemberName(member: PsiMember, newOriginalName: String): String? {
    if (member !is GrAccessorMethod) return null;
    return if (member.isSetter) {
      getSetterName(newOriginalName)
    }
    else {
      getAccessorName(if (member.name.startsWith("is")) "is" else "get", newOriginalName)
    }
  }

  override fun isQualificationNeeded(manager: PsiManager,
                                     before: PsiElement,
                                     after: PsiElement): Boolean {
    return before is GrAccessorMethod && (after !is GrAccessorMethod || !manager.areElementsEquivalent(before.property, after.property));
  }
}
```

### 2. Trait Rename Helper

Handles renaming of members in Groovy traits, ensuring that naming conventions (e.g., trait-specific prefixes) are preserved.

#### Use Case

In Groovy traits, fields are prefixed with the trait's name to avoid conflicts. This helper updates the generated member names accordingly.

#### Implementation

```kotlin
class TraitRenameHelper : GrRenameHelper {

  override fun getNewMemberName(member: PsiMember, newOriginalName: String): String? {
    if (member !is GrTraitField) return null;
    val prototype = member.prototype;
    val containingClass = prototype.containingClass ?: return null;
    return GrTraitUtil.getTraitFieldPrefix(containingClass) + newOriginalName;
  }

  override fun isQualificationNeeded(manager: PsiManager,
                                     before: PsiElement,
                                     after: PsiElement): Boolean {
    return before is GrTraitField && (after !is GrTraitField || !manager.areElementsEquivalent(after.prototype, before.prototype));
  }
}
```

## Registration Example

To register a custom `GrRenameHelper` implementation, include the following in your plugin's `plugin.xml` file:

```xml
<renameHelper implementation="com.example.MyCustomRenameHelper"/>
```

## Key Considerations

1. **Generated Members**: Ensure that renaming generated members like accessors or trait-prefixed fields is handled correctly to maintain consistency.
2. **Qualification Rules**: Handle scenarios where fully qualified names are required after renaming, especially in ambiguous contexts.
3. **Performance**: Optimize for efficiency, especially in large projects with complex rename operations.

## Conclusion

The `renameHelper` extension point provides essential hooks for customizing rename refactoring in Groovy, ensuring that language-specific constructs are handled gracefully. Whether dealing with generated members or trait fields, it ensures renaming operations are intuitive and error-free.

