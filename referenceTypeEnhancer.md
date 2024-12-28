# Reference Type Enhancer Extension Point

The `referenceTypeEnhancer` extension point allows developers to augment or customize type inference for Groovy reference expressions. This extension point is particularly useful in dynamic and domain-specific Groovy contexts where standard type inference mechanisms may not capture the full semantic richness of the code.

## Definition

```xml
<extensionPoint name="referenceTypeEnhancer" dynamic="true" interface="org.jetbrains.plugins.groovy.lang.psi.typeEnhancers.GrReferenceTypeEnhancer"/>
```

## Overview

Groovy's dynamic nature often involves working with runtime-generated types, such as map keys, closures, or custom DSL constructs. The `GrReferenceTypeEnhancer` extension point allows plugin developers to inject custom logic for deducing the type of a reference expression in such cases.

### Key Scenarios

1. **DSL Support**: Provide meaningful type information for references in a domain-specific language.
2. **Dynamic Property Resolution**: Infer types for dynamically resolved properties (e.g., map keys or method calls).
3. **Enhanced Code Insight**: Improve completion, navigation, and static analysis by resolving types that would otherwise remain unknown.

## Key Interface

### `GrReferenceTypeEnhancer`

The `GrReferenceTypeEnhancer` interface defines a single method:

#### `getReferenceType`

Determines the type of a given Groovy reference expression.

```java
public abstract @Nullable PsiType getReferenceType(GrReferenceExpression ref, @Nullable PsiElement resolved);
```

- **Parameters**:
  - `ref`: The Groovy reference expression being analyzed.
  - `resolved`: The resolved target of the reference, if available.
- **Returns**: The inferred type of the reference or `null` if no enhancement is applicable.

## Example Implementation

### Groovy Map Value Type Enhancer

This implementation enhances type inference for references that resolve to map properties. It identifies the type of the value associated with a specific map key.

#### Use Case

Consider a Groovy script where a map's keys and values are dynamically determined. For example:

```groovy
config = ["key1": 42, "key2": "value"]
println config.key1
```
The `key1` reference should be inferred as an `Integer`, and `key2` as a `String`.

#### Implementation

```java
public final class GroovyMapValueTypeEnhancer extends GrReferenceTypeEnhancer {
  @Override
  public PsiType getReferenceType(GrReferenceExpression ref, @Nullable PsiElement resolved) {
    if (!(resolved instanceof GroovyMapProperty)) return null;

    GrExpression qualifierExpression = ref.getQualifierExpression();
    if (qualifierExpression == null) return null;

    PsiType mapType = qualifierExpression.getType();

    if (!InheritanceUtil.isInheritor(mapType, CommonClassNames.JAVA_UTIL_MAP)) {
      return null;
    }

    PsiElement qResolved;

    if (qualifierExpression instanceof GrReferenceExpression) {
      qResolved = ((GrReferenceExpression)qualifierExpression).resolve();
    }
    else if (qualifierExpression instanceof GrMethodCall) {
      qResolved = ((GrMethodCall)qualifierExpression).resolveMethod();
    }
    else {
      return null;
    }

    String key = ref.getReferenceName();
    if (key == null) return null;

    for (GroovyMapContentProvider provider : GroovyMapContentProvider.EP_NAME.getExtensions()) {
      PsiType type = provider.getValueType(qualifierExpression, qResolved, key);
      if (type != null) {
        return type;
      }
    }

    if (mapType instanceof GrMapType) {
      return ((GrMapType)mapType).getTypeByStringKey(key);
    }

    return null;
  }
}
```

### Explanation

1. **Qualifying Expression**:
   - The enhancer retrieves the qualifier of the reference (`config` in the example).
   - It ensures the qualifier is a `Map` type.

2. **Map Key Analysis**:
   - The reference name (e.g., `key1`) is used to determine the value type.
   - It leverages custom logic from `GroovyMapContentProvider` or the `GrMapType` structure. (see [Map Content provider](./mapContentProvider.md) )

3. **Dynamic Value Resolution**:
   - For complex cases, the enhancer resolves the value type using domain-specific rules or additional plugins.

## Registration Example

To register a custom `GrReferenceTypeEnhancer` implementation, include the following in your plugin's `plugin.xml` file:

```xml
<referenceTypeEnhancer implementation="com.example.MyReferenceTypeEnhancer"/>
```

## Key Considerations

- **Performance**: Ensure the logic for type inference is efficient, especially when dealing with large or complex Groovy codebases.
- **Fallback**: Return `null` when no enhancement is applicable to allow standard type inference to proceed.
- **Extensions**: Utilize other extension points (e.g., `GroovyMapContentProvider`) for reusable and modular logic.

## Conclusion

The `referenceTypeEnhancer` extension point is a powerful tool for improving type inference in dynamic and domain-specific Groovy scenarios. By implementing a custom enhancer, you can provide richer code insight, enhancing the developer experience in IntelliJ IDEA.

