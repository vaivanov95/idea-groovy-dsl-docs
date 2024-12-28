# Closure Missing Method Contributor Extension Point

The `closureMissingMethodContributor` extension point in the IntelliJ Platform allows developers to handle dynamic method calls within Groovy closures. This extension point provides a way to dynamically resolve methods that are invoked inside a `GrClosableBlock` (Groovy closure) but are not explicitly defined.

## Definition

```xml
<extensionPoint name="closureMissingMethodContributor" dynamic="true"
                interface="org.jetbrains.plugins.groovy.lang.resolve.ClosureMissingMethodContributor"/>
```

## Overview

Groovy closures often involve dynamic method calls, particularly when working with DSLs (Domain-Specific Languages). For instance, method calls inside closures might resolve dynamically based on the surrounding context or the structure of the closure itself.

The `closureMissingMethodContributor` extension point provides a mechanism to resolve such methods and integrate them into the IntelliJ IDEA Groovy plugin’s code insight features, including:

- Code completion
- Syntax highlighting
- Reference resolution

## Key Interface

### `ClosureMissingMethodContributor`

This abstract class defines the contract for resolving methods inside closures.

#### Key Method

```java
public abstract boolean processMembers(GrClosableBlock closure,
                                       PsiScopeProcessor processor,
                                       GrReferenceExpression refExpr,
                                       ResolveState state);
```

- **Parameters**:
  - `closure`: The Groovy closure in which the dynamic method is invoked.
  - `processor`: The scope processor to handle the resolved elements.
  - `refExpr`: The reference expression representing the method call.
  - `state`: Additional resolve state information.
- **Returns**: A boolean indicating whether to continue processing.

This method is called for each closure in the context chain. It allows contributors to dynamically resolve methods based on the context of the closure.

### Static Utility Method

The utility method `processMethodsFromClosures` iterates through the context chain of a reference expression, invoking all registered contributors.

```java
public static boolean processMethodsFromClosures(GrReferenceExpression ref, PsiScopeProcessor processor) {
  if (!ResolveUtilKt.shouldProcessMethods(processor)) return true;
  for (PsiElement e = ref.getContext(); e != null; e = e.getContext()) {
    if (e instanceof GrClosableBlock) {
      ResolveState state = ResolveState.initial().put(ClassHint.RESOLVE_CONTEXT, e);
      for (ClosureMissingMethodContributor contributor : EP_NAME.getExtensions()) {
        if (!contributor.processMembers((GrClosableBlock)e, processor, ref, state)) {
          return false;
        }
      }
    }
  }
  return true;
}
```

This method ensures that all registered contributors are called to resolve dynamic methods inside closures.

## Example Implementation

### PluginXmlClosureMemberContributor

The `PluginXmlClosureMemberContributor` demonstrates how to resolve methods dynamically invoked within Groovy closures, specifically those related to plugin configurations.

#### Implementation

```java
public final class PluginXmlClosureMemberContributor extends ClosureMissingMethodContributor {
  @Override
  public boolean processMembers(GrClosableBlock closure, PsiScopeProcessor processor, GrReferenceExpression refExpr, ResolveState state) {
    PsiElement parent = closure.getParent();
    if (parent instanceof GrArgumentList) parent = parent.getParent();

    if (!(parent instanceof GrMethodCall)) return true;

    PsiMethod psiMethod = ((GrMethodCall)parent).resolveMethod();
    if (psiMethod == null) return true;

    List<GroovyMethodInfo> infos = GroovyMethodInfo.getInfos(psiMethod);
    if (infos.isEmpty()) return true;

    int index = PsiUtil.getArgumentIndex((GrMethodCall)parent, closure);
    if (index == -1) return true;

    for (GroovyMethodInfo info : infos) {
      GroovyMethodDescriptor.ClosureArgument[] closureArguments = info.getDescriptor().myClosureArguments;
      if (closureArguments != null) {
        for (GroovyMethodDescriptor.ClosureArgument argument : closureArguments) {
          if (argument.index == index && argument.methodContributor != null) {
            ClosureMissingMethodContributor instance = SingletonInstancesCache.getInstance(argument.methodContributor, info.getPluginClassLoader());
            if (!instance.processMembers(closure, processor, refExpr, state)) return false;
          }
        }
      }
    }

    return true;
  }
}
```

#### Explanation

- The contributor checks the context of the closure (e.g., whether it’s part of a method call).
- Resolves the enclosing method and retrieves its metadata.
- Processes metadata to resolve dynamic methods based on argument indices and specific contributors.

## Registration Example

To register a custom `ClosureMissingMethodContributor`, include the following in the plugin’s `plugin.xml` file:

```xml
<closureMissingMethodContributor implementation="com.example.MyClosureMethodContributor"/>
```

## Key Considerations

1. **Dynamic Contexts**: Ensure contributors account for the dynamic nature of closures and their contexts.
2. **Efficient Resolution**: Optimize method resolution to avoid performance overhead, especially in complex projects.
3. **IDE Integration**: Ensure resolved methods integrate seamlessly with code insight features like highlighting, completion, and navigation.

## Conclusion

The `closureMissingMethodContributor` extension point provides a powerful way to handle dynamic method resolution in Groovy closures, enabling enhanced IDE support for DSLs and other dynamic constructs.

