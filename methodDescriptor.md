# Method Descriptor Extension Point

The `methodDescriptor` extension point allows plugin developers to register method descriptors for Groovy classes. These descriptors provide additional metadata and functionality for specific methods, enabling enhanced IDE support, such as code insight, refactoring, or navigation.

---

## Extension Point Declaration

```xml
<extensionPoint name="methodDescriptor"
                dynamic="true"
                beanClass="org.jetbrains.plugins.groovy.extensions.GroovyMethodDescriptorExtension"/>
```

---

## Purpose and Capabilities

### **1. Enhancing IDE Features for Specific Methods**
This extension point enables plugins to define method descriptors that:

- Provide additional runtime or static metadata for Groovy methods.
- Enhance IDE support for specific methods with custom behavior or annotations.

### **2. Fine-Grained Method Targeting**
Each descriptor can target specific classes and methods using attributes like:

- `className`: Specifies the class the method belongs to.
- `lightMethodKey`: A unique identifier for lightweight methods.

---

## Core Class: `GroovyMethodDescriptorExtension`

### Overview
The `GroovyMethodDescriptorExtension` class extends `GroovyMethodDescriptor` and implements the `PluginAware` interface to associate descriptors with plugins.

### Key Attributes

- **`className`**: Defines the fully qualified name of the class containing the method.
- **`lightMethodKey`**: A unique key to identify lightweight or synthetic methods.

### Plugin Awareness
By implementing `PluginAware`, the extension point can access plugin-specific resources and class loaders:

```java
@Override
public final void setPluginDescriptor(@NotNull PluginDescriptor pluginDescriptor) {
    myPluginDescriptor = pluginDescriptor;
}

public @NotNull ClassLoader getLoaderForClass() {
    return myPluginDescriptor != null ?
           myPluginDescriptor.getClassLoader() :
           getClass().getClassLoader();
}
```

---

## Example Implementation

### Descriptor for a Custom Method
This example registers a method descriptor for a Groovy class `MyCustomClass`:

```xml
<extension
    point="org.intellij.groovy.methodDescriptor">
    <methodDescriptor
        class="com.example.MyCustomClass"
        lightMethodKey="myMethodKey"/>
</extension>
```

### Implementation of the Descriptor

```java
public final class MyCustomMethodDescriptor extends GroovyMethodDescriptorExtension {

  @Override
  public void describe() {
      // Define the behavior or metadata for the target method
  }
}
```

---

## Use Cases and Benefits

### **1. DSL-Specific Method Support**
Plugins for Groovy-based DSLs can define method descriptors to:

- Provide type hints and parameter information.
- Customize behavior for method calls, improving IDE validation and refactoring.

### **2. Enhancing Code Completion**
Descriptors can augment method metadata, enabling more accurate and context-aware code completion.

### **3. Framework and Library Integration**
For frameworks like Gradle or Grails, method descriptors can add custom annotations or behavior for framework-specific methods.

---

## Conclusion
The `methodDescriptor` extension point is a powerful tool for enhancing Groovy method handling in the IDE. By providing targeted metadata and functionality, it enables plugins to offer a more refined and productive development experience for Groovy users.

