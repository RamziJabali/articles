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
## Blocking vs Suspending 

Blocking Example:
```kotlin
fun main() {
	print("main start")
	threadRoutine(1, 500)
	threadRoutine(2, 800)
	thread.sleep(1000)
	println("main end")
}

fun threadRoutine(number: Int, delay: Long){
	thread {
		println("routine $number start work")
		Thread.sleep(delay)
		println("routine $number finished work")
	}
}
```

Suspend Example:
```kotlin
fun main() = runBlocking {
	print("main start")
	joinAll(
		async { coroutineWithThreadInfo(1, 500) },
		async { coroutineWithThreadInfo(2, 300) },
		async { /*other work! */ }
	
	threadRoutine(1, 500)
	threadRoutine(2, 800)
	thread.sleep(1000)
	println("main end")
}

suspend fun coroutineWithThreadInfo(number: Int, delay: Long){
	println("coroutine $number starts work on ${Thread.currentThread().name}")
	delay(delay)
	println("coroutine $number finished work on ${Thread.currentThread().name}"
}
```

![Suspend](https://raw.githubusercontent.com/RamziJabali/articles/v4/images/suspend_example.png)

![Blocking](https://raw.githubusercontent.com/RamziJabali/articles/v4/images/blocking_example.png)


## Coroutine Scope
- Controls the lifetime of the coroutine
	- Ex: viewModelScope
- Defines a scope for new coroutines. Every **coroutine builder** (like [launch](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html), [async](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html), etc.) is an extension on [CoroutineScope](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html) and inherits its [coroutineContext](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/coroutine-context.html) to automatically propagate all its elements and cancellation.
## Coroutine Builders

A coroutine builder, as the name implies, builds the coroutine which allows us to use it.

List of coroutine builders
1. Launch
2. Async
3. runBlocking

### 1. Launch
- Launches a new coroutine without blocking the current thread and returns a reference to the coroutine as a Job object.
- Does not give you access to the result outside of the coroutine.
- The coroutine context is inherited from the CoroutineScope.

```kotlin
CoroutineScope.launch(
	context: CoroutineContext = EmptyCoroutineContext,
	start: CoroutineStart = CoroutineStart.Default
) {
	// code
}
```

### Job
- Is a background job, that is cancellable and contains a life-cycle.
- Is returned from the **Launch** coroutine builder.

### 2. Async
- Creates a coroutine and returns its future result as a Deferred object.
- The coroutine context is inherited from the CoroutineScope.

```kotlin
CoroutineScope.async(
	context: CoroutineContext = EmptyCoroutineContext,
	start: CoroutineStart = CoroutineStart.Default
) {
	// code
}
```

### Deferred
- Is a type of **Job** that contains a result.
- Is returned from the **Async** coroutine builder.

