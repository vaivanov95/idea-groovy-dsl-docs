# Build System Groovy New Project Wizard Extension Point

The `buildSystemGroovyNewProjectWizard` extension point enables plugin developers to integrate custom Groovy build systems into the IDE's new project creation wizard. This extension point facilitates the creation of tailored Groovy project setups by defining specific build system configurations and providing user-friendly wizard steps.

---

## Extension Point Declaration

```xml
<extensionPoint qualifiedName="com.intellij.newProjectWizard.groovy.buildSystem"
                interface="org.jetbrains.plugins.groovy.config.wizard.BuildSystemGroovyNewProjectWizard"
                dynamic="true"/>
```

---

## Purpose and Capabilities

### **1. Extending the New Project Wizard**
This extension point allows developers to:
- Introduce new Groovy build systems (e.g., Gradle, Maven).
- Customize the project creation process with specific settings for their build system.

### **2. Build System Integration**
By implementing the `BuildSystemGroovyNewProjectWizard` interface, plugins can:
- Define wizard steps for configuring build-specific options.
- Set up initial project files and structures, such as build scripts and source directories.
- Generate sample code for quick project initialization.

---

## Key Interface

### `BuildSystemGroovyNewProjectWizard`

This interface allows plugins to define custom Groovy build systems in the new project wizard.

```kotlin
interface BuildSystemGroovyNewProjectWizard : NewProjectWizardMultiStepFactory<GroovyNewProjectWizard.Step> {
  companion object {
    val EP_NAME = ExtensionPointName<BuildSystemGroovyNewProjectWizard>("com.intellij.newProjectWizard.groovy.buildSystem")
  }
}
```

#### Key Properties:
- **`name`**: The display name of the build system (e.g., `Gradle`).
- **`ordinal`**: Determines the order of build systems in the wizard.

#### Key Method:
- **`createStep(parent: GroovyNewProjectWizard.Step): NewProjectWizardStep`**:
  Defines the main wizard step for the build system.

---

## Example Implementation: Gradle Integration

### Gradle Groovy New Project Wizard
This implementation adds support for creating Groovy projects using Gradle.

```kotlin
class GradleGroovyNewProjectWizard : BuildSystemGroovyNewProjectWizard {

  override val name = "Gradle"

  override val ordinal = 200

  override fun createStep(parent: GroovyNewProjectWizard.Step): NewProjectWizardStep =
    Step(parent).nextStep(::AssetsStep)

  class Step(parent: GroovyNewProjectWizard.Step) :
    GradleNewProjectWizardStep<GroovyNewProjectWizard.Step>(parent),
    BuildSystemGroovyNewProjectWizardData by parent {

    private val addSampleCodeProperty = propertyGraph.property(true)
      .bindBooleanStorage("ADD_SAMPLE_CODE")

    var addSampleCode by addSampleCodeProperty

    init {
      gradleDsl = GradleDsl.GROOVY
    }

    override fun setupSettingsUI(builder: Panel) {
      setupJavaSdkUI(builder)
      setupGroovySdkUI(builder)
      setupGradleDslUI(builder)
      setupParentsUI(builder)
      setupSampleCodeUI(builder)
    }

    private fun setupSampleCodeUI(builder: Panel) {
      builder.row {
        checkBox("Add sample code")
          .bindSelected(addSampleCodeProperty)
      }
    }

    override fun setupProject(project: Project) {
      val builder = GradleJavaModuleBuilder()
      setupBuilder(builder)
      setupBuildScript(builder) {
        when (val groovySdk = groovySdk) {
          null -> withPlugin("groovy")
          is FrameworkLibraryDistributionInfo -> withGroovyPlugin(groovySdk.version.versionString)
          is LocalDistributionInfo -> {
            withPlugin("groovy")
            withMavenCentral()
          }
        }
        withJUnit()
      }
      setupProject(project, builder)
    }
  }

  private class AssetsStep(private val parent: Step) : AssetsNewProjectWizardStep(parent) {
    override fun setupAssets(project: Project) {
      addEmptyDirectoryAsset("src/main/groovy")
      addEmptyDirectoryAsset("src/main/resources")
      if (parent.addSampleCode) {
        val sourcePath = "src/main/groovy/Main.groovy"
        addTemplateAsset(sourcePath, "Groovy Sample Code", "PACKAGE_NAME" to parent.groupId)
      }
    }
  }
}
```

### Capabilities Demonstrated
1. **User-Friendly Wizard Steps**:
   - Provides settings for SDKs, Gradle DSL, and sample code generation.

2. **Build Script Configuration**:
   - Adds the Groovy plugin and dependencies dynamically based on the selected SDK.

3. **Project Structure**:
   - Creates directories and adds sample Groovy code for a quick start.

---

## Use Cases and Benefits

### **1. Framework-Specific Project Templates**
Plugins can use this extension point to:
- Add preconfigured project templates for specific Groovy-based frameworks like Grails or Spock.

### **2. Enhanced Developer Experience**
- Simplifies the process of setting up new projects.
- Reduces configuration overhead by automating build script and structure setup.

### **3. Flexibility for Custom Build Systems**
- Supports integrating alternative or custom build systems tailored for specific use cases.

---

## Conclusion

The `buildSystemGroovyNewProjectWizard` extension point streamlines the creation of Groovy projects by enabling plugins to define custom build systems. By integrating with the IDE's new project wizard, it enhances developer productivity and ensures seamless setup for Groovy-based projects.

