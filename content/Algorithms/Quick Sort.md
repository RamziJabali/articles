---
title: Quick Sort
date: 2024-02-03
tags:
  - kotlin
  - algorithm
  - easy
---
```kotlin
// we want to choose a pivot randomly
fun quickSort(array: Array<Int>, startIndex: Int, endIndex: Int): Array<Int> {
	if (startIndex >= endIndex) return array
	// Calling partition to sort the array and give us back the pivot point
	val pivot: Int = partition(array, startIndex, endIndex)
	// we're splitting the array into 2 arrays one before the pivot and one after the pivot
	quickSort(array, startIndex, pivot - 1)
	quickSort(array, pivot + 1, endIndex)
	return array
}

private fun partition(array: Array<Int>, startIndex: Int, endIndex: Int): Int {
	var i = startIndex - 1 // setting up start index behind j
	val pivot = array[endIndex] // Pivot is the element at the last index
	for (j in startIndex until endIndex){ //going from start till end, pivot being the last index
		if (array[j] < array[pivot]){ // if element at j is < than pivot:
			i++ // increment i
			val temp = array[j] // save element at j in a temp
			array[j] = array[i] // set element at j to element at i
			array[i] = temp // set element at i with temp
		}
	}
	// once we are done we want to increment i inorder to swap the pivot to it's right place
	i++
	//swapping pivot with the i
	array[endIndex] = array[i]
	array[i] = pivot
	return i //returning the pivot point
}
```