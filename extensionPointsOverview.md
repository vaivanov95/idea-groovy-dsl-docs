# IntelliJ IDEA Groovy Plugin Extension Points Overview

This repository provides detailed documentation and articles on various extension points used in IntelliJ IDEA's Groovy plugin. Each file focuses on a specific extension point or related functionality, explaining its purpose, use cases, and example implementations.

---

## Files Overview

### General Extensions
- [AST Transformation Support](./extensionPoints/astTransformationSupport.md)  
  Discusses support for Groovy AST transformations and how to extend their capabilities.

- [Config Slurper Support](./extensionPoints/configSlurperSupport.md)  
  Details extending Groovy's `ConfigSlurper` for enhanced configuration capabilities.

- [Groovy Source Folder Detector](./extensionPoints/groovySourceFolderDetector.md)  
  Explains how to detect and mark Groovy source folders within a project.

- [Script Type Detector](./extensionPoints/scriptTypeDetector.md)  
  Provides information on identifying and categorizing Groovy script types.

---

### Code Completion and Navigation
- [Closure Completer](./extensionPoints/closureCompleter.md)  
  Enhances Groovy closures with context-aware completion and templates.

- [GDSL Top Level Provider](./extensionPoints/gdslTopLevelProvider.md)  
  Introduces methods and properties available at the top level of Groovy DSLs.

- [Expected Types Contributor](./extensionPoints/expectedTypesContributor.md)  
  Describes how to contribute expected types for better code suggestions.

- [Named Argument Provider](./extensionPoints/namedArgumentProvider.md)  
  Adds support for named arguments in Groovy method calls.

---

### Syntax and Type Handling
- [Signature Hint Processor](./extensionPoints/signatureHintProcessor.md)  
  Explains how to infer method signatures based on annotations.

- [Type Calculator](./extensionPoints/typeCalculator.md)  
  Details the extension for calculating Groovy expression types dynamically.

- [Type Converter](./extensionPoints/typeConverter.md)  
  Describes converting between types for enhanced type checking.

- [Reference Type Enhancer](./extensionPoints/referenceTypeEnhancer.md)  
  Explains enhancing type resolution for Groovy references.

- [Closure Missing Method Contributor](./extensionPoints/closureMissingMethodContributor.md)  
  Details handling missing method scenarios in Groovy closures.

---

### Refactoring and Inspection
- [Custom Annotation Checker](./extensionPoints/customAnnotationChecker.md)  
  Implements checks for Groovy annotations to enforce custom rules.

- [Method May Be Static Inspection Filter](./extensionPoints/methodMayBeStaticInspectionFilter.md)  
  Filters methods that should not be flagged as static candidates.

- [Rename Helper](./extensionPoints/renameHelper.md)  
  Provides assistance during renaming refactoring operations.

---

### Debugging and Execution
- [Position Manager Delegate](./extensionPoints/positionManagerDelegate.md)  
  Assists with Groovy code debugging by mapping positions.

- [Inline AST Transformation Support](./extensionPoints/inlineASTTransformationSupport.md)  
  Supports debugging and inline transformations in Groovy scripts.

---

### Gradle and Build Systems
- [Build System Groovy New Project Wizard](./extensionPoints/buildSystemGroovyNewProjectWizard.md)  
  Extends the new project wizard for build systems like Gradle.

- [Gradle-Specific Extensions](./extensionPoints/groovyFrameworkConfigNotification.md)  
  Focuses on handling Gradle framework-specific configurations.

---

### Dynamic Features and Enhancements
- [Delegates To Provider](./extensionPoints/delegatesToProvider.md)  
  Explains adding custom delegate types for closures.

- [PSI Enhancer Category](./extensionPoints/psiEnhancerCategory.md)  
  Adds properties and methods to IntelliJ IDEA's PSI elements dynamically.

- [Map Content Provider](./extensionPoints/mapContentProvider.md)  
  Contributes custom content for Groovy maps.

- [Inspection Disabler](./extensionPoints/inspectionDisabler.md)  
  Suppresses specific inspections in Groovy code.

---

### Other Utilities
- [Unresolved Highlight Filter](./extensionPoints/unresolvedHighlightFilter.md)  
  Filters unresolved references to reduce false positives.

- [Groovy Inlay Hint Filter](./extensionPoints/groovyInlayHintFilter.md)  
  Customizes the display of inlay hints in Groovy code.

- [Convert to Java](./extensionPoints/convertToJava.md)  
  Details how to convert Groovy constructs into Java equivalents.

- [Overload Resolver](./extensionPoints/overloadResolver.md)  
  Helps with resolving overloaded method calls in Groovy.

---

### Specialized Features
- [Method Descriptor](./extensionPoints/methodDescriptor.md)  
  Describes additional metadata for Groovy methods.

- [Method Comparator](./extensionPoints/methodComparator.MD)  
  Implements custom logic for method comparison.

- [Expected Package Name Provider](./extensionPoints/expectedPackageNameProvider.md)  
  Infers expected package names for Groovy files.

- [Groovy Element Filter](./extensionPoints/groovyElementFilter.md)  
  Filters Groovy elements for specific scenarios.

- [Variable Enhancer](./extensionPoints/variableEnhancer.md)  
  Adds enhancements to variables in Groovy code.

---

