# IntelliJ IDEA Groovy Plugin Extension Points Overview

This repository provides detailed documentation and articles on various extension points used in IntelliJ IDEA's Groovy plugin. Each file focuses on a specific extension point or related functionality, explaining its purpose, use cases, and example implementations.

---

## Files Overview

### AST Transformations
- [AST Transformation Support](./extensionPoints/astTransformationSupport.md)  
  Extends support for Groovy AST transformations, enabling custom processing during compilation.

- [Inline AST Transformation Support](./extensionPoints/inlineASTTransformationSupport.md)  
  Provides debugging and inline transformation capabilities for Groovy scripts.

---

### Source files detection and classification
- [Groovy Source Folder Detector](./extensionPoints/groovySourceFolderDetector.md)  
  Detects and marks directories as Groovy source folders in a project.

- [Script Type Detector](./extensionPoints/scriptTypeDetector.md)  
  Identifies and categorizes Groovy scripts based on their content or context.

---

### Code Completion and Navigation
- [Closure Completer](./extensionPoints/closureCompleter.md)  
  Enables context-aware completion and parameter templates for Groovy closures.

- [GDSL Top Level Provider](./extensionPoints/gdslTopLevelProvider.md)  
  Adds methods and properties to the top level of Groovy DSLs.

- [Named Argument Provider](./extensionPoints/namedArgumentProvider.md)  
  Adds support for named arguments in Groovy method calls.

---

### Syntax
- [Signature Hint Processor](./extensionPoints/signatureHintProcessor.md)  
  Infers method signatures using annotations or other hints.

- [Closure Missing Method Contributor](./extensionPoints/closureMissingMethodContributor.md)  
  Handles and defines custom behavior for missing methods in Groovy closures.

### Types

- [Type Calculator](./extensionPoints/typeCalculator.md)  
  Dynamically calculates types for Groovy expressions.

- [Type Converter](./extensionPoints/typeConverter.md)  
  Converts between types to support type checking and resolution.

- [Reference Type Enhancer](./extensionPoints/referenceTypeEnhancer.md)  
  Enhances type resolution for Groovy references.

- [Variable Enhancer](./extensionPoints/variableEnhancer.md)  
  Adds custom enhancements to variables in Groovy code.

- [Expected Types Contributor](./extensionPoints/expectedTypesContributor.md)  
  Contributes expected types to improve code completion and suggestions.

---

### Refactoring
- [Rename Helper](./extensionPoints/renameHelper.md)  
  Facilitates renaming refactoring operations with custom rules or behaviors.

- [Convert to Java](./extensionPoints/convertToJava.md)  
  Customizes the conversion of Groovy constructs into Java code.

---

### Inspections

- [Method May Be Static Inspection Filter](./extensionPoints/methodMayBeStaticInspectionFilter.md)  
  Filters methods that should not be flagged as static candidates during inspections.

- [Custom Annotation Checker](./extensionPoints/customAnnotationChecker.md)  
  Adds validation for custom annotations in Groovy code.

- [Inspection Disabler](./extensionPoints/inspectionDisabler.md)  
  Disables specific inspections for Groovy code.

---

### Debugging
- [Position Manager Delegate](./extensionPoints/positionManagerDelegate.md)  
  Maps source code positions to support debugging Groovy files.

---

### Gradle and Build Systems
- [Build System Groovy New Project Wizard](./extensionPoints/buildSystemGroovyNewProjectWizard.md)  
  Extends the new project wizard with build system-specific steps like Gradle.

- [Gradle-Specific Extensions](./extensionPoints/groovyFrameworkConfigNotification.md)  
  Provides notifications and checks for Gradle-specific configurations.

---

### Dynamic Features and Enhancements
- [Delegates To Provider](./extensionPoints/delegatesToProvider.md)  
  Defines custom delegate types for closures, enabling dynamic scoping and delegation.

- [PSI Enhancer Category](./extensionPoints/psiEnhancerCategory.md)  
  Dynamically adds properties and methods to PSI elements in IntelliJ IDEA.

- [Map Content Provider](./extensionPoints/mapContentProvider.md)  
  Contributes custom content for Groovy map structures.

---

### Other Utilities
- [Unresolved Highlight Filter](./extensionPoints/unresolvedHighlightFilter.md)  
  Filters unresolved references in Groovy code to reduce false positives.

- [Groovy Inlay Hint Filter](./extensionPoints/groovyInlayHintFilter.md)  
  Controls the visibility of inlay hints for Groovy code.

- [Config Slurper Support](./extensionPoints/configSlurperSupport.md)  
  Enhances Groovy's `ConfigSlurper` for richer configuration capabilities.

- [Overload Resolver](./extensionPoints/overloadResolver.md)  
  Resolves overloaded method calls based on custom logic.

---

### Specialized Features
- [Method Descriptor](./extensionPoints/methodDescriptor.md)  
  Provides additional metadata and descriptors for Groovy methods.

- [Method Comparator](./extensionPoints/methodComparator.MD)  
  Implements custom logic for comparing Groovy methods.

- [Expected Package Name Provider](./extensionPoints/expectedPackageNameProvider.md)  
  Infers the expected package name for Groovy files based on context.

- [Groovy Element Filter](./extensionPoints/groovyElementFilter.md)  
  Filters Groovy elements based on specific criteria or contexts.



---

