---
title: Coroutines Guide
date: 2024-06-01
tags:
  - coroutines
  - article
  - android
  - incomplete
---
# Introduction

### What is asynchronous programming?
- When you call an asynchronous function that does not return the result immediately, but at some point in the future will.

### Why not do it synchronously?
- Most Android code runs on the main(UI) thread.
	- If the main thread waits for an API request to finish, the UI is paused/frozen and will not respond.
	- If we call an asynchronous function on the main thread it goes to the background thread where it only goes back to the main thread when it finishes 

## Different Implementations

### Callback implementation
- Doesn't use a framework like RxJava or Coroutines
- End up with big amount of callbacks
- Have to manually cancel callbacks
- Callbacks are done on the main(UI) thread
- Hard to read, understand, and build on
- Very error-prone

Example:
```kotlin
 private var getAndroidVersionsCall: Call<List<AndroidVersion>>? = null
    private var getAndroidFeaturesCall: Call<VersionFeatures>? = null

    fun perform2SequentialNetworkRequest() {

        uiState.value = UiState.Loading

        getAndroidVersionsCall = mockApi.getRecentAndroidVersions()
        getAndroidVersionsCall!!.enqueue(object : Callback<List<AndroidVersion>> {
            override fun onFailure(call: Call<List<AndroidVersion>>, t: Throwable) {
                uiState.value = UiState.Error("Network Request failed")
            }

            override fun onResponse(
                call: Call<List<AndroidVersion>>,
                response: Response<List<AndroidVersion>>
            ) {
                if (response.isSuccessful) {
                    val mostRecentVersion = response.body()!!.last()
                    getAndroidFeaturesCall =
	                mockApi.getAndroidVersionFeatures(mostRecentVersion.apiLevel)
                    getAndroidFeaturesCall!!.enqueue(object : Callback<VersionFeatures> {
                        override fun onFailure(call: Call<VersionFeatures>, t: Throwable) {
                            uiState.value = UiState.Error("Network Request failed")
                        }

                        override fun onResponse(
                            call: Call<VersionFeatures>,
                            response: Response<VersionFeatures>
                        ) {
                            if (response.isSuccessful) {
                                val featuresOfMostRecentVersion = response.body()!!
                                uiState.value = UiState.Success(featuresOfMostRecentVersion)
                            } else {
                                uiState.value = UiState.Error("Network Request failed")
                            }
                        }
                    })
                } else {
                    uiState.value = UiState.Error("Network Request failed")
                }
            }
        })
    }

    override fun onCleared() {
        super.onCleared()

        getAndroidVersionsCall?.cancel()
        getAndroidFeaturesCall?.cancel()
    }
```


### RxJava Implementation
- Very flexible to use
- Slightly challenging to read and understand
- Uses reactive programming principles
- Every line requires you to understand what the operator is and does
- You have to manage lifecycle 

Example
```kotlin
 private val disposables = CompositeDisposable()

    fun perform2SequentialNetworkRequest() {
        uiState.value = UiState.Loading

        mockApi.getRecentAndroidVersions()
            .flatMap { androidVersions ->
                val recentVersion = androidVersions.last()
                mockApi.getAndroidVersionFeatures(recentVersion.apiLevel)
            }
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribeBy(
                onSuccess = { featureVersions ->
                    uiState.value = UiState.Success(featureVersions)
                },
                onError = {
                    uiState.value = UiState.Error("Network Request failed.")
                }
            )
            .addTo(disposables)
    }

    override fun onCleared() {
        super.onCleared()
        disposables.clear()
    }
```

### Coroutines implementation
- Easy to read
- Very Flexible
- Imperative programming style
- Manages lifecycle for you, due to strict coroutine scope restrictions 
- Not very error Prone
Example:
```kotlin
  fun perform2SequentialNetworkRequest() {
        uiState.value = UiState.Loading
        viewModelScope.launch {
            try {
                val recentVersions = mockApi.getRecentAndroidVersions()
                val mostRecentVersion = recentVersions.last()

                val featuresOfMostRecentVersion =
                    mockApi.getAndroidVersionFeatures(mostRecentVersion.apiLevel)

                uiState.value = UiState.Success(featuresOfMostRecentVersion)
            } catch (exception: Exception) {
                uiState.value = UiState.Error("Network Request failed")
            }
        }
    }
```

#  Coroutine Fundamentals

## Routines vs Coroutines

### What's the difference
-  Routines
	- A sequence of program instructions, that performs a specific task, packaged as a unit
- Coroutines
	- Co-Operative Routines
	- They can pass the control flow between each other
## Suspend functions

- `suspend`
	- Function that is doing some form of longer running operation
	- The special kinds of functions can be suspended by Kotlin Coroutines machinery and resumed at a later time without blocking the underlying thread
	- Suspend functions can only be called from other suspend functions or from coroutines
	- Suspend functions can call regular functions
## Coroutines and Threads

### Coroutines are more efficient than Threads 
- Using threads instead of coroutines is possible, but expensive
- Switching between threads is expensive, but coroutines does it all on the same thread
### Coroutines Example:

```kotlin
fun main() = runBlocking {
	repeat(1_000_000) { // 1 million repetitions  
		delay(5_000)
		print(".")
	}
}
```

The result will be 1 million "." being printed
### Thread Example:

```kotlin
fun main() = {
	repeat(1_000_000) { // 1 million repetitions  
		Thread.sleep(5_000)
		print(".")
	}
}
```

This will give you a `java.lang.outOfMemory` exception, meaning we ran out of memory.
Which means it did not finish the 1 million repetitions, because it is memory expensive to keep making threads.

Coroutines are efficient and lightweight threads!

### Coroutines vs Threads
- `Thread.sleep` commands are blocking calls
	- While they are sleeping they cannot do other tasks
- `suspend` does not block the main thread so the main thread can work on other tasks while our functions are suspended
