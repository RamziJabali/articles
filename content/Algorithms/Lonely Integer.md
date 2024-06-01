---
title: Lonely Integer
date: 2024-01-01T14:02:48-07:00
tags:
  - kotlin
  - easy
  - algorithm
---
Given an array of integers, where all elements but one occur twice, find the unique element.

Example
a=(1,2,3,4,2,1)

The unique element is 4.

Function Description

Complete the lonelyinteger function in the editor below.

lonelyinteger has the following parameter(s):

int afny: an array of integers

Returns

* int the element that occurs only once

Input Format

The first line contains a single integer, n, the number of integers in the array.

The second line contains n space-separated integers that describe the values in a.
Constraints

* 1 <= n < 100

* It is guaranteed that m is an odd number and that there is one unique element.

* 0 <= a[i] <= 100, where O <= i < n.

```kotlin
fun lonelyinteger(a: Array<Int>): Int {
    val maxSize = 100
    val repetitionsArray = Array<Int>(maxSize){0}
    for (index in a.indices) {
        val newIndex = a[index] % maxSize
        repetitionsArray[newIndex] += 1
    }
    for (index in repetitionsArray.indices) {
        if(repetitionsArray[index] == 1){
            return index
        }
    }
    return 0
}
```

## Solution #2
```kotlin
fun lonelyinteger(a: Array<Int>): Int {
    val numberMap = HashMap<Int, Int>()
    for (number in a){
        val currentQuantity = numberMap[number] ?: 0
        numberMap[number] = currentQuantity + 1
    }
    for (number in numberMap) {
        if(number.value == 1){
            return number.key
        }
    }
    return -1
```