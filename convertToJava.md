# Custom Method Invocator Extension Point

The `convertToJava.customMethodInvocator` extension point allows developers to customize how Groovy methods are translated into Java code during the "Convert to Java" refactoring process in IntelliJ IDEA. This extension is particularly useful for handling domain-specific methods or enhancing the conversion logic for certain method patterns.

---

## Extension Point Declaration

```xml
<extensionPoint name="convertToJava.customMethodInvocator" dynamic="true"
                interface="org.jetbrains.plugins.groovy.refactoring.convertToJava.invocators.CustomMethodInvocator"/>
```

---

## Key Interface: `CustomMethodInvocator`

The `CustomMethodInvocator` interface defines the contract for implementing custom logic to invoke Groovy methods during the conversion to Java. It allows extensions to intercept and process method calls with a specific translation logic.

### Interface Definition

```java
public abstract class CustomMethodInvocator {
  private static final ExtensionPointName<CustomMethodInvocator> EP_NAME =
    ExtensionPointName.create("org.intellij.groovy.convertToJava.customMethodInvocator");

  protected abstract boolean invoke(
    @NotNull ExpressionGenerator generator,
    @NotNull PsiMethod method,
    @Nullable GrExpression caller,
    GrExpression @NotNull [] exprs,
    GrNamedArgument @NotNull [] namedArgs,
    GrClosableBlock @NotNull [] closures,
    @NotNull PsiSubstitutor substitutor,
    @NotNull GroovyPsiElement context
  );

  public static boolean invokeMethodOn(
    @NotNull ExpressionGenerator generator,
    @NotNull GrGdkMethod method,
    @Nullable GrExpression caller,
    GrExpression @NotNull [] exprs,
    GrNamedArgument @NotNull [] namedArgs,
    GrClosableBlock @NotNull [] closures,
    @NotNull PsiSubstitutor substitutor,
    @NotNull GroovyPsiElement context
  ) {
    final PsiMethod staticMethod = method.getStaticMethod();
    for (CustomMethodInvocator invocator : EP_NAME.getExtensions()) {
      if (invocator.invoke(generator, staticMethod, caller, exprs, namedArgs, closures, substitutor, context)) {
        return true;
      }
    }
    return false;
  }
}
```

---

## Example Implementation

### Map Getter and Setter Invocator

This implementation translates `getAt` and `putAt` methods in Groovy into equivalent `get` and `put` calls for Java collections such as `Map` and `List`.

#### Code Example

```java
public final class MapGetterSetterInvocator extends CustomMethodInvocator {
  @Override
  protected boolean invoke(@NotNull ExpressionGenerator generator,
                           @NotNull PsiMethod method,
                           @Nullable GrExpression caller,
                           GrExpression @NotNull [] exprs,
                           GrNamedArgument @NotNull [] namedArgs,
                           GrClosableBlock @NotNull [] closures,
                           @NotNull PsiSubstitutor substitutor,
                           @NotNull GroovyPsiElement context) {
    if (!method.getName().equals("putAt") && !method.getName().equals("getAt")) return false;

    final PsiClass clazz = method.getContainingClass();
    if (clazz == null) return false;

    final String qname = clazz.getQualifiedName();
    if (!GroovyCommonClassNames.DEFAULT_GROOVY_METHODS.equals(qname)) return false;

    if (caller == null) return false;
    final PsiType type = caller.getType();

    if (method.getName().equals("getAt")) {
      if (InheritanceUtil.isInheritor(type, CommonClassNames.JAVA_UTIL_MAP)) {
        GenerationUtil.invokeMethodByName(caller, "get", exprs, namedArgs, closures, generator, context);
        return true;
      } else if (InheritanceUtil.isInheritor(type, CommonClassNames.JAVA_UTIL_LIST)) {
        GenerationUtil.invokeMethodByName(caller, "get", exprs, namedArgs, closures, generator, context);
        return true;
      }
    } else if (method.getName().equals("putAt")) {
      if (InheritanceUtil.isInheritor(type, CommonClassNames.JAVA_UTIL_MAP)) {
        GenerationUtil.invokeMethodByName(caller, "put", exprs, namedArgs, closures, generator, context);
        return true;
      } else if (InheritanceUtil.isInheritor(type, CommonClassNames.JAVA_UTIL_LIST)) {
        GenerationUtil.invokeMethodByName(caller, "set", exprs, namedArgs, closures, generator, context);
        return true;
      }
    }

    return false;
  }
}
```

#### Behavior
- **`getAt`:** Translates to `get` for both `Map` and `List`.
- **`putAt`:** Translates to `put` for `Map` and `set` for `List`.

#### DSL Use Cases
- **Dynamic Properties:** Handles scenarios where Groovy collections are accessed or modified dynamically.
- **Groovy-to-Java Translation:** Ensures accurate translation of Groovy idiomatic syntax into equivalent Java constructs.

---

## Capabilities and Use Cases

### **Capabilities**
1. **Custom Translation Logic:** Implementors can define how specific Groovy methods should be translated to Java.
2. **Domain-Specific Support:** Enables tailored handling of methods for frameworks or custom DSLs.
3. **Static and Dynamic Method Handling:** Supports static methods as well as dynamically resolved Groovy methods.

### **Use Cases**
- **Framework Adaptation:** Simplifies integration with Java frameworks by translating Groovy-specific constructs.
- **Enhanced Refactoring:** Improves the accuracy and readability of the "Convert to Java" refactoring output.
- **DSL Translation:** Converts domain-specific language constructs written in Groovy into equivalent Java code.

---

## Conclusion

The `convertToJava.customMethodInvocator` extension point is a powerful tool for bridging the gap between Groovy and Java. By providing a way to customize method invocation translation, it empowers developers to handle complex or domain-specific scenarios seamlessly. Whether for collections, DSLs, or frameworks, this extension enhances the Groovy-to-Java conversion experience, ensuring code accuracy and maintainability.

