# Groovy Class Descriptor Extension Point

The `classDescriptor` extension point enables the declaration of additional metadata about Groovy classes, allowing plugin developers to define specific behaviors, methods, or descriptors for Groovy classes. This can be useful for enhancing code insight, navigation, and completion in the IDE for Groovy code.

---

## Extension Point Declaration

```xml
<!--suppress PluginXmlValidity -->
<extensionPoint name="classDescriptor" dynamic="true"
                beanClass="org.jetbrains.plugins.groovy.extensions.GroovyClassDescriptor"/>
```

---

## Purpose and Capabilities

### **1. Metadata for Groovy Classes**
The `classDescriptor` extension point allows plugin developers to specify:
- The name of a Groovy class.
- Methods or properties associated with that class.

### **2. Enhanced IDE Features**
By providing detailed metadata about a class, plugins can enhance the following IDE features:
- **Code Completion**: Suggest appropriate methods or properties.
- **Navigation**: Enable direct navigation to method definitions or references.
- **Code Analysis**: Provide additional validation or inspections for Groovy classes.

---

## API Details

### **Class: `GroovyClassDescriptor`**
The `GroovyClassDescriptor` class defines the metadata structure for a Groovy class:

```java
package org.jetbrains.plugins.groovy.extensions;

import com.intellij.openapi.extensions.ExtensionPointName;
import com.intellij.util.xmlb.annotations.Attribute;
import com.intellij.util.xmlb.annotations.Property;
import com.intellij.util.xmlb.annotations.XCollection;

public class GroovyClassDescriptor {
  public static final ExtensionPointName<GroovyClassDescriptor> EP_NAME = new ExtensionPointName<>("org.intellij.groovy.classDescriptor");

  @Attribute("class")
  public String className;

  @Property(surroundWithTag = false)
  @XCollection
  public GroovyMethodDescriptorTag[] methods;
}
```

### Key Components

#### **Attributes**
- `className`: The fully qualified name of the Groovy class being described. This field is required and determines which class the metadata applies to.

#### **Properties**
- `methods`: An array of `GroovyMethodDescriptorTag` objects, which describe the methods or properties associated with the class.

---

## Example Implementation

### **Declaring a Class Descriptor**
The following XML snippet demonstrates how to define a `classDescriptor` for a custom Groovy class:

```xml
<classDescriptor class="com.example.MyGroovyClass">
  <methods>
    <method name="myMethod" returnType="java.lang.String">
      <parameter name="param1" type="int"/>
    </method>
  </methods>
</classDescriptor>
```

### **Enhancing a Groovy Class**
Consider the Groovy class `com.example.MyGroovyClass` with a method `myMethod(int param1)`. The above descriptor adds metadata about this method, allowing the IDE to suggest it in code completion and validate its usage.

---

## Use Cases and Benefits

### **1. Framework Support**
Plugins for frameworks like Grails or Spock can use this extension point to describe framework-specific classes, making it easier for developers to use them.

### **2. Custom DSLs**
For custom Groovy DSLs, the `classDescriptor` extension point provides a way to define the expected methods and properties, improving developer productivity by integrating them into the IDE.

### **3. Enhanced Code Assistance**
By describing methods and their signatures, this extension point ensures accurate code completion, navigation, and analysis for Groovy classes.

---

## Conclusion
The `classDescriptor` extension point is a powerful tool for enhancing the IDE’s understanding of Groovy classes. It allows plugin developers to define additional metadata, enabling advanced code insight and improving the developer experience for Groovy projects.

