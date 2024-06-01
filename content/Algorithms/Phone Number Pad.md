---
title: Phone Number Pad
date: 2024-01-01T14:02:48-07:00
tags:
  - kotlin
  - algorithm
  - medium
---
/ Given the phone dial pad attached below. Write a function that takes in a string of entered numbers
// and returns all possible corresponding text possibly meaning to enter as a list of strings.

// Example: Input “123”
// Output:  	[“ad”, “ae”, “af”, “bd”, “be”, “bf”, “cd”, “ce”, “cf”]

```kotlin
fun phoneNumberDial(numbers: String): List<String> {
    val phonePad = getPhonePadHashMap()
    var listOfCombinations = mutableListOf<String>()
    var lastCombinations = mutableListOf<String>()
    var currentPrimaryCombinationLetter = ""
    //base case length is 1
    if (numbers.length == 1) {
        return phonePad[numbers[0].toString()]!!
    }
    // length > 1
    for (character in phonePad[numbers[0].toString()]!!) { // check the character list of number 2 -> ["23"]
        currentPrimaryCombinationLetter += character // Add the first character in the list of number 2 -> A
        lastCombinations = phoneNumberDial(
            numbers.substring(
                1, numbers.length
            )
        ).toMutableList() // get the next number in the string
        for (character in lastCombinations) {
            listOfCombinations.add(currentPrimaryCombinationLetter + character)
        }
        lastCombinations.clear()
        currentPrimaryCombinationLetter = ""
    }
    return listOfCombinations
}

private fun getPhonePadHashMap(): HashMap<String, List<String>> = HashMap<String, List<String>>().apply {
    this.mapKeys {
        "0" to listOf("")
        "1" to listOf("")
        "2" to listOf("A", "B", "C")
        "3" to listOf("D", "E", "F")
        "4" to listOf("G", "H", "I")
        "5" to listOf("J", "K", "L")
        "6" to listOf("M", "N", "O")
        "7" to listOf("P", "Q", "R", "S")
        "8" to listOf("T", "U", "V")
        "9" to listOf("W", "Y", "X", "Z")
    }
}
```