---
title: Tower Breakers
date: 2024-01-01T14:02:48-07:00
tags:
---
![Screenshot 2024-05-23 at 11 56 58 AM](https://github.com/RamziJabali/algorithm-problems/assets/18749441/7866505d-8cd0-473a-bbb1-5a19adf78e28)
![Screenshot 2024-05-23 at 11 57 20 AM](https://github.com/RamziJabali/algorithm-problems/assets/18749441/525bfcbf-a36b-4c67-ac59-102bd1fd6f0a)

```kotlin
fun towerBreakers(n: Int, m: Int): Int {
    // special case
    if (m == 1) {
        return 2
    }
    return when (n % 2) {
        // even case
        0 -> {
            2
        }
        // odd case
        else -> {
            1
        }
    }
}
```