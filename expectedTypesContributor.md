# Groovy Expected Types Contributor Extension Point

The `expectedTypesContributor` extension point in IntelliJ IDEA enables plugins to provide context-aware type expectations for Groovy expressions. This extension is crucial for enhancing IDE features like code completion, type inference, and inspections, particularly in dynamic or loosely typed environments like Groovy DSLs.

## Extension Point Definition

```xml
<extensionPoint name="expectedTypesContributor" dynamic="true"
                interface="org.jetbrains.plugins.groovy.lang.psi.expectedTypes.GroovyExpectedTypesContributor"/>
```

## Overview

This extension point allows contributors to define expected types for a given Groovy expression. It is particularly useful when type constraints can be derived from the context, such as method arguments, closures, or collection initializations.

### Use Cases

1. **Dynamic Type Environments**: Enhance type inference in DSLs or Groovy scripts.
2. **Code Completion**: Provide better suggestions based on the expected types.
3. **Inspections**: Detect type mismatches in dynamic scenarios.

## Key Interfaces and Classes

### `GroovyExpectedTypesContributor`

The main interface for contributors. Implementations define how to calculate type constraints for a given expression.

```java
public abstract class GroovyExpectedTypesContributor {
  public abstract List<TypeConstraint> calculateTypeConstraints(@NotNull GrExpression expression);
}
```

- **`calculateTypeConstraints`**:
  - **Parameters**: A Groovy expression for which type constraints are to be calculated.
  - **Returns**: A list of `TypeConstraint` objects that describe the expected types.

### `TypeConstraint`

Defines a type constraint and provides methods to check satisfaction and obtain default types.

```java
public abstract class TypeConstraint {
  public abstract boolean satisfied(PsiType type, @NotNull PsiElement context);
  public abstract @NotNull PsiType getDefaultType();
}
```

- **`satisfied`**: Checks if a given type satisfies the constraint.
- **`getDefaultType`**: Provides a default type if no specific type is inferred.

### `SubtypeConstraint`

A common implementation of `TypeConstraint` that ensures the expected type is a subtype of a specified type.

```java
public final class SubtypeConstraint extends TypeConstraint {
  public boolean satisfied(PsiType type, @NotNull PsiElement context) {
    return TypesUtil.isAssignableByMethodCallConversion(getType(), type, context);
  }
}
```

### `SupertypeConstraint`

Ensures the expected type is a supertype of a specified type.

```java
public class SupertypeConstraint extends TypeConstraint {
  public boolean satisfied(PsiType type, @NotNull PsiElement context) {
    return TypesUtil.isAssignableByMethodCallConversion(type, getType(), context);
  }
}
```

## Example Implementation

### `GppExpectedTypesContributor`

This implementation enhances type inference for Groovy DSLs like GPP (Groovy Parallel Patterns), which use tuples and lists extensively.

#### Code Example

```java
final class GppExpectedTypesContributor extends GroovyExpectedTypesContributor {
  @Override
  public List<TypeConstraint> calculateTypeConstraints(@NotNull GrExpression expression) {
    final PsiElement parent = expression.getParent();
    if (parent instanceof GrListOrMap list) {
      if (!list.isMap()) {
        final PsiType listType = list.getType();
        if (!(listType instanceof GrTupleType)) {
          return Collections.emptyList();
        }
        return addExpectedConstructorParameters(list, expression);
      }
    }
    return Collections.emptyList();
  }

  private static List<TypeConstraint> addExpectedConstructorParameters(GrListOrMap list, GrExpression arg) {
    GroovyConstructorReference reference = list.getConstructorReference();
    if (reference == null) {
      return Collections.emptyList();
    }

    GrExpression[] args = list.getInitializers();
    List<TypeConstraint> result = new ArrayList<>();
    for (GroovyResolveResult constructorResult : reference.resolve(false)) {
      final Map<GrExpression, Pair<PsiParameter, PsiType>> map = GrClosureSignatureUtil.mapArgumentsToParameters(
        constructorResult, list, false, true, GrNamedArgument.EMPTY_ARRAY, args, GrClosableBlock.EMPTY_ARRAY
      );
      if (map == null) {
        continue;
      }
      final Pair<PsiParameter, PsiType> pair = map.get(arg);
      if (pair == null) {
        continue;
      }
      result.add(SubtypeConstraint.create(pair.second));
    }
    return result;
  }
}
```

### What is GPP?

**GPP (Groovy Parallel Patterns)** is a Groovy-based DSL for parallel programming. It provides patterns like pipelines and farms to simplify concurrent programming. Lists and tuples in GPP often represent structured data or arguments for parallel constructs.

## Registration

To register a custom `GroovyExpectedTypesContributor`, add the following entry to your `plugin.xml`:

```xml
<expectedTypesContributor implementation="com.example.MyExpectedTypesContributor"/>
```

## Conclusion

The `expectedTypesContributor` extension point enhances type inference and IDE features for Groovy. By providing context-aware type constraints, it significantly improves developer productivity and code quality, especially in DSLs like GPP.

