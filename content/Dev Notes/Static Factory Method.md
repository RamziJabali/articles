---
title: Static Factory Method
date: 2020-03-21
tags:
  - java
  - design-pattern
  - factory
  - article
---
### What is a Static Factory Method: Simply a static method that returns an instant of the class.

Advantages:

1. Unlike constructors they have names.
2. Different signatures.
3. Unlike constructors, they are not required to make a new Object every time they are invoked.
5. Unlike constructors, they can return an object of any subtype of their return type.

Disadvantages:

1. Only disadvantage of providing static factory methods is that classes without public or protected constructors cannot be subclassed.
3. They are not readily distinguishable from other static methods.


Example:
```java
 private CoordinatesStaticFactoryMethod(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
  public static final CoordinatesStaticFactoryMethod fromXY(double x, double y) {
        return new CoordinatesStaticFactoryMethod(x, y);
    }

   public static final CoordinatesStaticFactoryMethod fromAngles(double angle, double distance) {
       return new CoordinatesStaticFactoryMethod(distance * Math.cos(angle), distance * Math.sin(angle));
    }
```