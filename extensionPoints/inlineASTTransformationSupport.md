# Inline AST Transformation Support Extension Point

The `inlineASTTransformationSupport` extension point in the IntelliJ Platform provides mechanisms to apply inline Abstract Syntax Tree (AST) transformations to Groovy code. Unlike regular AST transformations, this extension deals with modifications to specific code subtrees, such as macros or other localized transformations, enhancing IDE support for such constructs.

## Definition

```xml
<extensionPoint name="inlineASTTransformationSupport" dynamic="true" interface="org.jetbrains.plugins.groovy.transformations.inline.GroovyInlineASTTransformationSupport"/>
```

## Overview

This extension point enables precise control and handling of AST transformations directly within the IDE, ensuring that transformed code behaves seamlessly with features like highlighting, type inference, and completion. To better understand its utility, we illustrate it with an example: **GINQ**.

## Understanding Local vs. Non-Local Transformations

AST transformations in Groovy can be classified as either **local** or **non-local** depending on their scope and impact:

### Local Transformations

- **Scope**: Limited to a specific subtree or expression in the AST.
- **Examples**: Macros, embedded DSLs (e.g., GINQ).
- **Use Case**: When a transformation affects only a single, self-contained part of the AST and does not introduce new members to the containing class.
- **Handled by**: `inlineASTTransformationSupport`.

### Non-Local Transformations

- **Scope**: Affects the structure of the entire class or script.
- **Examples**: AST transformations that add methods, fields, or implement interfaces.
- **Use Case**: When transformations modify the enclosing class structure, such as adding getters/setters or implementing new interfaces.
- **Handled by**: `astTransformationSupport`.

### When to Use Each

- Use **`inlineASTTransformationSupport`** for fine-grained transformations where the transformed code remains part of the same local AST context.
- Use **`astTransformationSupport`** for broader, structural changes to the class or script.

## Example Context: GINQ

### What is GINQ?

GINQ (Groovy Integrated Query) is a query language embedded into Groovy, designed for querying collections and data structures with an SQL-like syntax. For example:

```groovy
def result = from(users)
             .select("name", "age")
             .where { age > 18 }
             .orderBy("age")
```

Although `from`, `select`, and `where` resemble SQL keywords, they are actually valid Groovy method calls. These methods are part of a DSL (Domain-Specific Language) implemented in Groovy, leveraging the language's dynamic capabilities.

GINQ allows developers to write expressive, declarative queries within Groovy. While the syntax is intuitive, it involves complex AST transformations to align the code with Groovy’s execution model. These transformations are supported in the IDE via the `inlineASTTransformationSupport` extension.

## Key Classes

### `GroovyInlineASTTransformationSupport`

The `GroovyInlineASTTransformationSupport` interface defines how specific AST nodes (e.g., GINQ statements) are identified and transformed inline. It works in tandem with:

- **Transformation Roots**: Code elements identified for AST transformations.
- **Performers**: Classes responsible for handling all IDE features for the transformed code subtree.

#### Key Method:

- `@Nullable GroovyInlineASTTransformationPerformer getPerformer(GroovyPsiElement transformationRoot)`
  - Determines if a given element (`transformationRoot`) should be treated as a transformation root. If so, it assigns a `GroovyInlineASTTransformationPerformer` to handle the transformation.

### `GroovyInlineASTTransformationPerformer`

The performer handles all IDE features for the transformed code. This includes:

- Syntax highlighting
- Code completion
- Type inference
- Formatting

For example, GINQ queries require support for SQL-like constructs (`from`, `select`, `where`), inferred field types, and custom formatting rules.

## GINQ Inline Transformation Example

### GINQ Macro Transformation Support

This implementation provides inline AST transformation support for GINQ queries.

#### Implementation

```kotlin
internal class GinqMacroTransformationSupport : GroovyInlineASTTransformationSupport {

  override fun getPerformer(transformationRoot: GroovyPsiElement): GroovyInlineASTTransformationPerformer? {
    if (DumbService.isDumb(transformationRoot.project)) {
      return null
    }
    if (transformationRoot !is GrMethodCall) {
      return null
    }
    val macroMethod = transformationRoot.project.service<GroovyMacroRegistryService>()
      .resolveAsMacro(transformationRoot) ?: return null

    if (macroMethod.name !in GINQ_METHODS || macroMethod.containingClass?.name != "GinqGroovyMethods") return null

    return GinqTransformationPerformer(GinqRootPsiElement.Call(transformationRoot))
  }
}
```

### Responsibilities of the Performer

The `GinqTransformationPerformer` ensures that IDE features align with the transformed representation of the GINQ query. This involves:

1. **Highlighting**:

   - SQL-like constructs (`select`, `where`) are highlighted appropriately as Groovy method calls.

2. **Completion**:

   - Suggestions for valid fields, clauses, and operators during query construction.

3. **Type Inference**:

   - Determines types of query fields (`name`, `age`), enabling accurate inspections and quick fixes.

4. **Formatting**:

   - Ensures consistent indentation and alignment of query clauses.

#### GINQ Transformation Performer Example

**Highlighting**

```kotlin
override fun computeHighlighting(): List<HighlightInfo> {
  val visitor = GinqHighlightingVisitor()
  root.psi.accept(visitor)
  return visitor.keywords.mapNotNull {
    HighlightInfo.newHighlightInfo(HighlightInfoType.INFORMATION)
      .range(it)
      .textAttributes(GroovySyntaxHighlighter.KEYWORD)
      .create()
  }
}
```

**Completion**

```kotlin
override fun computeCompletionVariants(parameters: CompletionParameters, result: CompletionResultSet) {
  val position = parameters.position
  val tree = position.getClosestGinqTree(root)
  if (tree == null) {
    result.addFromSelectShutdown(root, position)
    return
  }
  result.addGinqKeywords(tree, parameters.offset, root, position)
}
```

## Implementation Guidelines

To implement a custom `GroovyInlineASTTransformationSupport`:

1. Implement the `org.jetbrains.plugins.groovy.transformations.inline.GroovyInlineASTTransformationSupport` interface.
2. Override the `getPerformer` method to identify transformation roots.
3. Create a corresponding `GroovyInlineASTTransformationPerformer` to manage IDE features for transformed code.
4. Register the implementation in the plugin’s `plugin.xml` file.

Example:

```xml
<inlineASTTransformationSupport implementation="com.example.MyCustomInlineASTTransformationSupport"/>
```

## Notes

- **Performance Considerations**: Ensure efficient root element identification, especially in large projects.
- **Integration with IDE Features**: The performer handles syntax highlighting, completion, type inference, and formatting for transformed code.
- **Use Case Versatility**: While the example focuses on GINQ, this extension point can be used for other Groovy-specific constructs, such as macros or DSLs.

