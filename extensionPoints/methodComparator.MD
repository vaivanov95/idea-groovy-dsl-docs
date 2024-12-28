# Method Comparator Extension Point

The `methodComparator` extension point in the IntelliJ Platform provides a mechanism for defining custom logic to compare and prioritize methods during Groovy method resolution.

## Definition

```xml
<extensionPoint name="methodComparator" dynamic="true" interface="org.jetbrains.plugins.groovy.lang.resolve.GrMethodComparator"/>
```

## Example Instances

Below are example implementations of the `methodComparator` extension point:

```xml
<methodComparator implementation="org.jetbrains.plugins.groovy.lang.resolve.DGMGetAtMethodFilter" order="last"/>
<methodComparator implementation="org.jetbrains.plugins.groovy.lang.resolve.GrDefaultMethodComparator" order="last"/>
```

## Key Method

Implementations of this extension point must provide the following method:

```java
Boolean dominated(GroovyMethodResult result1, GroovyMethodResult result2, Context context);
```

### Parameters:
- **`context`**: Includes information about the place where the comparison is being made (e.g., arguments, call site, or surrounding code context).
- **`result1` and `result2`**: Represent the two methods being compared to determine which is more suitable.

The return value indicates whether `result1` "dominates" over `result2` based on the custom comparison logic.

## Purpose of the Extension

The `methodComparator` extension point is used to customize or extend the default logic for method resolution in Groovy. This is particularly useful for:
- **Customizing priority rules** for specific methods in specific contexts.
- **Overriding default Groovy method resolution behavior** to handle specific edge cases or introduce new language features.

## Built-in Implementations

### `org.jetbrains.plugins.groovy.lang.resolve.DGMGetAtMethodFilter`
This implementation lowers the priority of `DefaultGroovyMethods.getAt(Object, String)` and `DefaultGroovyMethods.putAt(Object, String, Object)`. It ensures these methods are deprioritized when other applicable methods exist.

### `org.jetbrains.plugins.groovy.lang.resolve.GrDefaultMethodComparator`
This is the default implementation of the method comparator, which provides logic closely aligned with Groovy's compiler rules. Key features include:

- **Parameter Matching**:
  - Prefers methods whose parameters better match the provided arguments.
  - Properly handles variadic parameters and determines if they make a method more applicable.

- **Context Awareness**:
  - Accounts for special Groovy constructs like extension methods (`GrGdkMethod`) and callable types.
  - Ensures that Groovy-specific semantics are respected, such as prioritizing `Runnable` over `Callable` when necessary.

- **Ambiguity Resolution**:
  - Uses additional context (e.g., return types, receiver types, or resolution context) to decide between methods that might otherwise appear equally applicable.

## How it Works

The `methodComparator` extension point integrates into Groovy's method resolution logic in IntelliJ IDEA. It affects:
- **Code Completion**: By determining the priority and order of suggested methods.
- **Type Inference**: By helping the IDE select the most appropriate method for a given call.
- **Error Highlighting**: By ensuring method ambiguities or mismatches are flagged correctly.

## Implementation Guidelines

To implement a custom `methodComparator`:
1. Extend the `org.jetbrains.plugins.groovy.lang.resolve.GrMethodComparator` class.
2. Provide custom logic in the `dominated` method to compare methods based on your criteria.
3. Register your implementation in the plugin's `plugin.xml` file.

Example:

```xml
<methodComparator implementation="com.example.MyCustomMethodComparator" order="before default"/>
```

This extension point allows for fine-grained control over how IntelliJ resolves and ranks methods in Groovy code, ensuring better alignment with specific use cases or custom Groovy dialects.

