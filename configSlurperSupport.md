# Config Slurper Support Extension Point

The `configSlurperSupport` extension point enhances support for Groovy's `ConfigSlurper` feature within the IDE. This extension point allows plugin developers to provide domain-specific integrations for interpreting and handling Groovy configuration scripts. By extending this functionality, plugins can improve code insight, completion, and validation for configuration-based DSLs or frameworks.

---

## Extension Point Declaration

```xml
<extensionPoint name="configSlurperSupport"
                dynamic="true"
                interface="org.jetbrains.plugins.groovy.configSlurper.ConfigSlurperSupport"/>
```

---

## Purpose and Capabilities

### **1. Supporting Groovy's `ConfigSlurper`**
The Groovy `ConfigSlurper` is used for building configuration objects based on Groovy scripts. It is widely used in frameworks like Grails and other domain-specific Groovy DSLs. The `configSlurperSupport` extension point allows plugin developers to:

- Enhance code completion for dynamic properties within configuration scripts.
- Provide validation and error-checking for configuration keys.
- Improve navigation and reference resolution for configuration constructs.

### **2. Domain-Specific Enhancements**
Frameworks and libraries often extend the use of `ConfigSlurper` by introducing their own set of configuration properties. This extension point allows plugins to:

- Define custom property providers for specific configurations.
- Enable context-aware code insight tailored to the framework’s configuration model.

### **3. Enabling Property Completion**
Using the `PropertiesProvider` interface, plugins can provide lists of available configuration keys based on the script's context. This ensures developers get meaningful suggestions when writing configuration scripts.

---

## Key Interface

### `ConfigSlurperSupport`
This abstract class serves as the entry point for providing `ConfigSlurper` support. Implementing classes define how to extract property information from configuration scripts.

```java
public abstract class ConfigSlurperSupport {

  public static final ExtensionPointName<ConfigSlurperSupport> EP_NAME =
    ExtensionPointName.create("org.intellij.groovy.configSlurperSupport");

  public abstract @Nullable PropertiesProvider getProvider(@NotNull GroovyFile file);

  public @Nullable PropertiesProvider getConfigSlurperInfo(@NotNull PsiElement qualifierResolve) {
    return null;
  }

  public interface PropertiesProvider {
    void collectVariants(@NotNull List<String> prefix, @NotNull PairConsumer<String, Boolean> consumer);
  }
}
```

#### Key Methods:
- **`getProvider(GroovyFile file)`**:
  Returns a `PropertiesProvider` for the given Groovy file. This provider defines how configuration keys and values are interpreted in the context of the file.

- **`getConfigSlurperInfo(PsiElement qualifierResolve)`**:
  Optionally provides a `PropertiesProvider` based on the resolved element in a configuration expression.

- **`PropertiesProvider`**:
  An interface for supplying configuration property information. Its key method, `collectVariants`, gathers configuration keys based on a provided prefix.

---

## Example Implementation: `ConfigSlurperSupport`

### Overview
A common use case for `ConfigSlurperSupport` is enhancing IDE features for configuration scripts. The following example demonstrates how the support integrates with a hypothetical DSL to provide property suggestions and type-checking.

```java
public final class MyFrameworkConfigSlurperSupport extends ConfigSlurperSupport {

  @Override
  public @Nullable PropertiesProvider getProvider(@NotNull GroovyFile file) {
    if (isMyFrameworkConfigFile(file)) {
      return new MyFrameworkPropertiesProvider();
    }
    return null;
  }

  private boolean isMyFrameworkConfigFile(GroovyFile file) {
    // Logic to determine if the file belongs to the framework
    return file.getName().endsWith("MyFrameworkConfig.groovy");
  }

  private static class MyFrameworkPropertiesProvider implements PropertiesProvider {

    @Override
    public void collectVariants(@NotNull List<String> prefix, @NotNull PairConsumer<String, Boolean> consumer) {
      // Example: Providing keys like "database.url" and "database.user"
      if (prefix.isEmpty() || prefix.get(0).equals("database")) {
        consumer.consume("url", true);
        consumer.consume("user", false);
      }
    }
  }
}
```

### Capabilities Demonstrated
1. **File Recognition**:
   - The implementation identifies configuration files specific to the framework by checking the file name.

2. **Key Suggestions**:
   - Provides property keys such as `database.url` and `database.user` for completion and validation.

3. **Type Checking**:
   - The boolean parameter in `consumer.consume` can indicate whether the property is required or optional.

---

## Use Cases and Benefits

### **1. Framework-Specific Configuration Enhancements**
Plugins can enhance IDE support for frameworks by:
- Providing code completion for configuration keys.
- Validating required and optional keys.
- Offering navigation to the source of configuration definitions.

### **2. General DSL Support**
The extension point is not limited to frameworks like Grails. Any DSL that uses Groovy for configuration can benefit from tailored support, making it easier for developers to write and debug configurations.

### **3. Improved Developer Experience**
By surfacing meaningful insights into configuration files, this extension point reduces the cognitive load on developers, ensuring they can focus on building applications rather than resolving cryptic configuration issues.

---

## Conclusion

The `configSlurperSupport` extension point is a powerful tool for enhancing IDE features for Groovy-based configuration scripts. By leveraging this extension point, plugin developers can provide tailored, domain-specific enhancements that improve code insight, completion, and validation, delivering a smoother development experience.

