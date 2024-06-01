---
title: Anagram
date: 2024-01-01T14:02:48-07:00
tags:
  - kotlin
  - medium
---
```kotlin
// 2 strings, anagram words you can rearrange that makes it into another word
fun anagram(word1: String, word2: String): Boolean {
    if (word1.length != word2.length) return false
    val letterHashMap = HashMap<Char, Int>()
    // O(1)
    for (character in 'a'..'z') {
        letterHashMap[character] = 0
    }
    // O(n)
    for (index in word1.indices) {
        letterHashMap[word1[index].lowercaseChar()] = letterHashMap[word1[index].lowercaseChar()]!! + 1
    }
    // O(n)
    for (index in word2.indices) {
        letterHashMap[word2[index].lowercaseChar()] = letterHashMap[word2[index].lowercaseChar()]!! - 1
    }
    // O(1)
    for (character in 'a'..'z') {
        if (letterHashMap[character] != 0) {
            return false
        }
    }
    return true
}
```