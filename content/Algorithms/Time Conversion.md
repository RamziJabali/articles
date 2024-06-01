---
title: Time Conversion
date: 2024-01-01T14:02:48-07:00
tags:
  - kotlin
  - algorithm
  - easy
---
Given a time in 12-hour AM/PM format, convert it to military (24-hour) time.
Note: - 12:00:00AM on a 12-hour clock is 00:00:00 on a 24-hour clock.
- 12:00:00PM on a 12-hour clock is 12:00:00 on a 24-hour clock.
Example
S = '12:01:00PM'
    Return '12:01:00'

S = '12:01:00AM'
    Return '00:01:00'

Function Description

Complete the timeConversion function in the editor below. It should return a new string representing the input time in 24 hour format.

timeConversion has the following parameter(s):

string s: a time in 12 hour format
Returns

string: the time in 24 hour format
Input Format

A single string 8 that represents a time in 12-hour clock format (i.e.: hh:mm:ssAM or hh:mm:ssPM).
Constraints:

All input times are valid

Sample Input
07:05:45PM
Sample Output
19:05:45
```kotlin
fun timeConversion(s: String): String {
    val hours: String = s.subSequence(0, 2).toString()
    val time: String = ""
    if (s.takeLast(2) == "AM") {
        if (hours == "12"){
            return "00"+s.subSequence(2, s.length-2)
        }       
        return hours+ s.subSequence(2, s.length-2)

    } else {
         if (hours == "12") {
            return "12" + s.subSequence(2, s.length-2)
        }
        return (hours.toInt() + 12).toString() + s.subSequence(2, s.length -2)
    }
}