---
title: Build Konfig
author: Ramzi Eljabali
date: 2026-03-09
tags:
  - cook-book
  - build-konfig
  - copy-paste
---
# Cookbook

1. Dependencies:

```toml
[versions]
buildkonfig = "0.17.1"

[libraries]
# BuildKonfig  
buildKonfig-compiler = {group = "com.codingfeline.buildkonfig",  name = "buildkonfig-compiler", version.ref = "buildkonfig"}  

buildKonfig-gradlePlugin= {group = "com.codingfeline.buildkonfig",  name = "buildkonfig-gradle-plugin", version.ref = "buildkonfig"}

[plugins]
buildkonfig = { id = "com.codingfeline.buildkonfig", version.ref = "buildkonfig" }
```


2. Build-logic

```kotlin
dependencies{
	implementation(libs.buildKonfig.compiler)  
	implementation(libs.buildKonfig.gradlePlugin)
}
```

3. Convention Plugin

```kotlin
import com.android.build.gradle.internal.cxx.configure.gradleLocalProperties  
import com.codingfeline.buildkonfig.compiler.FieldSpec  
import com.codingfeline.buildkonfig.gradle.BuildKonfigExtension  
import org.gradle.api.Plugin  
import org.gradle.api.Project  
import org.gradle.internal.Actions.with  
import org.gradle.kotlin.dsl.apply  
import org.gradle.kotlin.dsl.configure  
import ramzi.eljabali.justmessage.convention.libsFindPlugin  
  
class BuildKonfigConventionPlugin : Plugin<Project> {  
    override fun apply(target: Project) {  
        with(target) {  
            with(pluginManager) {  
                apply(libsFindPlugin("buildKonfig"))  
            }  
            extensions.configure<BuildKonfigExtension> {  
		        packageName = pathToPackageName() 
                defaultConfigs {  
                    val apiKey = gradleLocalProperties(  
                        rootDir,  
                        rootProject.providers  
                    ).getProperty("API_KEY") ?: IllegalStateException(  
                        "Missing `API_KEY` property in local.properties."  
                    )  
                    buildConfigField(FieldSpec.Type.STRING, "API_KEY", apiKey as String)  
                }  
            }        
        }    
    }  
}
```

4. Register Plugin

```kotlin
gradlePlugin { 
	plugins {
		register("buildKonfig") {  
		    id = "just.message.build.konfig"  
		    implementationClass = "BuildKonfigConventionPlugin"  
		}
	}
}
```