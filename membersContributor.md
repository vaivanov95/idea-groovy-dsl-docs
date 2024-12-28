# Non-Code Members Contributor Extension Point

The `membersContributor` extension point in the IntelliJ Platform allows developers to extend Groovy programs with custom properties and methods. This is particularly useful for implementing custom resolution algorithms and providing dynamic code completion suggestions.

## Definition

```xml
<extensionPoint name="membersContributor" dynamic="true" interface="org.jetbrains.plugins.groovy.lang.resolve.NonCodeMembersContributor"/>
```

## Example Instances

Below are example implementations of the `membersContributor` extension point:

```xml
<membersContributor implementation="org.jetbrains.plugins.groovy.ext.newify.NewifyMemberContributor"/>
<membersContributor implementation="org.jetbrains.plugins.groovy.swingBuilder.SwingBuilderNonCodeMemberContributor"/>
<membersContributor implementation="org.jetbrains.plugins.groovy.lang.resolve.GrInterfaceDefaultMethodMemberContributor"/>
<membersContributor implementation="org.jetbrains.plugins.groovy.builder.XmlMarkupBuilderNonCodeMemberContributor"/>
<membersContributor implementation="org.jetbrains.plugins.groovy.dsl.GdslMemberContributor" order="last"/>
```

## Key Method

Implementations of this extension point must provide the following method:

```java
public void processDynamicElements(@NotNull PsiType qualifierType,
                                    @Nullable PsiClass aClass,
                                    @NotNull PsiScopeProcessor processor,
                                    @NotNull PsiElement place,
                                    @NotNull ResolveState state);
```

### Parameters:
- **`qualifierType`**: The type of the qualifier in the code being analyzed.
- **`aClass`**: The class associated with the qualifier type (if applicable).
- **`processor`**: The processor responsible for handling the resolved elements.
- **`place`**: The location in the code where resolution is being performed.
- **`state`**: The state of the resolution process.

## Purpose of the Extension

The `membersContributor` extension point enables IntelliJ IDEA to dynamically inject and resolve additional methods and properties into Groovy programs. This is essential for supporting dynamic features of Groovy or domain-specific enhancements, such as:
- **Custom Code Completion**: Providing additional suggestions during code completion.
- **Dynamic Member Resolution**: Resolving methods and properties not explicitly declared in the code.
- **Support for DSLs**: Extending Groovy with domain-specific features such as builders and annotations.

## Built-in Implementations

### `org.jetbrains.plugins.groovy.ext.newify.NewifyMemberContributor`
This contributor supports Groovy's `@Newify` annotation, which allows creating simplified constructors for classes.

#### Highlights:
- Resolves dynamically created constructor methods.
- Processes the `@Newify` annotation to infer class constructors based on the annotation's parameters.

### `org.jetbrains.plugins.groovy.swingBuilder.SwingBuilderNonCodeMemberContributor`
This contributor extends the Groovy SwingBuilder DSL, enabling the dynamic creation of UI components in Groovy scripts.

#### Highlights:
- Injects dynamic methods for creating and configuring Swing components (e.g., `button`, `frame`, `panel`).
- Provides support for nested structures and builder patterns common in UI design.

### `org.jetbrains.plugins.groovy.builder.XmlMarkupBuilderNonCodeMemberContributor`
This contributor enhances Groovy's `MarkupBuilder` DSL, enabling the creation of XML structures programmatically.

#### Highlights:
- Dynamically resolves methods for XML elements.
- Supports method overloads for element attributes and nested elements.

### `org.jetbrains.plugins.groovy.dsl.GdslMemberContributor`
This contributor processes Groovy DSL scripts (GDSL) to inject dynamic methods and properties into the code.

#### Highlights:
- Provides support for custom DSLs defined by GDSL scripts.
- Handles dynamic resolution for methods and properties defined in these scripts.

## How it Works

The `membersContributor` extension point integrates into IntelliJ IDEA’s resolution and code analysis processes. It works by dynamically injecting properties and methods into Groovy code at runtime. This process involves:

1. **Processing Resolution Hints**:
   - Contributors analyze hints provided by the `PsiScopeProcessor` (e.g., name or kind of the element being resolved) to decide which elements to offer.
2. **Injecting Members**:
   - Based on the resolution context, contributors inject additional properties or methods that match the given hints.
3. **Handling Completion**:
   - During code completion, contributors can provide all possible dynamic members if no specific hints are available.

## Implementation Guidelines

To implement a custom `membersContributor`:

1. Extend the `org.jetbrains.plugins.groovy.lang.resolve.NonCodeMembersContributor` class.
2. Override the `processDynamicElements` method to inject custom members dynamically.
3. Use resolution hints (`NameHint`, `ElementClassHint`) to filter and optimize the injected elements.
4. Register your implementation in the plugin's `plugin.xml` file.

Example:

```xml
<membersContributor implementation="com.example.MyCustomMembersContributor" order="before default"/>
```

## Notes

- Contributors must ensure performance by injecting only relevant elements based on the resolution hints.
- Changes to contributors should clear the internal cache to ensure the updated logic is applied.
- This extension point is crucial for enhancing the Groovy language's dynamic and flexible nature in IntelliJ IDEA.

