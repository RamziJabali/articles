---
title: Builder Factory Method
date: 2020-03-21
tags:
  - java
---
### Consider a builder when faced with many constructor parameters

Static factories and constructors share a limitation: they do not scale well to large 
numbers of optional parameters.

Example:
```
    public boolean isUserEnteringRow;
    public boolean isUserEnteringColumn;
    public boolean startOfGame;
    public boolean didCurrentUserMiss;
    public boolean didCurrentUserHitOwnShip;
    public boolean didCurrentUserHitEnemyShip;
    public boolean isShipAlreadyHit;
```
Traditionally, programmers have used `Telescoping constructors` pattern, in which you provide a contructor
with only the required parameters, another with single optional parameters, a third with two optional parameters,
and so on, culminating in a constructor with all the optional paramters.

The telescoping constructor patterens work, but it is hard to write client code when they are many parameters, and harder still to read it.

### A Second alternative when facing many contructor parameters is the JavaBeans pattern

In which you call a parametersless contructor to create the object and call the setter and getter methods
to set each required parameter and each optional parameter of interest.

```java
//JavaBeans Pattern - allows inconsistency, mandates mutability

public class NutritionFacts{
  //parameters initialized to default values(if any)
  private int servingSize = -1;//Required: no default values (if any)
  private int servings = -1;//Required: no default values (if any)
  private int calories = 0;
  private int fat = 0;
  private int sodium = 0;
  private int carbohydrates = 0;
}

public NutritionFacts(){}

public void setServingSize(int val){
servingSize = val;
}
"                                 "
```
Unfortunately, the JavaBeans pattern has serious disadvantages of its own.
Because contruction is split across multiple calls, a JavaBean may be in an insconsistent state partway through its construction.

### Luckily, there is a third alternative that combines the safety of telescoping constructor pattern with the readability of the JavaBeans pattern.

It is a form of the `Builder` pattern.
Alternative way to construct complex objects.
Instead of making a desired object direclty, the client calls the constructor(or a static factory) with all the required 
parameters and gets a `builder object`. Then the client calls setter-like methods on the builder object to set each optional parameter of interest. Finally, the client calls a parameterless `build` method to generate the object, which is immutable. 

Builder pattern aims to “Separate the construction of a complex object from its representation so that the same construction process can create different representations.”

The builder is a static member class of the class it builds.

Example:

```java
public class NutritionFacts{
  //parameters initialized to default values(if any)
  private int servingSize;
  private int servings;
  private int calories;
  private int fat;
  private int sodium;
  private int carbohydrates;

  public static class Builder{
  //required parameters
  private final int servingSize;
  private final int servings;
  
  //optional parameters
  private int calories = 0;
  private int fat = 0;
  private int carbohydrates = 0;
  private int sodium = 0;
  
  public Builder(int servingSize, int servings){
  this.servingSize = servingSize;
  this.servings = servings;
  }
  
  public Builder calories(int val){
  calories = val;
  return this;
  }

  public Builder fat(int val){
  fat = val;
  return this;
  }
  
  public Builder sodium(int val){
  sodium = val;
  return this;
  }
  
  public NutritionFacts(){
  return new NutritionFacts(this);
  }
}
```

Note that `NutritionFacts` is immuatable, and that all parameter default values are in a single location.

The builder's setter methods return the builder itself so that invocations can be chained. 

Here is how the client code looks:

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).
    calories(100).sodium(35).carbohydrates(27).build();
```

that above created user object does not have any setter method, so it’s state can not be changed once it has been built. This provides the desired immutability.