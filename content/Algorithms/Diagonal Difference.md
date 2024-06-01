---
title: Diagonal Difference
date: 2024-01-01T14:02:48-07:00
tags:
  - kotlin
  - algorithm
  - easy
---
Given a square matrix, calculate the absolute difference between the sums of its diagonals.

For example, the square matrix arr is shown below:

1 2 3
4 5 6
9 8 9  
The right-to-left diagonal = 3+9+5 = 17. The left-to-right diagonal = 1+9+5 = 15. Their absolute difference is |15 - 17| = 2.

Function description

Complete the  function in the editor below.

diagonalDifference takes the following parameter:

int arr[n][m]: an array of integers
Return

int: the absolute diagonal difference
Input Format

The first line contains a single integer, n, the number of rows and columns in the square matrix arr.
Each of the next arr[i] lines describes a row, and consists of n space-separated integers arr[i][j].

```kotlin
fun diagonalDifference(arr: Array<Array<Int>>): Int {
    val leftDiagonal = mutableListOf<Int>()
    val rightDiagonal = mutableListOf<Int>()
    val matrixSize = arr.size
    for (i in arr.indices) {
        leftDiagonal.add(arr[i][i])
        rightDiagonal.add(arr[i][matrixSize - i - 1])
    }
    val sumOfLeft = leftDiagonal.foldRight(0) { element, acc ->
        acc + element
    }

    val sumOfRight = rightDiagonal.foldRight(0){ element, acc ->
        acc + element
    }

    return Math.abs(sumOfLeft - sumOfRight)
}
```
