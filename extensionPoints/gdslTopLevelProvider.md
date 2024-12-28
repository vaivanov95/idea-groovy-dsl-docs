# GDSL Top-Level Provider Extension Point

The `gdslTopLevelProvider` extension point empowers developers to extend Groovy's Domain-Specific Language (DSL) capabilities in IntelliJ IDEA by defining methods and properties accessible at the top level of Groovy DSL scripts. These extensions enable custom behavior, enriching Groovy DSL support in the IDE and enhancing code insights such as completion, navigation, and error checking.

---

## Extension Point Declaration

```xml
<extensionPoint name="gdslTopLevelProvider"
                dynamic="false"
                interface="org.jetbrains.plugins.groovy.dsl.dsltop.GdslMembersProvider"/>
```

### Purpose
This extension point:
- Provides methods and properties that can be invoked at the top level of Groovy DSL closures.
- Enhances the expressiveness and utility of Groovy DSLs by injecting domain-specific constructs into the Groovy scripting environment.
- Improves IDE support for custom DSLs by leveraging IntelliJ’s PSI model and dynamic extensions.

---

## Key Interface: `GdslMembersProvider`

The `GdslMembersProvider` interface is implemented to supply top-level methods and properties for use in Groovy DSLs.

```java
package org.jetbrains.plugins.groovy.dsl.dsltop;

import com.intellij.openapi.extensions.ExtensionPointName;

public interface GdslMembersProvider {
  ExtensionPointName<GdslMembersProvider> EP_NAME =
    ExtensionPointName.create("org.intellij.groovy.gdslTopLevelProvider");
}
```

### Overview
Implementations of this interface:
- Provide utility methods and properties accessible within Groovy DSL closures.
- Use the `GdslMembersHolderConsumer` parameter to register methods and properties dynamically during the DSL script analysis.

---

## Example Implementations

### **1. Default GDSL Members**

The `GroovyDslDefaultMembers` class supplies a set of common methods for handling PSI elements, such as finding classes or adding members.

#### Key Methods

1. **`findClass(String fqn, GdslMembersHolderConsumer consumer)`**
   - Locates a class by its fully qualified name.
   - Example use: `findClass("java.lang.String", consumer)`

2. **`delegatesTo(PsiElement elem, GdslMembersHolderConsumer consumer)`**
   - Delegates methods and properties from a class or expression to the current context.
   - Useful for extending DSL behavior with additional methods from a given class.

3. **`add(PsiMember member, GdslMembersHolderConsumer consumer)`**
   - Registers a new member (e.g., a method or field) to the current DSL context.

4. **`enclosingCall(String name, GdslMembersHolderConsumer consumer)`**
   - Retrieves the enclosing method call by name, useful for contextual DSL processing.

#### Implementation

```java
public final class GroovyDslDefaultMembers implements GdslMembersProvider {

  public @Nullable PsiClass findClass(String fqn, GdslMembersHolderConsumer consumer) {
    JavaPsiFacade facade = JavaPsiFacade.getInstance(consumer.getProject());
    return facade.findClass(fqn, GlobalSearchScope.allScope(consumer.getProject()));
  }

  public void delegatesTo(@Nullable PsiElement elem, GdslMembersHolderConsumer consumer) {
    if (elem instanceof PsiClass clazz) {
      NonCodeMembersHolder holder = new NonCodeMembersHolder();

      for (PsiMethod method : clazz.getAllMethods()) {
        if (!method.isConstructor()) holder.addDeclaration(method);
      }
      for (PsiField field : clazz.getAllFields()) {
        holder.addDeclaration(field);
      }

      consumer.addMemberHolder(holder);
    }
  }
}
```

---

### **2. GDK Method DSL Provider**

This implementation integrates Groovy Default Methods (GDK) into the DSL context, allowing dynamic method injection based on a specified category class.

#### Key Methods

1. **`category(String className, GdslMembersHolderConsumer consumer)`**
   - Registers methods from a specified category class as available in the current DSL context.

2. **`processCategoryMethods(String className, GdslMembersHolderConsumer consumer, boolean isStatic)`**
   - Processes and registers methods from the category class dynamically, filtering static and instance methods as required.

#### Implementation

```java
public final class GdkMethodDslProvider implements GdslMembersProvider {

  public void category(String className, GdslMembersHolderConsumer consumer) {
    processCategoryMethods(className, consumer, false);
  }

  private static void processCategoryMethods(String className, GdslMembersHolderConsumer consumer, boolean isStatic) {
    GlobalSearchScope scope = consumer.getResolveScope();
    PsiClass categoryClass = JavaPsiFacade.getInstance(consumer.getProject()).findClass(className, scope);
    if (categoryClass == null) return;

    NotNullLazyValue<GdkMethodHolder> methodsMap = NotNullLazyValue.volatileLazy(() -> {
      return GdkMethodHolder.getHolderForClass(categoryClass, isStatic);
    });

    consumer.addMemberHolder(new CustomMembersHolder() {
      @Override
      public boolean processMembers(GroovyClassDescriptor descriptor, PsiScopeProcessor processor, ResolveState state) {
        return methodsMap.getValue().processMethods(processor, state, descriptor.getPsiType(), descriptor.getProject());
      }
    });
  }
}
```

---

## Use Cases

### **1. Enriching DSLs with Domain-Specific Behavior**
Developers can use `gdslTopLevelProvider` to inject domain-specific methods and properties directly into Groovy DSLs. For example:
- Adding utility methods for configuring build scripts in Gradle.
- Defining top-level methods for Groovy-based testing frameworks.

### **2. Context-Aware Method Injection**
Implementations can dynamically determine the methods to inject based on the DSL script’s context, enabling:
- Customized behavior for different project types.
- Dynamic delegation of methods based on runtime conditions.

### **3. Enhancing IDE Support for Custom DSLs**
The injected methods and properties are integrated with IntelliJ IDEA’s code insight features, providing:
- Code completion for injected methods and properties.
- Navigation and resolution for dynamically added members.
- Error highlighting for invalid DSL constructs.

---

## Conclusion

The `gdslTopLevelProvider` extension point enables IntelliJ IDEA plugins to enhance Groovy DSLs by injecting top-level methods and properties dynamically. By leveraging this extension point, developers can create rich, domain-specific tools and workflows that seamlessly integrate into the IntelliJ ecosystem, offering enhanced productivity and better user experiences for Groovy developers.

