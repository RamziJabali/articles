---
title: 8 Counting Sort 1
date: 2024-01-01T14:02:48-07:00
tags:
---
![Screenshot 2024-05-16 at 4 43 48 PM](https://github.com/RamziJabali/algorithm-problems/assets/18749441/85d7328c-9d39-44e1-8cf0-744fd7a04f7a)
![Screenshot 2024-05-16 at 4 44 27 PM](https://github.com/RamziJabali/algorithm-problems/assets/18749441/d5f2bd81-7ce1-492e-a164-835814353462)![Screenshot 2024-05-16 at 4 43 48 PM](https://github.com/RamziJabali/algorithm-problems/assets/18749441/85d7328c-9d39-44e1-8cf0-744fd7a04f7a)
![Screenshot 2024-05-16 at 4 44 27 PM](https://github.com/RamziJabali/algorithm-problems/assets/18749441/d5f2bd81-7ce1-492e-a164-835814353462)

Sample Input

100
63 25 73 1 98 73 56 84 86 57 16 83 8 25 81 56 9 53 98 67 99 12 83 89 80 91 39 86 76 85 74 39 25 90 59 10 94 32 44 3 89 30 27 79 46 96 27 32 18 21 92 69 81 40 40 34 68 78 24 87 42 69 23 41 78 22 6 90 99 89 50 30 20 1 43 3 70 95 33 46 44 9 69 48 33 60 65 16 82 67 61 32 21 79 75 75 13 87 70 33  
Sample Output

0 2 0 2 0 0 1 0 1 2 1 0 1 1 0 0 2 0 1 0 1 2 1 1 1 3 0 2 0 0 2 0 3 3 1 0 0 0 0 2 2 1 1 1 2 0 2 0 1 0 1 0 0 1 0 0 2 1 0 1 1 1 0 1 0 1 0 2 1 3 2 0 0 2 1 2 1 0 2 2 1 2 1 2 1 1 2 2 0 3 2 1 1 0 1 1 1 0 2 2 
Explanation

Each of the resulting values result[i] represents the number of times  appeared in arr.

```kotlin
fun countingSort(arr: Array<Int>): Array<Int> {
    val countingArray = Array(100) {0}
    for(index in arr){
        countingArray[index]+= 1
    }
    return countingArray
}
```

### For dimension 0 < n < infinite

```kotlin
fun countingSort(arr: Array<Int>): Array<Int> {
    val max = arr.max()+1
    println("max number = $max")
    val countingArray = Array(max) {0}
    println("array size = ${countingArray.size}")
    for(index in arr){
        countingArray[index]+= 1
    }
    return countingArray
}
```