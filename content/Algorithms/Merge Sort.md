---
title: Merge Sort
date: 2024-02-03
tags:
  - kotlin
  - algorithm
  - easy
---
```kotlin
fun mergeSort(array: Array<Int>) {
	//base case
	if (array.size <= 1) return
	// we first want to split the array into 2
	val leftArray = array.sliceArray(0 until array.size / 2)
	val rightArray = array.sliceArray(array.size / 2  until array.size)
	mergeSort(leftArray)
	mergeSort(rightArray)
	merge(leftArray, rightArray, array)
}

private fun merge(leftArray: Array<Int>, rightArray: Array<Int>, completeArray: Array<Int>) {
	var leftIndex = 0
	var rightIndex = 0
	var arrayIndex = 0
	while (leftIndex < leftArray.size && rightIndex < rightArray.size) {
		if (leftArray[leftIndex] < rightArray[rightIndex]) {
			completeArray[arrayIndex] = leftArray[leftIndex]
			leftIndex++
			arrayIndex++
		} else {
			completeArray[arrayIndex] = rightArray[rightIndex]
			rightIndex++
			arrayIndex++
		}
	}
	//making sure that both arrays are taken care of even when they are not equal in size
	while(leftIndex < leftArray.size){
		completeArray[arrayIndex] = leftArray[leftIndex]
		leftIndex++
		arrayIndex++
	}
	while(rightIndex < rightArray.size){
		completeArray[arrayIndex] = rightArray[rightIndex]
		rightIndex++
		arrayIndex++
	}
}
```