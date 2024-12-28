# AST Transformation Support Extension Point

The `astTransformationSupport` extension point in the IntelliJ Platform enables developers to apply Abstract Syntax Tree (AST) transformations to Groovy code. This extension is instrumental in supporting Groovy transformations, injecting synthetic elements, and modifying the behavior of Groovy classes at the AST level.

## Definition

```xml
<extensionPoint name="astTransformationSupport" dynamic="true" interface="org.jetbrains.plugins.groovy.transformations.AstTransformationSupport"/>
```

## Key Classes

### `AstTransformationSupport`
The `AstTransformationSupport` interface provides a method to apply transformations to the AST.

#### Key Method:
- `void applyTransformation(@NotNull TransformationContext context)`
  - Applies transformations to the specified context.

### `TransformationContext`
The `TransformationContext` interface provides a mutable view of a Groovy class, allowing modifications to its structure and properties.

#### Key Methods:
- **Class and Hierarchy Access**:
  - `@NotNull GrTypeDefinition getCodeClass()`
    - Returns the class being transformed.
  - `@NotNull PsiClass getHierarchyView()`
    - Provides a hierarchical view of the class.

- **Field and Method Management**:
  - `void addField(@NotNull GrField field)`
    - Adds a field to the class.
  - `void addMethod(@NotNull PsiMethod method, boolean prepend)`
    - Adds a method to the class, optionally prepending it.
  - `void removeMethod(@NotNull PsiMethod method)`
    - Removes a specified method from the class.

- **Superclasses and Interfaces**:
  - `void setSuperType(@NotNull String fqn)`
    - Sets the superclass by fully qualified name.
  - `void addInterface(@NotNull String fqn)`
    - Adds an interface to the class.

- **Annotations and Modifiers**:
  - `@Nullable PsiAnnotation getAnnotation(@NotNull String fqn)`
    - Retrieves an annotation by its fully qualified name.
  - `void addAnnotation(@NotNull GrAnnotation annotation)`
    - Adds an annotation to the class.
  - `void addModifier(@NotNull GrModifierList modifierList, @NotNull String modifier)`
    - Adds a modifier to the class.

- **Utility Methods**:
  - `boolean isInheritor(@NotNull PsiClass baseClass)`
    - Checks if the class inherits from a specified base class.

## Example Implementation

### `FieldScriptTransformationSupport`
This example implementation demonstrates how to inject fields into a Groovy script class:

```java
public final class FieldScriptTransformationSupport implements AstTransformationSupport {
  @Override
  public void applyTransformation(@NotNull TransformationContext context) {
    if (!(context.getCodeClass() instanceof GroovyScriptClass scriptClass)) return;
    final GroovyFile containingFile = scriptClass.getContainingFile();

    for (GrVariableDeclaration declaration : containingFile.getScriptDeclarations(true)) {
      if (declaration.getModifierList().hasAnnotation(GroovyCommonClassNames.GROOVY_TRANSFORM_FIELD)) {
        for (GrVariable variable : declaration.getVariables()) {
          context.addField(new GrScriptField(variable, scriptClass));
        }
      }
    }
  }
}
```

## Purpose of the Extension

The `astTransformationSupport` extension point enables plugins to:
- Inject synthetic members (fields, methods, annotations) into Groovy classes.
- Modify the structure of Groovy classes at the AST level.
- Enhance support for Groovy transformations and DSLs.

## How it Works

1. **Transformation Application**:
   - The `applyTransformation` method is called for each Groovy class being analyzed or compiled.
   - Implementations inspect and modify the `TransformationContext`.

2. **AST Modifications**:
   - Synthetic fields, methods, and inner classes can be added to the class.
   - Superclasses and interfaces can be updated dynamically.
   - Modifiers and annotations can be injected into the class.

## Implementation Guidelines

To implement a custom `AstTransformationSupport`:

1. Implement the `org.jetbrains.plugins.groovy.transformations.AstTransformationSupport` interface.
2. Define your transformation logic in the `applyTransformation` method.
3. Use the `TransformationContext` to inspect and modify the class structure.
4. Register your implementation in the plugin's `plugin.xml` file.

Example:

```xml
<astTransformationSupport implementation="com.example.MyCustomAstTransformationSupport"/>
```

## Notes

- Ensure that transformations are efficient and avoid unnecessary modifications.
- Use `TransformationContext` methods to make targeted and meaningful changes.
- This extension point is critical for supporting Groovy’s dynamic features and custom DSLs.

