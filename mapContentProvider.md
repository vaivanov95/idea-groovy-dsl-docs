# Map Content Provider Extension Point

The `mapContentProvider` extension point in the IntelliJ Platform allows plugins to enhance IDE features for Groovy maps by providing domain-specific insights. It is particularly useful in DSLs (Domain-Specific Languages) where map-like structures have predefined or inferred keys and types.

## Definition

```xml
<extensionPoint name="mapContentProvider" dynamic="true" interface="org.jetbrains.plugins.groovy.extensions.GroovyMapContentProvider"/>
```

## Overview

The `mapContentProvider` is designed to:

1. **Enhance Map Key Support**: Provide domain-specific keys for Groovy maps.
2. **Infer Key Types**: Determine the type associated with specific map keys.
3. **Integrate with IDE Features**: Improve code completion, navigation, and inspections for Groovy maps.

This extension is particularly useful for DSLs, where maps are often used to define structured data with well-known keys.

## Key Interface

### `GroovyMapContentProvider`

The `GroovyMapContentProvider` abstract class provides two main methods:

#### 1. `getKeyVariants`

Computes all possible keys for a given map.

```java
protected Collection<String> getKeyVariants(@NotNull GrExpression qualifier, @Nullable PsiElement resolve) {
    throw new UnsupportedOperationException();
}
```

- **Parameters**:
  - `qualifier`: The map's qualifier expression.
  - `resolve`: The element to which the map resolves.
- **Returns**: A collection of strings representing possible keys.

#### 2. `getValueType`

Determines the type associated with a specific map key.

```java
public @Nullable PsiType getValueType(@NotNull GrExpression qualifier, @Nullable PsiElement resolve, @NotNull String key) {
    return null;
}
```

- **Parameters**:
  - `qualifier`: The map's qualifier expression.
  - `resolve`: The element to which the map resolves.
  - `key`: The key whose type is to be determined.
- **Returns**: The `PsiType` associated with the key, or `null` if unknown.

## Implementation Example

Here is an example implementation for enhancing a Groovy DSL that uses `ConfigSlurper`:

### `ConfigSlurperMapContentProvider`

This implementation enhances support for maps in the `ConfigSlurper` DSL, providing domain-specific keys and types.

#### Code Example

```java
package org.jetbrains.plugins.groovy.configSlurper;

import com.intellij.openapi.util.Pair;
import com.intellij.openapi.util.Ref;
import com.intellij.psi.JavaPsiFacade;
import com.intellij.psi.PsiElement;
import com.intellij.psi.PsiType;
import com.intellij.psi.util.InheritanceUtil;
import org.jetbrains.annotations.NotNull;
import org.jetbrains.annotations.Nullable;
import org.jetbrains.plugins.groovy.extensions.GroovyMapContentProvider;
import org.jetbrains.plugins.groovy.lang.psi.api.statements.expressions.GrExpression;
import org.jetbrains.plugins.groovy.lang.psi.api.statements.expressions.GrMethodCall;
import org.jetbrains.plugins.groovy.lang.psi.api.statements.expressions.GrReferenceExpression;
import org.jetbrains.plugins.groovy.lang.resolve.api.GroovyMapProperty;

import java.util.*;

public final class ConfigSlurperMapContentProvider extends GroovyMapContentProvider {

  private static @Nullable Pair<ConfigSlurperSupport.PropertiesProvider, List<String>> getInfo(@NotNull GrExpression qualifier,
                                                                                               @Nullable PsiElement resolve) {
    if (!InheritanceUtil.isInheritor(qualifier.getType(), GroovyCommonClassNames.GROOVY_UTIL_CONFIG_OBJECT)) {
      return null;
    }

    GrExpression resolvedQualifier = qualifier;
    PsiElement resolveResult = resolve;
    List<String> path = new ArrayList<>();

    while (resolveResult instanceof GroovyMapProperty) {
      if (!(resolvedQualifier instanceof GrReferenceExpression expr)) return null;

      path.add(expr.getReferenceName());

      resolvedQualifier = expr.getQualifierExpression();
      if (resolvedQualifier instanceof GrReferenceExpression) {
        resolveResult = ((GrReferenceExpression)resolvedQualifier).resolve();
      }
      else if (resolvedQualifier instanceof GrMethodCall) {
        resolveResult = ((GrMethodCall)resolvedQualifier).resolveMethod();
      }
      else {
        return null;
      }
    }
    if (resolveResult == null) {
      return null;
    }

    Collections.reverse(path);

    ConfigSlurperSupport.PropertiesProvider propertiesProvider = null;

    for (ConfigSlurperSupport slurperSupport : ConfigSlurperSupport.EP_NAME.getExtensions()) {
      propertiesProvider = slurperSupport.getConfigSlurperInfo(resolveResult);
      if (propertiesProvider != null) break;
    }

    if (propertiesProvider == null) return null;

    return Pair.create(propertiesProvider, path);
  }

  @Override
  protected Collection<String> getKeyVariants(@NotNull GrExpression qualifier, @Nullable PsiElement resolve) {
    Pair<ConfigSlurperSupport.PropertiesProvider, List<String>> info = getInfo(qualifier, resolve);
    if (info == null) return Collections.emptyList();

    final Set<String> res = new HashSet<>();

    info.first.collectVariants(info.second, (variant, isFinal) -> res.add(variant));

    return res;
  }

  @Override
  public PsiType getValueType(@NotNull GrExpression qualifier, @Nullable PsiElement resolve, final @NotNull String key) {
    Pair<ConfigSlurperSupport.PropertiesProvider, List<String>> info = getInfo(qualifier, resolve);
    if (info == null) return null;

    final Ref<Boolean> res = new Ref<>();

    info.first.collectVariants(info.second, (variant, isFinal) -> {
      if (variant.equals(key)) {
        res.set(isFinal);
      }
      else if (variant.startsWith(key) && variant.length() > key.length() && variant.charAt(key.length()) == '.') {
        res.set(false);
      }
    });

    if (res.get() != null && !res.get()) {
      return JavaPsiFacade.getElementFactory(qualifier.getProject()).createTypeByFQClassName(GroovyCommonClassNames.GROOVY_UTIL_CONFIG_OBJECT, qualifier.getResolveScope());
    }

    return null;
  }
}
```

## Use Cases

### DSLs with Predefined Map Keys
DSLs often use maps with predefined keys to structure data. For example, `ConfigSlurper` maps define configuration hierarchies. This extension allows:

- Suggesting valid keys during code completion.
- Providing type information for keys.
- Improving navigation and highlighting for map elements.

### Dynamic Map Structures
For dynamically structured maps, this extension can infer possible keys based on runtime or context-specific rules.

## Registration

To register a `GroovyMapContentProvider` implementation, add the following to your `plugin.xml`:

```xml
<mapContentProvider implementation="com.example.MyMapContentProvider"/>
```

## Conclusion

The `mapContentProvider` extension point significantly enhances the IDE experience for DSLs and domain-specific map usages, providing contextual insights for map keys and their types.

