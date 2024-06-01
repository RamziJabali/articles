---
title: Binary Search Tree
date: 2024-02-02
tags:
  - kotlin
---
```kotlin
class BinarySearchTree(
    private var value: Int,
    private var leftNode: BinarySearchTree? = null,
    private var rightNode: BinarySearchTree? = null,
) {
    fun addNumber(number: Int): Boolean {
        if (number > this.value) {
            if (this.rightNode == null) {
                this.rightNode = BinarySearchTree(value = number)
            } else {
                this.rightNode?.addNumber(number)
            }
        } else {
            if (this.leftNode == null) {
                this.leftNode = BinarySearchTree(value = number)
            } else {
                this.leftNode?.addNumber(number)
            }
        }
        return true
    }

    fun getSmallestNumber(): Int  = when {
        this.leftNode == null -> this.value
        else -> this.leftNode!!.getSmallestNumber()
    }

    fun getLargestNumber(): Int {
        return -1
    }

    fun printTree() {

    }

    fun convertArrayToTree(array: Array<Int>): Boolean {
        return true
    }

}
```