# Overload Resolver Extension Point

The `overloadResolver` extension point in the IntelliJ Platform provides a mechanism for defining custom logic to compare and prioritize Groovy method overloads during method resolution.

## Definition

```xml
<extensionPoint name="overloadResolver" dynamic="true" interface="org.jetbrains.plugins.groovy.lang.resolve.api.GroovyOverloadResolver"/>
```

## Example Instances

Below are example implementations of the `overloadResolver` extension point:

```xml
<overloadResolver implementation="org.jetbrains.plugins.groovy.lang.resolve.DGMGetAtOverloadResolver" order="last"/>
<overloadResolver implementation="org.jetbrains.plugins.groovy.lang.resolve.impl.DistanceOverloadResolver" order="last"/>
```

## Key Method

Implementations of this extension point must provide the following method:

```java
int compare(@NotNull GroovyMethodResult left, @NotNull GroovyMethodResult right);
```

### Return Value:
- **Negative Value**: Indicates that `left` is more preferable.
- **Zero**: Indicates that neither `left` nor `right` is preferable.
- **Positive Value**: Indicates that `right` is more preferable.

## Purpose of the Extension

The `overloadResolver` extension point is used to refine or override the default logic for resolving and prioritizing Groovy method overloads. This ensures that the IDE’s behavior aligns with specific language rules or provides additional flexibility for custom Groovy dialects.

It plays a key role in:
- **Resolving ambiguities** when multiple method overloads match a call.
- **Ensuring consistency** with Groovy compiler rules or custom language extensions.

## Built-in Implementations

### `org.jetbrains.plugins.groovy.lang.resolve.DGMGetAtOverloadResolver`
This implementation lowers the priority of specific `DefaultGroovyMethods` overloads:
- `DefaultGroovyMethods.getAt(Object, String)`
- `DefaultGroovyMethods.putAt(Object, String, Object)`

#### Purpose:
Groovy tends to prioritize `DGM#getAt(Object, String)` over `DGM#getAt(Map, Object)` due to argument distance, despite the latter being more specific and better aligned with generic types. This implementation:
- Artificially lowers the priority of `Object`-based overloads in favor of `Map`-based ones to ensure better generic handling.
- Applies similar logic to `putAt` overloads.

#### Method Highlights:
- **Extension Method Handling**: Considers whether the method is an extension method and resolves it accordingly.
- **Parameter Matching**: Identifies and deprioritizes methods that use overly generic parameter types (`Object`) compared to more specific overloads.

### `org.jetbrains.plugins.groovy.lang.resolve.impl.DistanceOverloadResolver`
This implementation compares overloads based on the **argument-to-parameter mapping distance**.

#### Purpose:
When multiple methods are applicable, this resolver:
- Prefers methods with closer argument-to-parameter matches.
- Ensures that the method with the most specific mapping is selected.

#### Method Highlights:
- **Mapping Comparison**: Evaluates the quality of the argument mapping for both `left` and `right` methods.
- **Null Mapping Handling**: Handles cases where one or both methods lack a valid argument mapping, ensuring a consistent resolution outcome.

## How it Works

The `overloadResolver` extension point integrates into the Groovy method resolution process in IntelliJ IDEA. It affects:
- **Code Completion**: By influencing the priority of overloads suggested in autocompletion.
- **Type Inference**: By helping select the most appropriate method overload based on context.
- **Error Highlighting**: By ensuring that ambiguous or incorrect overload selections are flagged appropriately.

## Implementation Guidelines

To implement a custom `overloadResolver`:
1. Extend the `org.jetbrains.plugins.groovy.lang.resolve.api.GroovyOverloadResolver` interface.
2. Provide custom logic in the `compare` method to evaluate and prioritize method overloads.
3. Register your implementation in the plugin's `plugin.xml` file.

Example:

```xml
<overloadResolver implementation="com.example.MyCustomOverloadResolver" order="before default"/>
```

This extension point enables fine-grained control over how IntelliJ IDEA resolves and prioritizes Groovy method overloads, ensuring alignment with specific rules or use cases.

