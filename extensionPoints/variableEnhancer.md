# Variable Enhancer Extension Point

The `variableEnhancer` extension point in the IntelliJ Platform allows developers to enhance the inferred types of Groovy variables dynamically. This capability is particularly useful in scenarios involving DSLs (Domain-Specific Languages) where variable types may not be explicitly defined in the code but are implied by context.

## Definition

```xml
<extensionPoint name="variableEnhancer" dynamic="true" interface="org.jetbrains.plugins.groovy.lang.psi.typeEnhancers.GrVariableEnhancer"/>
```

## Overview

### Purpose
In Groovy-based DSLs, variable types often depend on runtime behavior or conventions rather than explicit type declarations. For example:
- A parameter in a closure might derive its type from the context in which the closure is invoked.
- A variable in a DSL block may represent a custom object whose type is determined by the DSL logic.

The `variableEnhancer` extension point enables developers to augment IntelliJ's type inference system, providing accurate type information for such variables. This ensures better code assistance, including syntax highlighting, code completion, and inspections.

### How It Works
When IntelliJ encounters a Groovy variable, it consults all registered `GrVariableEnhancer` implementations to determine if an enhanced type can be inferred. The process continues until one enhancer returns a non-null type or all enhancers are consulted.

## Key Interface

### `GrVariableEnhancer`

The `GrVariableEnhancer` interface defines a single method:

```java
public abstract @Nullable PsiType getVariableType(GrVariable variable);
```

- **Parameters**:
  - `variable`: The Groovy variable whose type is being inferred. This can include parameters, local variables, and synthetic variables such as closure parameters.
- **Returns**: The inferred `PsiType` for the variable, or `null` if the enhancer does not provide a type.

#### Static Utility Method

```java
public static @Nullable PsiType getEnhancedType(final GrVariable variable) {
    for (GrVariableEnhancer enhancer : EP_NAME.getExtensions()) {
        final PsiType type = enhancer.getVariableType(variable);
        if (type != null) {
            return type;
        }
    }
    return null;
}
```

This method simplifies the process of retrieving the enhanced type by iterating through all registered enhancers.

## Example Implementation

### Abstract Closure Parameter Enhancer

One common use case is inferring types for closure parameters in Groovy DSLs. Parameters in closures often derive their types from the calling context.

```java
public abstract class AbstractClosureParameterEnhancer extends GrVariableEnhancer {

  @Override
  public final PsiType getVariableType(GrVariable variable) {
    if (!(variable instanceof GrParameter)) {
      return null;
    }

    GrFunctionalExpression functionalExpression;
    int paramIndex;

    if (variable instanceof ClosureSyntheticParameter) {
      functionalExpression = ((ClosureSyntheticParameter) variable).getClosure();
      paramIndex = 0;
    } else {
      PsiElement eParameterList = variable.getParent();
      if (!(eParameterList instanceof GrParameterList parameterList)) return null;

      PsiElement eFunctionalExpression = eParameterList.getParent();
      if (!(eFunctionalExpression instanceof GrFunctionalExpression)) return null;

      functionalExpression = (GrFunctionalExpression) eFunctionalExpression;
      paramIndex = parameterList.getParameterNumber((GrParameter) variable);
    }

    PsiType res = getClosureParameterType(functionalExpression, paramIndex);
    return res instanceof PsiPrimitiveType ? ((PsiPrimitiveType) res).getBoxedType(functionalExpression) : res;
  }

  protected abstract @Nullable PsiType getClosureParameterType(@NotNull GrFunctionalExpression closure, int index);
}
```

### Use Case: Groovy DSLs
In many Groovy DSLs, closures are used to configure objects. For example:

```groovy
person {
  name 'John Doe'
  age 30
}
```

Here, the `person` block defines a closure where `name` and `age` represent methods or properties of a `Person` object. The types of these variables can be inferred based on the DSL structure.

#### Implementation for a DSL

```java
public class DslVariableEnhancer extends AbstractClosureParameterEnhancer {

  @Override
  protected @Nullable PsiType getClosureParameterType(@NotNull GrFunctionalExpression closure, int index) {
    // Example: Check if the closure is configuring a Person object
    if (closure.getText().contains("person")) {
      return JavaPsiFacade.getInstance(closure.getProject())
                          .getElementFactory()
                          .createTypeByFQClassName("com.example.Person", closure.getResolveScope());
    }
    return null;
  }
}
```

## Registration Example

To register a custom `GrVariableEnhancer` implementation, include the following in your plugin's `plugin.xml` file:

```xml
<variableEnhancer implementation="com.example.DslVariableEnhancer"/>
```

## Benefits for DSLs
- **Enhanced Code Assistance**: Accurate type inference ensures better autocompletion and code navigation.
- **Error Detection**: Identifying type mismatches early during development.
- **Improved Readability**: Developers can understand variable types directly from IDE hints.

## Key Considerations
1. **Performance**: Ensure your implementation is efficient, especially for large projects.
2. **Context Awareness**: Enhance types only where it makes sense to avoid unnecessary computations.
3. **Fallbacks**: Return `null` for unsupported cases to allow other enhancers to process the variable.

## Conclusion
The `variableEnhancer` extension point is a powerful tool for enhancing type inference in Groovy, particularly for DSLs where types are often implicit. By providing accurate type information, it bridges the gap between Groovy's dynamic nature and IntelliJ's static analysis capabilities, delivering a seamless development experience.

