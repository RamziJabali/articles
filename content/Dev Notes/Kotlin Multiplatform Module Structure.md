---
title: Kotlin Multiplatform Module Structure
author: "{{author}}"
date: 2026-01-31
tags:
  - article
  - multi-module
  - kotlin-multiplatform
  - compose-multiplatform
  - architecture
---

# Kotlin Multiplatform Multi Module structure

## Introduction
A quick Compose/Kotlin multiplatform multi module architecture approach to make code comprehension clean.

## Key Points

- **Kotlin Modules vs Kotlin Multiplatform Modules** 
- **Multi Module Graph**
## Section 1: Kotlin Modules vs Kotlin Multiplatform Modules
- A normal Kotlin module (JVM/Android) targets one runtime one main source set
	- JVM(`kotlin(""jvm)`) - src/main/kotlin
	- Android(`kotlin("android")`) - src/main/kotlin
	- All the code is compiled for one platform.

- Kotlin Multiplatform (KMP / KMM) module
	- A Kotlin Multiplatform module is multi-target by design.
	- Instead of “one module = one platform”, it’s:
		- one module → multiple targets → multiple source sets

- The key difference is the 
	- source set hierarchy

```
src/
  commonMain/
  commonTest/
  androidMain/
  androidTest/
  iosMain/
  iosTest/
```

- Gradle configuration and compilation model

```kotlin
plugins {
    kotlin("multiplatform")
}

kotlin {
    androidTarget()
    iosX64()
    iosArm64()

    sourceSets {
        val commonMain by getting
        val androidMain by getting
        val iosMain by getting
    }
}
```

- `commonMain`  
    → pure Kotlin (no Android / iOS APIs)

- `androidMain`  
    → Android-specific implementations

- `iosMain`  
    → iOS-specific implementations

- `commonMain` is compiled once

- Platform source sets are compiled per target

- Gradle enforces platform boundaries

- `expect / actual` and dependency isolation are enabled
## Section 2: Multi Module Approach

- Instead of separating the whole system into 
	- `core`
	- `domain`
	- `date`
	- `presentation`

- We want an approach that mixes those into a more organized multi module approach.
	- `core`
		- Contains logic that is **shared across multiple features** and is safe for reuse.
		- It is still layered internally to preserve separation of concerns:
			- `core:presentation`
			    - Shared UI primitives
			    - Navigation helpers
			    - State holders / base ViewModels
			- `core:domain`
			    - Shared models
			    - Common use cases
			    - Error types
			- `core:data`
			    - Shared repositories or data abstractions
			    - Network helpers
			- `core:designsystem`
			    - Reusable Compose components
			    - Theme, typography, spacing
			    - Custom design library (UI contract, not screens)
	
	- `feature`
		- `jog` - applying those buckets on the feature level, allowing for code cleanliness and isolation
			- `domain`
			- `data`
			- `database`
			- `presentation`
		- `settings`
			- `domain`
			- `presentation`

## Conclusion

This approach reduces overall module size while increasing **clarity, cohesion, and isolation**.

By slicing the system vertically by feature and keeping `core` focused on true shared infrastructure, we create an architecture that is:

- Easier to reason about
- Safer to change
- More scalable as the codebase grows   
