---
title: Singletons
date: 2020-03-21
tags:
  - java
  - design-pattern
  - article
  - singleton
---
### Enforce The Singleton property with a private constructor or an enum type

A `Singleton` is simply a class that is instantiated exactly once, can have only one object (an instance of the class) at a time.

To design a singleton class:

1) Make constructor as private.

2) Write a static method that has return type object of this singleton class. Here, the concept of Lazy initialization is 
used to write this static method.

Example:
```java
//Singelton with public final field
public class Elvis{
  public static final Elvis INSTANCE = new Elvis();
  
  private Elvis(){....}
  
  public void leaveTheBuilding(){....}
 }
```
Could also be done as follows:
```java
// Java program implementing Singleton class 
// with getInstance() method 
class Singleton 
{ 
    // static variable single_instance of type Singleton 
    private static Singleton single_instance = null; 
  
    // variable of type String 
    public String s; 
  
    // private constructor restricted to this class itself 
    private Singleton() 
    { 
        s = "Hello I am a string part of Singleton class"; 
    } 
  
    // static method to create instance of Singleton class 
    public static Singleton getInstance() 
    { 
        if (single_instance == null) 
            single_instance = new Singleton(); 
  
        return single_instance; 
    } 
} 
```
Here is what the driver class looks like:

```java
// Driver Class 
class Main 
{ 
    public static void main(String args[]) 
    { 
        // instantiating Singleton class with variable x 
        Singleton x = Singleton.getInstance(); 
  
        // instantiating Singleton class with variable y 
        Singleton y = Singleton.getInstance(); 
  
        // instantiating Singleton class with variable z 
        Singleton z = Singleton.getInstance(); 
  
        // changing variable of instance x 
        x.s = (x.s).toUpperCase(); 
  
        System.out.println("String from x is " + x.s); 
        System.out.println("String from y is " + y.s); 
        System.out.println("String from z is " + z.s); 
        System.out.println("\n"); 
  
        // changing variable of instance z 
        z.s = (z.s).toLowerCase(); 
  
        System.out.println("String from x is " + x.s); 
        System.out.println("String from y is " + y.s); 
        System.out.println("String from z is " + z.s); 
    } 
} 
```

Output:
```java
String from x is HELLO I AM A STRING PART OF SINGLETON CLASS
String from y is HELLO I AM A STRING PART OF SINGLETON CLASS
String from z is HELLO I AM A STRING PART OF SINGLETON CLASS


String from x is hello i am a string part of singleton class
String from y is hello i am a string part of singleton class
String from z is hello i am a string part of singleton class
```
![1](https://github.com/RamziJabali/Effective-Java-Practices/blob/master/pics-for%3Deffective-java/singleton-class-java.png)

Explanation:
1) In the Singleton class, when we first time call getInstance() method, it creates an object of the class with name single_instance and return it to the variable. 

2) Since single_instance is static, it is changed from null to some object. 

3) Next time, if we try to call getInstance() method, since single_instance is not null, it is returned to the variable, instead of instantiating the Singleton class again. This part is done by if condition.

### Implementing Singleton class with method name as that of class name
```java
   // static method to create instance of Singleton class 
    public static Singleton Singleton() 
    { 
        // To ensure only one instance is created 
        if (single_instance == null) 
        { 
            single_instance = new Singleton(); 
        } 
        return single_instance; 
    } 
```