---
title: Kotlin 2.0.0
author: Ramzi Eljabali
date: 2024-09-14
tags:
  - journal
  - kotlin
  - release-notes
  - k2-compiler
  - compose-compiler
  - bench-marks
  - smart-cast
  - power-assert
  - context-sensitive-resolution
  - guarded-conditions
---

# Kotlin 2.0.0

## Introduction

With the release of Kotlin 2.0.0 a lot of new tech had been uncovered and has become stable for devs to to use.

I was especially excited, as I wanted to see what new features Kotlin was getting. I was hoping to see if Jetpack Compose would be mentioned one way or another in this release and if so in what capacity.

I will be covering items that have made me feel excited!

## Key Points

- **Point 1:** Kotlin K2 Compiler 
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
- **Point 6:** Context-Sensitive Resolution
	- Will eliminate the need to reference the base class for `enum` and `sealed` classes in `when` conditions 
- 

## Kotlin K2 Compiler
- With the release of Kotlin 2.0.0, the new Kotlin K2 compiler is used by default and is stable for all target  platforms.
- The K2 compiler brings major performance improvements, speeds up new language feature development, unifies all platforms that Kotlin supports, and provides better architecture for Multiplatform Projects.
- The K2 compiler introduces extremely fast build times. All comparisons were done with version 1.9.23.
	1. Up to 94% compilation speed gains for clean builds
	2. Up to 488% faster on initialization phase
	3. Up to 376% faster in the analysis phase
- To get the full K2 experience use the K2 Mode plugin 
	- Using K2 mode on IntelliJ and Android Studio you have to be on version 2024.2.

- Smartcast improvements:
	- In Kotlin 2.0.0, if you combine type checks for objects with an `or` operator (`||`), a smart cast is made to their closest common supertype. Before this change, a smart cast was always made to the `Any` type.
	- Previously if you had a nullable variable type you would only be able to smartcast it within the scope of the `if` expression which evaluated it to not `null`. This information would not be spread outside of that scope. From Kotlin 2.0.0, if you declare a variable before using it in your `if`, `when`, or `while` condition, then any information collected by the compiler about the variable will be accessible in the corresponding block for smart-casting.

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
- Though is not confirmed, is a pretty exciting potential new addition to Kotlin.
- `dataarg` similar to `varargs` but for data classes.
## Guarded Conditions
- Used to help us check multiple conditions within a `when` block without repeating the variable name.
- Also allows for `if` conditions on to the branches of `when` statments.
- Previous implementation: 
```kotlin
val searchPanel = SelectedSearchPanel
when {
	searchPanel is SearchPanel.NewsPanel && !searchPanel.isBlocked -> item { NewsResult() }
	searchPanel is SearchPanel.SpeakersPanel -> item {...}
	searchPanel is SearchPanel.TalksPanel -> item {...}
}
```
- Kotlin 2.0.0
```kotlin
when(searchPanel = SelectedSearchPanel) {
	is SearchPanel.NewsPanel if !searchPanel.isBlocked -> {...}
	is SearchPanel.SpeakersPanel -> {...}
	is SearchPanel.TalksPanel -> {...}
}
```

## Context-Sensitive Resolution
- This is coming out in Kotlin 2.2.0.
- This adds on to what we previously discussed, by no longer requiring you to add the base class to your `sealed` types or `enums`.
- So going of the previous example we can completely remove the `SearchPanel` base class as follows.
```kotlin
when(searchPanel = SelectedSearchPanel) {
	is NewsPanel if !searchPanel.isBlocked -> {...}
	is SpeakersPanel -> {...}
	is TalksPanel -> {...}
}
```

## Conclusion
- Excluding speculated features like `dataargs`, these additions will help cut down on build times, add cleanliness to our code, and remove pain points that were there previously.
- I am very excited to try these out in my projects, as the highlighted features code issues that I have come across while trying to write clean and concise code.

## References
- [K2 Migration Guide](https://kotlinlang.org/docs/k2-compiler-migration-guide.html#performance-improvements)
- [K2 Mode](https://blog.jetbrains.com/idea/2024/08/meet-the-renovated-kotlin-support-k2-mode/)
- [Power Assert Compiler Plugin](https://kotlinlang.org/docs/whatsnew20.html#experimental-kotlin-power-assert-compiler-plugin)
- [Smart Cast Improvements](https://kotlinlang.org/docs/whatsnew20.html#smart-cast-improvements)
- [Kotlin Features Survey #2](https://blog.jetbrains.com/kotlin/2021/06/kotlin-features-survey-edition-2/)
- [Improve resolution using expected type #379](https://github.com/Kotlin/KEEP/issues/379)

