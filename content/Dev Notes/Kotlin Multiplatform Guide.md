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
- **Point 2:**
- **Point 3:**
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

### KMM Project
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

## Section 2: {{Subheading 2}}
<!-- Write about the second key point with supporting content, data, or stories. -->

## Section 3: {{Subheading 3}}
<!-- Dive into the third key point and provide further analysis, solutions, or elaboration. -->

## Conclusion


## References
 - [Create your first cross-platform app](https://www.jetbrains.com/help/kotlin-multiplatform-dev/multiplatform-create-first-app.html#examine-the-project-structure)
- [Setup an Environment KMM](https://www.jetbrains.com/help/kotlin-multiplatform-dev/multiplatform-setup.html)
- [Kotlin Multiplatform Plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform)
- [Compose Multiplatform](https://www.jetbrains.com/compose-multiplatform/)
- [Create An App With Shared Logic and UI](https://www.jetbrains.com/help/kotlin-multiplatform-dev/compose-multiplatform-create-first-app.html#examine-the-project-structure)
