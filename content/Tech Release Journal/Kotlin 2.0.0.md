---
title: Kotlin 2.0.0
author: Ramzi Eljabali
date: 2024-09-14
tags:
  - article
---

# Kotlin 2.0.0

## Introduction

With the release of Kotlin 2.0.0 a lot of new tech had been uncovered and has become stable for devs to to use.

I was especially excited, as I wanted to see what new features Kotlin was getting. I was hoping to see if Jetpack Compose would be mentioned one way or another in this release and if so in what capacity.

I will be covering items that have made me feel excited!

## Key Points

- **Point 1:** Kotlin K-2 Compiler 
	- Strong benchmarks for build times
	- Language feature improvements
	- Impactful smart cast improvements
- **Point 2:** New Compose Compiler Plugin
	- New Compose compiler Gradle plugin
- **Point 3:** Experimental Power-Assert Compiler Plugin
	- This will help with unit testing
- **Point 4:** Extensible Data Arguments
	- Although coming in 2.2.0 is worthy of mentioning
- **Point 5:** Guarded Conditions
	- Will help with `when` conditions
- **Point 6:** Context Sensitive Resolution
	- Will eliminate the need to reference the base class for `enum` and `sealed` classes in `when` conditions 
- 

## Kotlin K-2 Compiler
- With the release of Kotlin 2.0.0, the new Kotlin K-2 compiler is used by default and is stable for all target  platforms.
- The K-2 compiler brings major performance improvements, speeds up new language feature development, unifies all platforms that Kotlin supports, and provides better architecture for Multiplatform Projects.
- The K-2 compiler introduces extremely fast build times. All comparisons were done with version 1.9.23.
	1. Up to 94% compilation speed gains for clean builds
	2. Up to 488% faster on initialization phase
	3. Up to 376% faster in the analysis phase

## New Compose Compiler Plugin
- The Jetpack Compose compiler, that converts `composables` to Kotlin code, has now been merged into the Kotlin repository.
	- This will help transition Compose projects to Kotlin 2.0.0, as the compiler will always ship with Kotlin.
	- This also bumps the Compose compiler to version 2.0.0.

## Experimental Power-Assert Compiler Plugin
- A testing plugin that generates failure messages that include intermediate values of the assertion expression.
- When an assertion fails in a test, the improved error message shows the values of all the variables and subexpressions within the assertion.
- This makes it clear which part of the condition causes failure.
- By default, it transforms `assert()` function calls but can also transform other functions, such as `require()`, `check()`, and `assertTrue()`.
- Example error message provided by the plugin
```text
Incorrect length
assert(hello.length == world.substring(1, 4).length) { "Incorrect length" }
       |     |      |  |     |               |
       |     |      |  |     |               3
       |     |      |  |     orl
       |     |      |  world!
       |     |      false
       |     5
       Hello
```


```kotlin
// build.gradle.kts
plugin {
	kotlin("multiplatform") version "2.0.0"
	kotlin("plugin.power-assert") version "2.0.0"
}

powerAssert {
	functions = listOf("kotlin.assert", "kotlin.test.assertTrue", "kotlin.test.assertEquals", "kotlin.test.assertNull")
}
```

- Since it's experimental you will have to use this annotation
	- `@OptIn(ExperimentalKotlinGradlePluginApi::class)`

[For more information about Power Assert](https://kotlinlang.org/docs/power-assert.html)

## Extensible Data Arguments
- Though is coming out in a later version, version 2.2.0, is a pretty exciting new addition to Kotlin.
- 

## Conclusion
<!-- Summarize the article, restating the key ideas. You can also end with a call to action or closing thoughts. -->

## References
<!-- If you have used any sources, list them here for further reading or citation purposes. -->
- [Source 1](link)
- [Source 2](link)

