---
title: Android Terminology
date: 2024-06-08
tags:
  - android
  - terminology
  - core
  - incomplete
---
1. Android SDK:
	1. Android Software Development Kit: A set of development tools and libraries provided by google for creating apps for the Android OS.

2. Gradle:
	1. Gradle is the build automation tool used in Android Studio to manage dependencies and build Android projects.

3. .APK(Android Package) file is an archive file that contains the required resources to launch an application. This is what Android devices use to install applications.

4. .aab(Android App Bundle) is an archive file, that contains all the contents of the Android app project. It even contains metadata that is not required at run-time. AAB cannot be installed on Android devices.
## Understand the architecture of the Android System:

Android Operating system is a multi-user Linux system in which each app is a different user. By default the OS assigns each app a unique user ID, which is used by the system and is unknown to other apps. The system sets permissions for all files in an app so that only the user ID assigned to the app can access them.

It's possible for two apps to have the same Linux user ID, in which case they are able to access each others files. They have to be signed with the same certificate.

## App Components
- They are the basic building blocks of an Android application.
- Each component is an entry point through which the system or the user can enter your app. Some components depend on others.

### Components
1. Activities
2. Services
3. Broadcast receivers
4. Content providers

### Activities



