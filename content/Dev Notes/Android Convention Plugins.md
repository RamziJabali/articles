---
title: Android Convention Plugins
author: Ramzi Eljabali
date: 2026-02-25
tags:
  - article
  - convention-plugin
  - build-logic
  - gradle
  - agp
  - kotlin
  - kotlin-compiler
  - kotlin-multiplatform
---

# Android Convention Plugin

## Introduction

Something I have come across recently that has caused me some issues is not tracking specific dependencies. I add dependencies into a module and forget to add all of them in a different module which led me down a rabbit hole wondering why my feature was not working as expected. The compiler didn't say anything and the app didn't crash it simply didn't fully work as expected.

This kind of issue can sometimes slip past compilation when the missing piece isn't referenced directly at compile time (for example, when it is used via reflection or only exercised in certain runtime paths). After spending a day debugging I realized I had dropped a dependency when forking over from one module to another.

## Key Points

- **Point 1:** What are convention plugin
- **Point 2:** Setting up build logic for convention plugin
- **Point 3:** Creating a convention plugin
- **Point 4:** Referencing our convention plugins in our `build.gradle`

## Section 1: What are convention plugin
Convention plugins are **Gradle plugins you write to centralize repeated build configuration**, such as:
- Plugins 
- Android configuration
- Kotlin configuration
- Dependency bundles

Instead of repeating configuration in every module:

```kotlin
android {  
    compileSdk = 36  
  
    defaultConfig {  
        minSdk = 26  
    }  
}
```

You define it once and reuse it:
```kotlin
plugins {  
    alias(libs.plugins.convention.android.application)  
}
```


A common pattern is to host convention plugins inside an **included build**, typically named:
- `build-logic`
	- The name can be different, it's convention to use `build-logic` as the name.

## Section 2: Setting up build logic for convention plugin
 
We first need to set up an included build to host our convention plugins.


1.  In the root directory we will
	1. Make a directory called:
		- `build-logic`
			- Add the `convention` directory(will be converted into a Gradle module)

```text
build-logic/
└── convention/
```

2. In the `build-logic` directory we will add the: 
	- `settings.gradle.kts`
	
```text
	build-logic/settings.gradle.kts
```

> After syncing, Gradle will treat `build-logic` as an **included build**. Inside it, `:convention` is a normal Gradle module where your plugin code lives.

Complete file:
```kotlin
rootProject.name = "build-logic"  
  
dependencyResolutionManagement {  
    repositories {  
        google()  
        mavenCentral()  
        gradlePluginPortal()  
    }  
    versionCatalogs {  
        create("libs") {  
            from(files("../gradle/libs.versions.toml"))  
        }  
    }
}  
  
include(":convention")
```
 
 This allows the convention plugins to use the same version catalog as the main project.

3. In the `build-logic` directory we will add: 
	- `gradle.properties`
		- We want to configure it as follows:

```kotlin
org.gradle.parallel=true # Makes use of parallel building  
org.gradle.caching=true # Makes use of caching  
org.gradle.configureondemand=true # Makes gradle configure only what is being used
```

4. We've already created the `convention` directory.
	- Use your normal Kotlin package under `src/main/kotlin`.

We should now have the following structure:
```
just-jog-kmm/ <your project name>
│
├── build-logic/
     ├── gradle.properties.kts
     ├── settings.gradle.kts
     └── convention/
		    ├── build.gradle.kts // WE will be adding this now
            └── src
				 └── main
					  └── kotlin
						   └── ramzi
							    └── eljabali
								     └── justjog
									      └── convention
```

5. We will be adding `build.gradle.kts` to the `convention` module we just created

Here is the complete `build.gradle.kts` file
```kotlin
import org.jetbrains.kotlin.gradle.dsl.JvmTarget  
  
group = "ramzi.eljabali.justmessage.buildlogic"  
  
plugins {  
    `kotlin-dsl`  
}  
  
// The dependencies we need within this kotlin module  
dependencies {  
    compileOnly(libs.android.gradlePlugin)  
    compileOnly(libs.android.tools.common)  
    compileOnly(libs.kotlin.gradlePlugin)  
    compileOnly(libs.compose.gradlePlugin)  
}  
  
java {  
    sourceCompatibility = JavaVersion.VERSION_17  
    targetCompatibility = JavaVersion.VERSION_17  
}  

// Compile Kotlin into Java 17-compatible bytecode.  
kotlin {  
    compilerOptions {  
        jvmTarget = JvmTarget.JVM_17  
    }  
}  
  
/**  
 * When you write custom Gradle plugins, Gradle checks that your plugin: 
 *  - Has a valid ID
 *  - Has a valid implementation class 
 *  - Has proper metadata 
 *  - Is structured correctly 
 *  - Doesn’t violate plugin publishing rules 
 */   
tasks {  
    validatePlugins {  
        enableStricterValidation = true  
        failOnWarning = true  
    }  
}  
```

Dependencies used
```toml
android-desugarJdkLibs = { module = "com.android.tools:desugar_jdk_libs", version.ref = "androidDesugarJdkLibs" }  

android-gradlePlugin = { group = "com.android.tools.build", name = "gradle", version.ref = "agp" }  

android-tools-common = { group = "com.android.tools", name = "common", version.ref = "androidTools" }  

compose-gradlePlugin = { group = "org.jetbrains.kotlin", name = "compose-compiler-gradle-plugin", version.ref = "kotlin" }  

kotlin-gradlePlugin = { module = "org.jetbrains.kotlin:kotlin-gradle-plugin", version.ref = "kotlin" }  

androidx-room-gradle-plugin = { module = "androidx.room:room-gradle-plugin", version.ref = "room" }  

ksp-gradlePlugin = { group = "com.google.devtools.ksp", name = "com.google.devtools.ksp.gradle.plugin", version.ref = "ksp" }
```

6. We want to make sure we add this `build-logic` as an `includedBuild` inside of your projects root `settings.gradle`:
```kotlin
pluginManagement {  
    includeBuild("build-logic")
    // leave remaining code as is
}
```
 
 With that out of the way we should be able to now sync our `gradle` files and have the `convention` module build successfully.
 
## Section 3: Creating a convention plugins
We've now successfully set up our `build-logic` module with the settings and properties we desire. We will now start defining convention plugins within the `convention` module we constructed.

> For clarity I have already added the different `android` configurations into `toml` under `[versions]` to be referenced later on. 
```toml
[versions]
projectApplicationId = "ramzi.eljabali.justjog"
projectVersionName = "1.0"
projectMinSdkVersion = "26"
projectTargetSdkVersion = "36"
projectCompileSdkVersion = "36"
projectVersionCode = "1"
```

Let's first make our `android` gradle code block into a convention plugin so that we can recycle it through out our project.

1. Let's start by making a class in the `kotlin` source set within our `convention` module called:
	- `AndroidApplicationConventionPlugin`
		- This class will extend `Plugin<T>` with type `Project`
			- We will then implement the `Plugin` interface
		- We will use the `apply()` to define our `android` configuration
		- We will need to first create an extension function for the `VersionCatalog` to let us reference the version catalog as `libs` similar to what we would reference it as in a regular `build.gradle`
			- Create a file called `ProjectExtensions` and add this code in there.

```kotlin
val Project.libs: VersionCatalog  
    get() = extensions.getByType<VersionCatalogsExtension>().named("libs")
```

2. We want to use the `PluginManager` to apply the `android.application` plugin. This allows us to then use `extensions.configure<ApplicationExtension>` to configure the our `android` settings.
	- We will adding majority of the android configuration here except for a couple that will be abstracted into a reusable `Project` extension function:

```kotlin
import com.android.build.api.dsl.ApplicationExtension  
import org.gradle.api.JavaVersion  
import org.gradle.api.Plugin  
import org.gradle.api.Project  
import org.gradle.kotlin.dsl.configure  
import ramzi.eljabali.justjog.convention.configureKotlinAndroid  
import ramzi.eljabali.justjog.convention.libs  
  
class AndroidApplicationConventionPlugin : Plugin<Project> {  
    override fun apply(target: Project) {  
        with(target) {  
            // equivalent to a plugins{} block  
            with(pluginManager) {  
                // applies the android application plugin  
                apply("com.android.application")  
            }  
            // This is the equivalent to `android{}` block  
            // That block exists because of `com.android.application`
            extensions.configure<ApplicationExtension> {  
                namespace = libs.findVersion("projectApplicationId").get().toString()  
  
                defaultConfig {  
                    applicationId = libs.findVersion("projectApplicationId").get().toString()  
                    targetSdk = libs.findVersion("projectTargetSdkVersion").get().toString().toInt()  
                    versionCode = libs.findVersion("projectVersionCode").get().toString().toInt()  
                    versionName = libs.findVersion("projectVersionName").get().toString()  
                }  
                packaging {  
                    resources {  
                        excludes += "/META-INF/{AL2.0,LGPL2.1}"  
                    }  
                }                
                
                buildTypes {  
                    getByName("release") {  
                        isMinifyEnabled = false  
                    }  
                }  
                configureKotlinAndroid(this)  // We will discuss this next
            }  
	    }    
    }  
}
```

3. Some `android` configs options are used in other modules like a KMM module:
	- `minSdk`
	- `compileSdk`
4. Since we're striving for reusability we will create an extension function for `Project` to allow us to set those values and be able to use it outside `ApplicationExtension`.
	- Inside of the `convention` package create a file called `KotlinAndroid` and we want to add the following function to it:
		- `fun Project.configureKotlinAndroid(commonExtension: CommonExtension<*, *, *, *, *, *>  ){`
			- This will allow us to reference it within our `AndroidApplicationConventionPlugin` like we need.
			- We're using `commonExtension` and not `ApplicationExtension` so that we can use it in other contexts outside of Android Application like a KMM module.
			- We will also add `compileOptions` block and set the `sourceCompatibility` and `targetCompatibility` to a java version of our choosing to standardize this across our `build.gradle`
				- We also want to set `isCoreLibraryDesugaringEnabled = true` to allow for some backwards compatibility conversions between different java libraries like the Java DateTime library.
			- We'll do the same thing we did in the android convention plugin:

```kotlin
internal fun Project.configureKotlinAndroid(  
    commonExtension: CommonExtension<*, *, *, *, *, *>  
) {  
    with(commonExtension) {  
        compileSdk = libs.findVersion("projectCompileSdkVersion").get().toString().toInt()  
        defaultConfig.minSdk = libs.findVersion("projectMinSdkVersion").get().toString().toInt()  
        // Standardize Java versions across build gradle  
        compileOptions {  
            sourceCompatibility = JavaVersion.VERSION_17  
            targetCompatibility = JavaVersion.VERSION_17  
            isCoreLibraryDesugaringEnabled = true  
        }  
  
        dependencies {  
            // implementation under the hood  
            "coreLibraryDesugaring"(libs.findLibrary("android-desugarJdkLibs").get())  
        }  
    
        configureKotlin() // discussed next
    }
}
```


5. We want to also abstract `compileOptions` into it's own extension function for reusability among pure kotlin modules.

```kotlin
/**  
 * Centralizing Kotlin Compiler Configuration 
 *  - Setting the jvmTarget for our Kotlin Compiler
 *  - Allow usage of APIs marked with `@ExperimentalCoroutinesApi` without 
 *      requiring annotation everywhere 
 */
internal fun Project.configureKotlin() {  
    tasks.withType<KotlinCompile>().configureEach {  
        compilerOptions {  
            jvmTarget.set(JvmTarget.JVM_17)  
            freeCompilerArgs.add(  
                "-opt-in=kotlinx.coroutines.ExperimentalCoroutinesApi"  
            )  
        }  
    }
}
```

6. Now that we have gotten all the nitty gritty out of the way we should have module structure as follows:

```
just-jog-kmm/ <your project name>
│
├── build-logic/
     ├── build.gradle.kts
     ├── settings.gradle.kts
     └── convention/
		    ├── build.gradle.kts
            └── src
				 └── main
					  ├── AndroidApplicationConventionPlugin.kt        // new
					  └── kotlin
						   └── ramzi
							    └── eljabali
								     └── justjog
									      └── convention
										       ├── KotlinAndroid.kt    // new 
										       └── ProjectExtension.kt // new
```

## Section 4: Referencing our convention plugins in our build.gradle

We're in the home stretch now. We just need to make our convention plugin known gradle and then reference it within our version catalog to for clarity.

1. We need to go back to our `build.gradle(:convention)` and register our `AndroidApplicationConventionPlugin` as an `androidApplication`.
	- This registers `AndroidApplicationConventionPlugin` as a Gradle plugin so it can be applied via its plugin ID.

> Ensure your registered `id`, your version-catalog plugin entry, and the ID you apply in consuming modules all match. Also, `implementationClass` should typically be the fully-qualified class name (including the package).

```kotlin
gradlePlugin {  
    plugins {  
        register("androidApplication") {  
            id = "justjog.convention.application"
            implementationClass = "AndroidApplicationConventionPlugin"  
        }  
    }
}
```

Here is a clear mental model to clarify all of this:

| Thing                                | Responsibility                |
| ------------------------------------ | ----------------------------- |
| `gradlePlugin {}`                    | Registers your plugin ID      |
| `AndroidApplicationConventionPlugin` | Your custom plugin logic      |
| `com.android.application`            | Creates `android {}` DSL      |
| `ApplicationExtension`               | Backing type for `android {}` |
2. We'll move over to `libs.version.toml` to define our convention plugin as an `version catalog` plugin

```kotlin
[plugins]  
convention-android-application =  { id = "justjog.convention.application", version= "unspecified"}
```
4. Sync after those additions to make our `libs` plugin visible to our gradle
5. We then go to our `build.gradle(:app)`
	1. Replace: `alias(libs.plugins.android.application)` with 
		- `alias(libs.plugins.convention.android.application)`
	2. Delete the `android {}` DSL block
	3. Sync

```kotlin
import org.jetbrains.kotlin.gradle.dsl.JvmTarget  
  
plugins {  
    alias(libs.plugins.kotlin.multiplatform)  
    alias(libs.plugins.convention.android.application)  // HERE
    alias(libs.plugins.compose.multiplatform)  
    alias(libs.plugins.compose.compiler)  
    alias(libs.plugins.compose.hot.reload)  
}  
  
kotlin {  
    androidTarget {  
        compilerOptions {  
            jvmTarget.set(JvmTarget.JVM_11)  
        }  
    }    listOf(  
        iosArm64(),  
        iosSimulatorArm64()  
    ).forEach { iosTarget ->  
        iosTarget.binaries.framework {  
            baseName = "ComposeApp"  
            isStatic = true  
        }  
    }  
    sourceSets {  
        androidMain.dependencies {  
            implementation(compose.preview)  
            implementation(libs.androidx.activity.compose)  
        }  
        commonMain.dependencies {  
            implementation(projects.core.data)  
            implementation(projects.core.domain)  
            implementation(projects.core.designsystem)  
            implementation(projects.core.presentation)  
  
            implementation(projects.feature.auth.domain)  
            implementation(projects.feature.auth.presentation)  
  
            implementation(projects.feature.chat.data)  
            implementation(projects.feature.chat.database)  
            implementation(projects.feature.chat.domain)  
            implementation(projects.feature.chat.presentation)  
  
  
            implementation(libs.jetbrains.compose.runtime)  
            implementation(libs.foundation)  
            implementation(libs.material3)  
            implementation(libs.ui)  
            implementation(libs.components.resources)  
            implementation(libs.ui.tooling.preview)  
            implementation(libs.jetbrains.compose.viewmodel)  
            implementation(libs.jetbrains.lifecycle.compose)  
        }  
  
        jvmMain.dependencies {  
            implementation(compose.desktop.currentOs)  
            implementation(libs.kotlinx.coroutines.swing)  
        }  
    }}  
  
dependencies {  
    debugImplementation(compose.uiTooling)  
}
```

You have now created a custom convention plugin for the `com.application.android` and configured it to your desire. You now have your plugins all in one centralized place and easily configurable.

## Conclusion
- Although it needed a lot of work to get started we are now able to create convention plugins for simpler things such as hilt dependencies, work manager dependencies, any group of dependencies that are required together to work.


## Benefits

| Benefit | What it solves | Example |
|---|---|---|
| Consistency | Same Android/Kotlin config everywhere | One source of truth for `compileSdk`, Java/Kotlin targets |
| Less duplication | Avoid copy/paste across modules | No repeated `android {}` blocks |
| Easier dependency hygiene | Group dependencies that must travel together | “feature-ui” plugin applies Compose + required libs |
| Faster changes | Update once, affects all modules | Change JVM target in one place |
| Onboarding clarity | Fewer Gradle concepts per module | Modules apply `alias(libs.plugins.convention...)` |
| Safer reviews | Smaller diffs in module `build.gradle` | Most logic lives in build-logic |
