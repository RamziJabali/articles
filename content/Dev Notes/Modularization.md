---
title: Modularization
author: Ramzi Eljabali
date: 2024-09-17
tags:
  - article
  - modularization
  - android
---

# Modularization

## Introduction
What is **modularization**?
- This is the practice of breaking down a code base into loosely coupled smaller self containing parts.
- In Android, a project is considered to be multi-module if there is multiple Gradle modules.
Who is **modularization** for?
- It's for everyone who wants their codebase to be broken down into independent pieces that are reusable and self-sustainable.
- In my opinion this becomes a viable option when your codebase is big and hard to maintain.

## Key Points
- **What is Modularization** 
- **Why Modularization Is Important** 
- **Who Modularization Is For**
- **Modularization Terminology & Principles*
- **Modularization Benefits**
- **How To Modularize Your Application**
- **Modularization Example**

## 1. What Is Modularization
This is the practice of breaking down a code base into loosely coupled, smaller, self containing parts. Each part being it's own module.

## 2. Why Modularization Is Important
It's important because it allows for your codebase to be broken down into independent reusable pieces called `modules`. 

This then allows these modules easier maintainability and scalability, which in turn extends the same benefits to your codebase.
## 3. Who Is Modularization For
1. **Developers** who want their codebase to be broken down into independent, reusable, and self-sustainable pieces, simplifying debugging, updating, and reusability.
2. **Projects with Complex or Growing Codebases**, where modularization becomes a viable option when the code is large, difficult to maintain, or frequently evolving.

## Modularization Terminology & Principles
- **Module**:
	- A self-contained piece of code, typically representing a specific functionality or feature. Modules can be reused across different parts of an application.
	- Is a part of your code that has high module cohesion and low coupling with other modules.
- **Modular Design:** The architectural approach where the system is broken down into modules with clear boundaries, responsibilities, and minimal dependencies between them.
- **Module Cohesion**:
	- Degree of interaction within your module.
- **Module Coupling**:
	- Degree of dependence and interaction between modules.
- **MAXMIN**:
	- Maximal interaction within module, minimal interaction between modules
	- MAX modular cohesion MIN modular coupling
- **Abstraction:** The process of exposing only the essential features of a module and hiding its complexity, allowing other parts of the system to interact with it at a higher level.
- **Encapsulation:** The concept of bundling data(attributes) and functions that operate on data into a singular class, restricting the access to some of the objects components.
- **Separation of Concerns (SoC):** A design principle where different parts of the codebase (modules) handle distinct concerns or functionality, improving modularity and clarity.
- **Package:** A collection of related modules bundled together. 
- **Library:** A reusable collection of precompiled modules or functions that developers can integrate into their applications. 
## Modularization Benefits

| Benefit       | Summary                                                                                                                                                                                                                                                                                                                                                      |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Scalability   | In a tightly coupled codebase a single change can trigger a cascade of alterations in seemingly unrelated parts of code. A properly modularized project will embrace the [separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns) principle and therefore limit the coupling. This empowers the contributors through greater autonomy. |
| Encapsulation | Encapsulation means that each part of your code should have the smallest possible amount of knowledge about other parts. Isolated code is easier to read and understand.                                                                                                                                                                                     |
| Testability   | A testable code is one where components can be easily tested in isolation.                                                                                                                                                                                                                                                                                   |
## How To Modularize Your Application
According to Android documentation we should be modularizing our applications by **features** and **layers**.
![[Pasted image 20240917210519.png]]



## Conclusion
<!-- Summarize the article, restating the key ideas. You can also end with a call to action or closing thoughts. -->

## References
<!-- If you have used any sources, list them here for further reading or citation purposes. -->
- [Source 1](link)
- [Source 2](link)

