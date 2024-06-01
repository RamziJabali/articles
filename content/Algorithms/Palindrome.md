---
title: Palindrome
date: 2024-01-01T14:02:48-07:00
tags:
---
```kotlin

fun palindromeChecker(word: String): Boolean {
    // case 1 empty string return false:
    if (word.isEmpty()) return false
    //  case 2 checking if the word is a palindrome
    var bIndex = word.length - 1
    var fIndex = 0
    while (fIndex != word.length - 1) {
        if (word[fIndex] != word[bIndex]) {
            return false
        }
        if (word.length % 2 == 0 && fIndex == word.length / 2) {
            return true
        } else if (fIndex == bIndex) {
            return true
        }
        fIndex++
        bIndex--
    }
    return false

}

fun palindromeChecker2(word: String): Boolean {
    // case 1 empty string return false:
    if (word.isEmpty()) return false
    //  case 2 reverse the string and compare the 2
    var reversedString = ""
    for (index in word.length-1 downTo 0){
        reversedString += word[index]
    }
    println(reversedString)
    return word == reversedString
}
```
