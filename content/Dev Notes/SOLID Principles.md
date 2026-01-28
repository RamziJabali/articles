---
title: SOLID Principles
author: Ramzi Eljabali
date: 2025-06-06
tags:
  - article
  - interface
  - abstraction
  - SOLID
---

# Solid Principles

## Introduction

SOLID  principles are object-oriented design guidelines you can implement in any development process.

## Key Points

- **S - Single Responsibility:** 
	- Make sure the class/file/code container has a single responsibility. A single purpose.
	- Ex:
		- `WebsocketListener` it's only responsibility is to connect, reconnect, disconnect, onFailure. All these have to do with one thing websockets.

- **O - Open/Closed:**
	- Open for extension closed for modification. Extending the existing behavior without having to modify the underlying behavior.
	- Ex:
		- `interface Animal()`
			- `fun walk()`
			- `fun eat()`
			- `fun attack()`
		- Creating a `Cat` as an Implementation of `Animal` you're extending on the base class `Animal` instead of having to modify `Animal` classes `eat()` or `attack()` for the `Cat` instance.

- **L - Liskov Substitution:**
	- If you have a parent class you should be able to pass it a child class without breaking any behavior.
	- Ex:
		- Anywhere an `Animal` is expected, a `Cat` should work **without surprises**. As it is an implementation of `Animal`
		- No behavior changes, no broken assumptions

- **I - Interface Segregation:**
	- You shouldn't be forced to implement a function you don't need.
	- It is better to have many specific interfaces than one general-purpose interface.
	- Ex:
		- `interface Animal`
			- `fun walk()` <-- not every animal walks some swim
			- `fun eat()` 
			- `fun attack()` <- not every animal attacks
			- `fun fly()` <---- not needed as it is not applicable to 

```kotlin
interface Animal {
    fun eat()
}

interface Walkable {
    fun walk()
}

interface Attackable {
    fun attack()
}

interface Flyable {
    fun fly()
}

```

```kotlin
class Crow : Animal, Walkable, Flyable {
    override fun eat() {}
    override fun walk() {}
    override fun fly() {}
}

class Cat : Animal, Walkable, Attackable {
    override fun eat() {}
    override fun walk() {}
    override fun attack() {}
}
```

- **D - Dependency Inversion:**
	- You should depend on abstractions instead of concrete implementation. An abstraction is an `interface` or `abstract` class. You're abstracting the need without  having the how.
	- Ex:
		- Having a database class that has a concrete implementation. If you were to want to replace the backend to something else, you will have to replace all where the concrete implementation is being referenced. 
		- Instead with an abstraction you can change the implementation without needing to update/change the points of reference. Simply replacing the implementation will suffice making it extremely simple to implement big features without the fear of a major refactor.


