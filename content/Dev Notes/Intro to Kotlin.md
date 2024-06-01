---
title: Intro to Kotlin
date: 2021-03-01
tags:
  - kotlin
  - article
---
  
<h2 align="center">Unit</h2>

`Unit` in Kotlin corresponds to the void in Java. Like void, `Unit` is the return type of any function that does not return any meaningful value, and it is optional to mention the `Unit` as the return type. But unlike void, `Unit` is a real class (Singleton) with only one instance.Aug 25, 2017


<h2 align="center">Print</h2>

In Kotlin the execution of the instructions begin with the `main` method.

```kotlin
fun main(){
   println("Hello world!")
}
```

Similar to Java Kotlin uses the keyword `print` and `println` to print is arguments to the output.

```kotlin
fun main(){ 
   print("Hello")
   println(" people!")
   print(2021)// Adds a line so that the text appears on the next line
}
```

<h2 align="center">Functions</h2>

Calling member functions uses the dot notation:

`Stream().read() // create instance of class Stream and call read()`

The function below has two parameters of type `Int` and has a return type of `Int` as well.

```kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}
```

A function body can be an expression. Its return type is inferred. Like so:

```kotlin
fun sum(a: Int, b: Int) = a + b

fun main() {
    println("sum of 19 and 23 is ${sum(19, 23)}")
}

//OutPut
//sum of 19 and 23 is 42
```

You can also make functions that do not return any value

```kotlin
fun printSum(a: Int, b: Int): Unit {
    println("sum of $a and $b is ${a + b}")
}
```

```kotlin
fun printMessage(message: String): Unit {                               // 1
    println(message)
}

fun printMessageWithPrefix(message: String, prefix: String = "Info") {  // 2
    println("[$prefix] $message")
}

fun sum(x: Int, y: Int): Int {                                          // 3
    return x + y
}

fun multiply(x: Int, y: Int) = x * y                                    // 4

fun main() {
    printMessage("What's up")                                               // 5                    
    printMessageWithPrefix("Hello", "Log")                              // 6
    printMessageWithPrefix("Hello")                                     // 7
    printMessageWithPrefix(prefix = "Log", message = "Hello")           // 8
    println(sum(1, 2))                                                  // 9
    println(multiply(2, 4))                                             // 10
}

/*
OutPut:
What's up
[Log] Hello
[Info] Hello
[Log] Hello
3
8
```
1) A simple function that takes a parameter of type `String` and returns `Unit` (i.e., no return value).
2) A function that takes a second optional parameter with default value `Info`. The return type is omitted, meaning that it's actually `Unit`.
3) A function that returns an integer.
4) A single-expression function that returns an integer (inferred).
5) Calls the first function with the argument `Hello`.
6) Calls the function with two parameters, passing values for both of them.
7) Calls the same function omitting the second one. The default value `Info` is used.
8) Calls the same function using named arguments and changing the order of the arguments.
9)Prints the result of the `sum` function call.
10) Prints the result of the `multiply` function call.


<h3 align="center">Infix Functions</h3>

What is infix function?

A function marked with the `infix` keyword can also be called using the infix notation (omitting the dot and the parentheses for the call). Infix functions must meet the following requirements.

1) Member functions and extensions with a single parameter can be turned into infix functions.
2) They must have single parameter
3) The parameter must not accept variable number of arguments and must have no default value

```kotlin
infix fun Int.shl(x: Int): Int { ... }

// calling the function using the infix notation
1 shl 2

// is the same as
1.shl(2)
```

```kotlin
fun main() {  
  infix fun Int.times(str: String) = str.repeat(this)        // 1
  println(2 times "Bye ")//OUTPUT: Bye Bye                   // 2

  val pair = "Ferrari" to "Katrina"                          // 3
  println(pair) // OUTPUT: (Ferrari, Katrina) 
  
  infix fun Int.plus(number: Int) = number + this
  println(1 plus 2) // 3
  
  infix fun String.onto(other: String) = Pair(this, other)   // 4
  val myPair = "McLaren" onto "Lucas"
  println(myPair)
  
  val sophia = Person("Sophia")
  val claudia = Person("Claudia")
  sophia likes claudia                                       // 5
}
  class Person(val name: String) {
  val likedPeople = mutableListOf<Person>()
  infix fun likes(other: Person) { likedPeople.add(other) }  // 6
}
```

1) Defines an infix extension function on `Int`.
2) Calls the `infix` function.
3) Creates a `Pair` by calling the infix function `to` from the standard library.
4) Here's your own implementation of  `to` creatively called `onto`.
5) Infix notation also works on members functions (methods).
6) The containing class becomes the first parameter.

<h3 align="center">Operator Functions</h3>

Certain functions can be "upgraded" to operators, allowing their calls with the corresponding operator symbol.

```kotlin
    operator fun Int.times(str: String) = str.repeat(this)       // 1
    println(2 * "Bye ")                                          // 2
    //Bye Bye 
    
    operator fun String.get(range: IntRange) = substring(range)  // 3
    val str = "Always forgive your enemies; nothing annoys them so much."
    println(str[0..14])//says display from element 0 -> 14
    //Always forgive // it's the number of characters to display 
```
1) This takes the infix function from above one step further using the `operator` modifier.
2) The operator symbol for `times()` is `*` so that you can call the function using `2 * "Bye"`.
3) An operator function allows easy range access on strings.
4) The `get()` operator enables bracket-access syntax.


<h3 align ="center">Functions with vararg Parameters</h3>

`Vararg`s allow you to pass any number of arguments by separating them with commas.

```kotlin
fun printAll(vararg messages: String) {                            // 1
    for (m in messages) println(m)
}
printAll("Hello", "Hallo", "Salut", "Hola", "你好")                 // 2

fun printAllWithPrefix(vararg messages: String, prefix: String) {  // 3
    for (m in messages) println(prefix + m)
}
printAllWithPrefix(
    "Hello", "Hallo", "Salut", "Hola", "你好",
    prefix = "Greeting: "                                          // 4
)

fun log(vararg entries: String) {
    printAll(*entries)                                             // 5
}
```
1) The vararg modifier turns a parameter into a vararg.
2) This allows calling printAll with any number of string arguments.
3) Thanks to named parameters, you can even add another parameter of the same type after the vararg. This wouldn't be allowed in Java because there's no way to pass a value.
4) Using named parameters, you can set a value to prefix separately from the vararg.
5) At runtime, a vararg is just an array. To pass it along into a vararg parameter, 6) use the special spread operator * that lets you pass in *entries (a vararg of String) instead of entries (an Array<String>).

```kotlin
fun printAll(vararg messages: String) {                            // 1
    for (m in messages) println(m)
}
printAll("Hello", "Hallo", "Salut", "Hola", "你好")                 // 2

fun printAllWithPrefix(vararg messages: String, prefix: String) {  // 3
    for (m in messages) println(prefix + m)
}
printAllWithPrefix(
    "Hello", "Hallo", "Salut", "Hola", "你好",
    prefix = "Greeting: "                                          // 4
)

fun log(vararg entries: String) {
    printAll(*entries)                                             // 5
}
```


<h2 align="center">Variables</h2>
 

 
```kotlin
var a: String = "initial"  // 1
println(a)
val b: Int = 1             // 2
val c = 3                  // 3

``` 
1) Declares a mutable variable and initializes it.
2) Declares an immutable variable and initializes it.
3) Declares an immutable variable and initializes it without specifying the type. The  compiler infers the type Int.
 
 

Read-only local variables are defined using the keyword `val`. They can be assigned a value only once. 
Which is like Java's `final` keyword.

```kotlin
val a: Int = 1  // immediate assignment
val b = 2   // `Int` type is inferred
val c: Int  // Type required when no initializer is provided
c = 3       // deferred assignment
```
Variables that can be reassigned use the `var` keyword.

```kotlin
var x = 5 // `Int` type is inferred
x += 1
```

You can declare variables globaly as well.

```kotlin
val PI = 3.14
var x = 0

fun incrementX() { 
    x += 1 
}
```

<h2 align="center">Creating Classes and Instances</h2>

The keyword `class` is used to define a class like Java.

`class House` 

The properties of a class can be listed within it's declaration or body.

```kotlin
class Rectangle(var height: Double, var length: Double) {
    var perimeter = (height + length) * 2
}
```

The default constructor with parameters listed in the class declaration is available automatically.

```kotlin
class Rectangle(var height: Double, var length: Double) {
    var perimeter = (height + length) * 2 
}

fun main() {
    val rectangle = Rectangle(5.0, 2.0)
    println("The perimeter is ${rectangle.perimeter}")
}

//The perimeter is 14.0
```

Inheritance between classes is declared by a colon (`:`). 
Classes are final by default; to make a class inheritable, mark it as `open`.

```kotlin
open class Shape

class Rectangle(var height: Double, var length: Double): Shape() {
    var perimeter = (height + length) * 2
}
```
 
 ```kotlin
 class Customer                                  // 1

class Contact(val id: Int, var email: String)   // 2

fun main() {

    val customer = Customer()                   // 3
    
    val contact = Contact(1, "mary@gmail.com")  // 4

    println(contact.id)                         // 5
    contact.email = "jane@gmail.com"            // 6
}
```
1) Declares a class named Customer without any properties or user-defined constructors. A non-parameterized default constructor is created by Kotlin automatically.
2) Declares a class with two properties: immutable id and mutable email, and a constructor with two parameters id and email.
3) Creates an instance of the class Customer via the default constructor. Note that there is no new keyword in Kotlin.
4) Creates an instance of the class Contact using the constructor with two arguments.
5) Accesses the property id.
6) Updates the value of the property email.
 
<h2 align="center">String Templates</h2>

```kotlin
var a = 1
// simple name in template:
val s1 = "a is $a" 

a = 2
// arbitrary expression in template:
val s2 = "${s1.replace("is", "was")}, but now is $a"

//a was 1, but now is 2
```
<h2 align="center">Conditional Expressions</h2>

<h3 align="center">If statement</h3>

```kotlin
fun maxOf(a: Int, b: Int): Int {
    if (a > b) {
        return a
    } else {
        return b
    }
}
```

In Kotlin, `if` can also be used as an expression.

```kotlin
fun maxOf(a: Int, b: Int) = if (a > b) a else b

// As expression
val max = if (a > b) a else b
```

Branches of if branches can be blocks. In this case, the last expression is the value of a block:

```kotlin
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```

If you're using `if` as an expression, for example, for returning its value or assigning it to a variable, the else branch is mandatory.


<h3 align="center">When expression</h3>

when defines a conditional expression with multiple branches.
It is similar to the switch statement in C-like languages. 
Its simple form looks like this.
```kotlin
fun describe(obj: Any): String =
    when (obj) {
        1          -> "One"
        "Hello"    -> "Greeting"
        is Long    -> "Long"
        !is String -> "Not a string"
        else       -> "Unknown"
    }
```

```kotlin
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> { // Note the block
        print("x is neither 1 nor 2")
    }
}
```

`when`matches its argument against all branches sequentially until some branch condition is satisfied.

`when` can be used either as an expression or as a statement. If it is used as an expression, the value of the first matching branch becomes the value of the overall expression. If it is used as a statement, the values of individual branches are ignored. 

The `else` branch is evaluated if none of the other branch conditions are satisfied. 
If `when` is used as an expression, the `else` branch is mandatory, unless the compiler can prove that all possible cases are covered with branch conditions, for example, with `enum` class entries and sealed class subtypes).

The way to define a common behavior for multiple cases is by using a comma `,` that allows you to combine the cases in a single line.

```kotlin
when(x){
 0, 1 -> print ("x == 0 or x ==1")
 else -> print ("otherwise")
}
```

You can use arbitrary expressions (not only constants) as branch conditions

```
when (x) {
    parseInt(s) -> print("s encodes x")
    else -> print("s does not encode x")
}
```

You can also check a value for being in or !in a range or a collection:

```
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```

Another option is checking that a value is or !is of a particular type. 

```
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```

`when` can also be used as a replacement for an `if`-`else` `if` chain. If no argument is supplied, the branch conditions are simply boolean expressions, and a branch is executed when its condition is true:

```
when {
    x.isOdd() -> print("x is odd")
    y.isEven() -> print("y is even")
    else -> print("x+y is odd")
}
```


```
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```

<h3 align="center">For Loop</h3>

The syntax for the For Loop

```
for(item in colection) print(item)
```

The body of a `for` can be a block

```
for (item: Int in ints){
 //block of code
}
```

As mentioned before, for iterates through anything that provides an iterator.

This means that it:

has a member or an extension function `iterator()` that returns `Iterator<>`:

has a member or an extension function `next()`

has a member or an extension function `hasNext()` that returns `Boolean`.

All of these three functions need to be marked as `operator`.

To iterate over a range of numbers, use a range expression:

```
for (i in 1..3) {
        println(i)
    }
    for (i in 6 downTo 0 step 2) {
        println(i)
    }
```

```
val items = listOf("apple", "banana", "kiwifruit")
    for (item in items) {
        println(item)
     }
//or

val items = listOf("apple", "banana", "kiwifruit")
for (index in items.indices) {
    println("item at $index is ${items[index]}")
}
```

A `for` loop over a range or an array is compiled to an index-based loop that does not create an iterator object.

If you want to iterate through an array or a list with an index, you can do it this way:

```
for (i in array.indices) {
    println(array[i])
}
```
Alternatively, you can use the `withIndex` library function:

```
fun main() {
    val array = arrayOf("a", "b", "c")
    for ((index, value) in array.withIndex()) {
        println("the element at $index is $value")
    }
}
```

 How to iterate through a 2D array in Kotlin 
 
 ```
   for (row in Model.playerGrid) {
        for(column in row){
            print("[${column.player}]")
        }
        println()
    }

 output:
[ ][ ][ ]
[ ][ ][ ]
[ ][ ][ ]
 ```


<h3 align="center">While Loop and do Do While</h3>

while and do-while loops execute their body continuously while their condition is satisfied. The difference between them is the condition checking time:

while checks the condition and, if it's satisfied, executes the body and then returns to the condition check.

do-while executes the body and then checks the condition. If it's satisfied, the loop repeats. So, the body of do-while executes at least once regardless of the condition.

```
val items = listOf("apple", "banana", "kiwifruit")
var index = 0
while (index < items.size) {
    println("item at $index is ${items[index]}")
    index++
}
```

```
while (x > 0) {
    x--
}

do {
    val y = retrieveData()
} while (y != null) // y is visible here!
```
<h3 align="center">Iterators</h3> 
 
 Made for traversing a collection of elements.
 
iterators – objects that provide access to the elements sequentially without exposing the underlying structure of the collection.
 
You can define your own iterators in your classes by implementing the `iterator` operator in them.
 
Once the iterator passes through the last element, it can no longer be used for retrieving elements; neither can it be reset to any previous position. To iterate through the collection again, create a new iterator.

```
val numbers = listOf("one", "two", "three", "four")
val numbersIterator = numbers.iterator()
while (numbersIterator.hasNext()) {
    println(numbersIterator.next())
}
```

<h2 align="center">Ranges</h2>

Check if a number is within a range using the `in` operator.

```
val x = 10
val y = 9
if (x in 1..y+1) {
    println("fits in range")
}
```
Check if a number is out of range.

you can do NOT in as a means to check if something is out of range:

`!in`

```
val list = listOf("a", "b", "c")

if (-1 !in 0..list.lastIndex) {
    println("-1 is out of range")
}
if (list.size !in list.indices) {
    println("list size is out of valid list indices range, too")
}
```

Iterate over a range.

```
for (x in 1..5) {
    print(x)
}
```

Or over a progression.

```
for (x in 1..10 step 2) {
    print(x)
}

println()

for (x in 9 downTo 0 step 3) {
    print(x)
}
```

<h2 align="center">Collections</h2>

How to iterate over a collection

```
val items = listOf("apple", "banana", "kiwifruit")

for(item in items){
println(item)
}
```

How to check if a collection contains an object using the `in` operator

```
when {
    "orange" in items -> println("juicy")
    "apple" in items -> println("apple is also juicy")
}
```

Using lambda expressions to filter and map collections:

```
fun main() {
    val fruits = listOf("banana", "avocado", "apple", "kiwifruit")
    fruits
        .filter { it.startsWith("a") }
        .sortedBy { it }
        .map { it.uppercase() }
        .forEach { println(it) }
}
```

<h2 align="center">Nullable values and null checks</h2>

A reference must be explicitly marked as nullable when `null` value is possible. Nullable type names have `?` at the end.

Return `null` if `str` does not hold an integer:

```
fun parseInt(Str: String): Int?{

}
```

Use a function returning nullable value:

```
fun printProduct(arg1: String, arg2: String) {
    val x = parseInt(arg1)
    val y = parseInt(arg2)

    // Using `x * y` yields error because they may hold nulls.
    if (x != null && y != null) {
        // x and y are automatically cast to non-nullable after null check
        println(x * y)
    }
    else {
        println("'$arg1' or '$arg2' is not a number")
    }    
}
```

or

```
if (x == null) {
    println("Wrong number format in arg1: '$arg1'")
    return
}
if (y == null) {
    println("Wrong number format in arg2: '$arg2'")
    return
}

// x and y are automatically cast to non-nullable after null check
println(x * y)
```

<h2 align="center">Type checks and automatic casts</h2>

The `is` operator checks if an expression is an instance of a type. If an immutable local variable or property is checked for a specific type, there's no need to cast it explicitly:

```
fun getStringLength(obj: Any): Int? {
    if (obj is String) {
        // `obj` is automatically cast to `String` in this branch
        return obj.length
    }

    // `obj` is still of type `Any` outside of the type-checked branch
    return null
}
```

or 

```
fun getStringLength(obj: Any): Int? {
    if (obj !is String) return null

    // `obj` is automatically cast to `String` in this branch
    return obj.length
}
```

or 

```
fun getStringLength(obj: Any): Int? {
    // `obj` is automatically cast to `String` on the right-hand side of `&&`
    if (obj is String && obj.length > 0) {
        return obj.length
    }

    return null
}
```

<h2 align="center">Basic types in Kotlin</h2>

<h3 align="center">Arrays</h3>

Different ways to define and initialize an array in Kotlin

```
    var x: IntArray = intArrayOf(1, 2, 3) // [1, 2, 3]
    var y = IntArray(5) //has a size of 5, but all elements are 0
    
    //[42, 42, 42, 42, 42]
    val arr = IntArray(5) { 42 } // has a size of 5 with all elements of value 42

    // e.g. initialise the values in the array using a lambda
    // Array of int of size 5 with values [0, 1, 2, 3, 4] (values initialised to their index value)
    var arr2 = IntArray(5) { it * 2 }
```
https://kotlinlang.org/docs/basic-types.html#numbers


<h2 align="center">Null in Kotlin</h2>
 
Variables cannot be assigned `null` unless they are declared with the key symbol `?` at the end of the type.

Cannot be null

`var cannotBeNull: Int = null`

Will produce a compiler error
 
`var canBeNull: Int? = 2`
 
We are declaring a nullable Int
 
`canBeNull = null`

Sets `null` value to the nullable variable
 
If a function has a parameter that is not nullable and you pass it a nullable variable it will produce a compiler error.
 
 
<h2 align="center">Generics in Kotlin</h2>
 
<h3 align="center">Generic classes</h3>

```
class MutableStack<E>(vararg items: E) {              // 1

  private val elements = items.toMutableList()

  fun push(element: E) = elements.add(element)        // 2

  fun peek(): E = elements.last()                     // 3

  fun pop(): E = elements.removeAt(elements.size - 1)

  fun isEmpty() = elements.isEmpty()

  fun size() = elements.size

  override fun toString() = "MutableStack(${elements.joinToString()})"
} 
```
1) Defines a generic class `MutableStack<E>` where `E is called the generic type parameter. At use-site, it is assigned to a specific type such as `Int` by declaring a `MutableStack<Int>`.
2) Inside the generic class, `E` can be used as a parameter like any other type
3) You can also use `E` as a return type
 
 <h3 align="center">Generic Functions</h3>
 
You can also generify functions if their logic is independent of a specific type. For instance, you can write a utility function to create mutable stacks:

```
fun <E> mutableStackOf(vararg elements: E) = MutableStack(*elements)

fun main() {
  val stack = mutableStackOf(0.62, 3.14, 2.7)
  println(stack)
}
```
 
Note that the compiler can infer the generic type from the parameters of `mutableStackOf` so that you don't have to write `mutableStackOf<Double>(...)`.

<h3 align="center">Inheritance in Kotlin</h3>
  
open class Dog {                // 1
    open fun sayHello() {       // 2
        println("wow wow!")
    }
}

class Yorkshire : Dog() {       // 3
    override fun sayHello() {   // 4
        println("wif wif!")
    }
}

fun main() {
    val dog: Dog = Yorkshire()
    dog.sayHello()
}
 
1) Kotlin classes are final by default. If you want to allow the class inheritance, mark the class with the `open` modifier.
2) Kotlin methods are also final by default. As with the classes, the `open` modifier allows overriding them.
3) A class inherits a superclass when you specify the `: SuperclassName()` after its name. The empty parentheses `()` indicate an invocation of the superclass default constructor.
4) Overriding methods or attributes requires the `override` modifier.
 
<h4 align="center">Inheritance with Parameterized Constructor</h4>
 
```
open class Tiger(val origin: String) {
    fun sayHello() {
        println("A tiger from $origin says: grrhhh!")
    }
}

class SiberianTiger : Tiger("Siberia")        // 1

fun main() {
    val tiger: Tiger = SiberianTiger()
    tiger.sayHello()
}
```
1) If you want to use a parameterized constructor of the superclass when creating a subclass, provide the constructor arguments in the subclass declaration.

<h4 align="center">Passing Constructor Arguments to Superclass</h4>
 
```
open class Lion(val name: String, val origin: String) {
    fun sayHello() {
        println("$name, the lion from $origin says: graoh!")
    }
}

class Asiatic(name: String) : Lion(name = name, origin = "India") // 1

fun main() {
    val lion: Lion = Asiatic("Rufo")                              // 2
    lion.sayHello()
}
```
1) `name` in the `Asiatic` declaration is neither a var nor val: it's a constructor argument, whose value is passed to the `name` property of the superclass `Lion`.
2) Creates an `Asiatic` instance with the name `Rufo`. The call invokes the `Lion` constructor with arguments `Rufo` and `India`.