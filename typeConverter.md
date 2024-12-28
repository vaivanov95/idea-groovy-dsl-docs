# Type Converter Extension Point

The `typeConverter` extension point in the IntelliJ Platform allows plugins to define custom rules for type conversions in Groovy. It enables the IDE to handle language-specific type conversion scenarios, improve type inference, and validate type compatibility in Groovy code.

## Definition

```xml
<extensionPoint name="typeConverter" dynamic="true" interface="org.jetbrains.plugins.groovy.lang.psi.typeEnhancers.GrTypeConverter"/>
```

## Overview

The `typeConverter` extension point is designed to:

1. **Define Conversion Rules**: Specify custom logic for type conversions in Groovy.
2. **Validate Type Compatibility**: Enhance error-checking for type assignments and method calls.
3. **Integrate with Type Inference**: Provide additional insights for type inference mechanisms in the IDE.

It is especially useful in contexts such as:

- Handling conversions between specific types in DSLs.
- Supporting custom annotations or language features that involve type transformations.
- Extending type inference in Groovy to cover non-standard cases.

## Key Interface

### `GrTypeConverter`

The `GrTypeConverter` abstract class provides methods to define and validate type conversions.

#### 1. `isConvertible`

Determines if one type can be converted to another in a specific context.

```java
public abstract @Nullable ConversionResult isConvertible(@NotNull PsiType targetType,
                                                          @NotNull PsiType actualType,
                                                          @NotNull Position position,
                                                          @NotNull GroovyPsiElement context);
```

- **Parameters**:
  - `targetType`: The desired type after conversion.
  - `actualType`: The type being converted.
  - `position`: The context of the conversion (e.g., method parameter, assignment).
  - `context`: The PSI element where the conversion is taking place.
- **Returns**: A `ConversionResult` indicating if the conversion is valid.

#### 2. `isApplicableTo`

Specifies the positions where this converter applies.

```java
public boolean isApplicableTo(@NotNull Position position) {
    return position == Position.ASSIGNMENT || position == Position.RETURN_VALUE;
}
```

- **Parameters**:
  - `position`: The context of the conversion.
- **Returns**: `true` if the converter is applicable, `false` otherwise.

#### 3. `reduceTypeConstraint`

Reduces type constraints for type inference.

```java
public @Nullable Collection<ConstraintFormula> reduceTypeConstraint(@NotNull PsiType leftType,
                                                                    @NotNull PsiType rightType,
                                                                    @NotNull Position position,
                                                                    @NotNull PsiElement context) {
    return null;
}
```

- **Parameters**:
  - `leftType`: The expected type.
  - `rightType`: The actual type.
  - `position`: The context of the conversion.
  - `context`: The PSI element where the conversion is taking place.
- **Returns**: A collection of `ConstraintFormula` objects representing reduced constraints, or `null` if not applicable.

#### `Position` Enum

Defines the context of the conversion:

- `EXPLICIT_CAST`: An explicit type cast.
- `ASSIGNMENT`: A variable assignment.
- `METHOD_PARAMETER`: A method call argument.
- `GENERIC_PARAMETER`: A generic type parameter.
- `RETURN_VALUE`: A method or closure return value.

## Example Implementations

### 1. Named Parameter Converter

This converter handles named parameters in Groovy methods annotated with `@NamedParams`.

#### Use Case

- Converts `GrMapTypeFromNamedArgs` (a map of named arguments) to the `Map` type expected by a method.
- Validates that required named parameters are present.

#### Implementation

```kotlin
class GrNamedParamsConverter : GrTypeConverter() {

  override fun isApplicableTo(position: Position): Boolean {
    return position == Position.METHOD_PARAMETER
  }

  override fun reduceTypeConstraint(leftType: PsiType, rightType: PsiType, position: Position, context: PsiElement): Collection<ConstraintFormula>? {
    val result = createConversion(leftType, rightType, context)
    return if (result is Result.Success) listOf(result.typeConstraint) else null
  }

  private fun createConversion(targetType: PsiType, actualType: PsiType, context: PsiElement): Result {
    if (actualType !is GrMapTypeFromNamedArgs || targetType !is PsiClassType || !PsiTypesUtil.classNameEquals(targetType, CommonClassNames.JAVA_UTIL_MAP)) {
      return Result.NotNamedParamsError
    }

    val namedParamList = collectNamedParams(context.resolveMethod() ?: return Result.NotNamedParamsError)
    for (param in namedParamList) {
      if (param.required && param.name !in actualType.stringKeys) {
        return Result.IncompatibleTypesError
      }
    }

    return Result.Success(TypeConstraint(targetType, targetType, context))
  }

  override fun isConvertible(targetType: PsiType, actualType: PsiType, position: Position, context: GroovyPsiElement): ConversionResult? {
    return when (createConversion(targetType, actualType, context)) {
      Result.NotNamedParamsError -> null
      Result.IncompatibleTypesError -> ConversionResult.ERROR
      else -> ConversionResult.OK
    }
  }

  private abstract class Result {
    object NotNamedParamsError : Result()
    object IncompatibleTypesError : Result()
    data class Success(val typeConstraint: TypeConstraint) : Result()
  }
}
```

### 2. Number Converter

This converter handles numeric conversions in Groovy, including `BigDecimal` and `BigInteger`.

#### Use Case

- Validates numeric conversions in assignments, method calls, and type casts.

#### Implementation

```java
public final class GrNumberConverter extends GrTypeConverter {

  @Override
  public boolean isApplicableTo(@NotNull Position position) {
    return true;
  }

  @Override
  public @Nullable ConversionResult isConvertible(@NotNull PsiType targetType,
                                                  @NotNull PsiType actualType,
                                                  @NotNull Position position,
                                                  @NotNull GroovyPsiElement context) {
    if (position == Position.METHOD_PARAMETER) {
      return methodParameterConvert(targetType, actualType);
    }
    if (TypesUtil.isNumericType(targetType) && TypesUtil.isNumericType(actualType)) {
      return ConversionResult.OK;
    }
    return null;
  }

  private static ConversionResult methodParameterConvert(PsiType targetType, PsiType actualType) {
    if (TypesUtil.isClassType(actualType, JAVA_MATH_BIG_DECIMAL)) {
      return isFloatOrDoubleType(targetType) ? ConversionResult.OK : null;
    }
    return null;
  }
}
```

## Registration

To register a `GrTypeConverter` implementation, add the following to your `plugin.xml`:

```xml
<typeConverter implementation="com.example.MyTypeConverter"/>
```

## Conclusion

The `typeConverter` extension point provides a powerful mechanism to define custom type conversion rules in Groovy. By leveraging this extension point, plugins can improve type inference, validation, and compatibility checks, making it invaluable for DSLs and advanced Groovy features.

