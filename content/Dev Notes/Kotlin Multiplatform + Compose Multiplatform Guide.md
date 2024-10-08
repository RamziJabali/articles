---
title: Kotlin Multiplatform Guide
author: Ramzi Eljabali
date: 2024-10-02
tags:
  - article
  - kotlin
  - kotlin-multiplatform
  - swift
  - compose
  - incomplete
---

# Kotlin Multiplatform Guide

## Introduction
Hello, I am writing this as I am working on a KMM project of my own. This will serve as guide on how to get started with a KMM and CMM project. 

## Key Points
- **What is KMM**
- **Creating KMM Project**
	- How to start your Kotlin Multiplatform project.
- **Package Structure**
	- How the project is structured
	- What do the packages mean
	- What is the difference between `iosApp`, `androidApp`, and `shared`
	- What is the difference between `shared`, `iosMain`, and `androidMain`
- **What is CMM**
- **How to Setup CMM With Our KMM Project**
## Section 1: What is KMM
- KMM is Kotlin Multiplatform Mobile.
- It's meant to be a solution for cross platform development between Android and iOS. Serves as a means to share code between both platforms.
- In the case of the mobile scene even serve as a means to share UI code.
## Section 2: Creating a KMM Project 
### Setup your environment
- Before creating your KMM project you want to make sure that you have your environment set up KMM development.
1. Ensure you have [Android Studio](https://developer.android.com/studio) installed 
	- you can download the [Jetbrains Toolbox](https://www.jetbrains.com/toolbox-app/) to manage downloads and updates for Android Studio
2. Install [Xcode](https://apps.apple.com/us/app/xcode/id497799835) to run your iOS application on an simulated or real device.
3. Install [Java Development Kit(JDK)](https://www.oracle.com/java/technologies/downloads/?er=221886) 
	- Run `java -version` in your terminal to check if you have Java installed.
	- You can also run it in your Android studio terminal
4. Have the [Kotlin Multiplatform Plugin](https://kotlinlang.org/docs/multiplatform-plugin-releases.html?_gl=1*1s7ofhp*_gcl_au*MTA5MzQzNTUzNy4xNzIzODU2MDk4*_ga*MzY3MTE1NjIyLjE2OTA5MTI5ODg.*_ga_9J976DJZ68*MTcyNzkxODg3Ni42NS4xLjE3Mjc5MjA1MjcuNjAuMC4w) installed in your Android Studio IDE
5. [Kotlin plugin](https://kotlinlang.org/docs/releases.html?_gl=1*1waamr4*_gcl_au*MTA5MzQzNTUzNy4xNzIzODU2MDk4*_ga*MzY3MTE1NjIyLjE2OTA5MTI5ODg.*_ga_9J976DJZ68*MTcyNzkxODg3Ni42NS4xLjE3Mjc5MjA2OTIuNjAuMC4w#update-to-a-new-release) 
	- Should come stock with Android Studio

### Selecting KMM Project Configuration
- Now that we have the environment setup we can go 1 of 2 ways
	1. Creating the project via the [Kotlin Multiplatform Wizard](https://kmp.jetbrains.com/?_gl=1*1evsy1g*_gcl_au*MTA5MzQzNTUzNy4xNzIzODU2MDk4*_ga*MzY3MTE1NjIyLjE2OTA5MTI5ODg.*_ga_9J976DJZ68*MTcyNzkxODg3Ni42NS4xLjE3Mjc5MjA4NzAuNDMuMC4w)
	2. Creating the project via Android Studio
- I am doing it through Android Studio, doing it through the wizard is pretty straightforward.
1. Click Kotlin Multiplatform App

![[Kotlin Multiplatform App.png]]
2. I included test and selected `Regular Framework` for the iOS framework distribution as dealing with cocoa pods and Ruby has come with a plethora of difficulties when dealing with it in the past.
![[Added Tests + Regular Framework.png]]
3. Click Finish
Now you have created a KMM project.

## Section 3: Package Structure
KMM package structure is going to as follows
```bash
JustJogKMM
|
├── androidApp
|		├── src
|			├── main
|			└── res
|
├── gradle
|
├── iosApp
|		├── assets.xcassets
|		├── Preview Content
|		└── iosApp.xcodeproj
|
└── shared
		├── androidMain
		├── androidUnitTest
		├── commonMain
		├── commonTest
		├── iosMain
		└── iosTest
```

### What is the difference between `androidApp`, `iosApp`, and `shared`

### What is the difference between `androidMain`, `shared`, and `iosMain`

## What is CMM
CMM or Compose Multiplatform mobile is a declarative framework for sharing UIs across multiple platforms. Based on Kotlin Multiplatform and Jetpack Compose.

## How to Setup CMM With Our KMM Project
1. You have a KMM project setup.
2. Under `Gradle Scripts`
	- Go to `gradle.properties`
3. Setup the versions for `kotlin`, `agp`, and `compose`
	- Add them to the bottom of the file.
```kotlin
kotlin.version=2.0.20
agp.version=8.0.1
compose.version=1.6.11
```
4. While still in the `gradle.properties` we need to opt into experimental Compose Multiplatform API's
```kotlin
org.jetbrains.compose.experimental.uikit.enabled=true
```
5. Now go into `build.gradle.kts` project level
	- We need to specify the compose gradle plugin
```kotlin 
plugins {  
    // this is necessary to avoid the plugins to be loaded multiple times  
    // in each subproject's classloader
    alias(libs.plugins.androidApplication) apply false         alias(libs.plugins.androidLibrary) apply false  
    alias(libs.plugins.jetbrainsCompose) apply false  
    alias(libs.plugins.compose.compiler) apply false  
    alias(libs.plugins.kotlinMultiplatform) apply false  
}
```
6. add `alias(libs.plugins.jetbrainsCompose)` and `alias(libs.plugins.compose.compiler)` to `plugins` of
	1. `build.gradle` androidApp level
	2. `build.gradle` shared level 
7. Once you have done that ensure that in your `build.gradle` shared level you add `isStatic = true`
	- Otherwise the shared module will not be found in XCode
```kotlin
listOf(  
    iosX64(),  
    iosArm64(),  
    iosSimulatorArm64()  
).forEach {  
    it.binaries.framework {  
        baseName = "shared"  
        isStatic = true  
    }  
}
```
8. Add the Compose Multiplatform dependencies in the `sourceSets` block for the shared code:
```kotlin
val commonMain by getting {
    dependencies {
        implementation(compose.runtime)
        implementation(compose.foundation)
        implementation(compose.material)
        @OptIn(org.jetbrains.compose.ExperimentalComposeLibrary::class)
        implementation(compose.components.resources)
    }
}
```
9.Go to `settings.gradle` 
- Add the Compose Multiplatform Maven path so that it finds 
`maven("https://maven.pkg.jetbrains.space/public/p/compose/dev")`
- Paste this plugins block in the `pluginManagement` block:
```kotlin
plugins {
    val kotlinVersion = extra["kotlin.version"] as String
    val agpVersion = extra["agp.version"] as String
    val composeVersion = extra["compose.version"] as String

    kotlin("jvm").version(kotlinVersion)
    kotlin("multiplatform").version(kotlinVersion)
    kotlin("android").version(kotlinVersion)

    id("com.android.application").version(agpVersion)
    id("com.android.library").version(agpVersion)

    id("org.jetbrains.compose").version(composeVersion)
}
```
- All together:
```kotlin
pluginManagement {  
    repositories {  
        google()  
        gradlePluginPortal()  
        mavenCentral()  
        maven("https://maven.pkg.jetbrains.space/public/p/compose/dev")  
    }  
    plugins {  
        val kotlinVersion = extra["kotlin.version"] as String  
        val agpVersion = extra["agp.version"] as String  
        val composeVersion = extra["compose.version"] as String  
  
        kotlin("jvm").version(kotlinVersion)  
        kotlin("multiplatform").version(kotlinVersion)  
        kotlin("android").version(kotlinVersion)  
  
        id("com.android.application").version(agpVersion)  
        id("com.android.library").version(agpVersion)  
  
        id("org.jetbrains.compose").version(composeVersion)  
    }  
}
```

---
- Now that all the set up has been done we can head to `shared` -> `commonMain` and create our shared Compose code!

## Adding Compose View to Both Android and iOS with CMM
- I went and created a shared view in `shared` -> `commonMain`
```kotlin
@Composable  
fun GreetingView(name: String) {  
    var counterMutableState by remember { mutableStateOf(0) }  
    Column(  
        modifier = Modifier.fillMaxSize(),  
        horizontalAlignment = Alignment.CenterHorizontally,  
        verticalArrangement = Arrangement.Center  
    ) {  
        Text(text = "Hello $name!")  
        Button(onClick = { counterMutableState++ }) {  
            Text("+ 1")  
        }  
        TextButton(onClick = { counterMutableState++ }) {  
            Text("Counter: $counterMutableState")  
        }  
  
        Button(onClick = { counterMutableState = 0 }) {  
            Text("Reset Counter")  
        }  
    }}
```

- From here I went to `androidApp`-> `MainActivity` and referenced it in my UI.
- To use it on iOS I went to `shared` -> `iosMain` and created a `kotlin` file and added the following
```kotlin
fun MainViewController() = ComposeUIViewController {  
    GreetingView(Greeting().greet())  
}
```

- You then go to `iosApp` -> `iosApp.xcodeproj` right click and click `open in` -> `Xcode`
	- It will open the project for you in the Xcode IDE
	- Build and Run the code as is to sync it with our code
- Create a `.swift` file under `iosApp` and call it `ComposeView`
```swift
import Foundation
import SwiftUI
import shared

struct ComposeView: UIViewControllerRepresentable {
    func updateUIViewController(_ uiViewController: UIViewControllerType, context: Context) {}

    func makeUIViewController(context: Context) -> 
    some UIViewController {
        AppKt.MainViewController()
    }
}
```

- Go to `ContentView` and update it
```swift
struct ContentView: View {

var body: some View {
        ComposeView()
    }
}
```

### Results
![Android]()
![iOS]()

## Conclusion


## References
- [Create your first cross-platform app](https://www.jetbrains.com/help/kotlin-multiplatform-dev/multiplatform-create-first-app.html#examine-the-project-structure)
- [Setup an Environment KMM](https://www.jetbrains.com/help/kotlin-multiplatform-dev/multiplatform-setup.html)
- [Kotlin Multiplatform Plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform)
- [Compose Multiplatform](https://www.jetbrains.com/compose-multiplatform/)
- [Create An App With Shared Logic and UI](https://www.jetbrains.com/help/kotlin-multiplatform-dev/compose-multiplatform-create-first-app.html#examine-the-project-structure)
- [Compose Multiplatform GitHub](https://github.com/JetBrains/compose-multiplatform)
- [Kotlin GitHub](https://github.com/JetBrains/kotlin)
