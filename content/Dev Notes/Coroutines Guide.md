---
title: Coroutines Guide
date: 2024-06-01
tags:
  - coroutines
  - article
  - android
  - dispatchers
  - coroutine-cancellation
  - structured-concurrency
  - hierarchy
  - exception-handling
  - coroutine-scope
  - coroutine-context
  - asynchronous
  - cancelling-coroutines
  - coroutine-exception-handler
---
# Introduction

## What is asynchronous programming?
- When you call an asynchronous function that does not return the result immediately, but at some point in the future will.

## Why not do it synchronously?
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

## Callbacks Example
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

## RxJava Example
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
## Coroutines Example
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

---
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

---
# Intermediate
## Understanding Coroutine Context and Dispatchers 
### Intro
- Using coroutines to do operations in the background
- Coroutines context and how to switch context
- Different types of coroutines dispatcher and when to use them
- Coroutine Scope vs Coroutine Context
### Coroutine Context
- Every coroutine has at it's core a context
- The context is defined in the scope of the coroutine
	- Ex: `viewModelScope`
- A coroutine context is an indexed set of element instances
- Context elements are `[dispatcher, job, error handler, coroutine name]`
[coroutine scope]()
- Coroutine scope has a coroutine context as its site property
- Coroutine context is a map of context elements
- Dispatchers, Jobs, Error Handler, and Name
#### What is a dispatcher
- Dispatchers: defines on which thread or thread pool our coroutine will be executed on.
#### What is a job
- Job: Control the lifecycle of a coroutine, and are used to build parent child hierarchy of coroutines
### Coroutine Scope - Coroutine Context illustration 
```kotlin
viewModelScope
     coroutine context
          Dispatchers.Main, SupervisorJob
          Error Handler, Name
```

---
#### Coroutine Dispatchers
##### `Dispatchers.Main`
- Only available on applications with UI
- Special thread(Android Main thread) that can perform UI operations
- Defined as the dispatcher for the `viewModelScope`
- Android `Main` dispatcher uses `Handler` for main looper internally
##### `Dispatchers.IO`
- Performs IO related, blocking operations
- Uses shared thread pool internally, limited to 64 threads
##### `Dispatchers.Default`
- `Default` dispatcher if no other is defined
- Optimized for CPU-intensive work, heavy calculations, sorting a big list, or passing a big JSON file
- Uses a shared thread pool internally, maximum number of threads is equal to the number of CPU cores
##### `Dispatchers.Unconfined`
- Not confined to any thread
- Initially running on the thread the coroutine is started on
##### Custom Dispatchers
- `newSingleThreadContext` - To create a dispatcher that runs on a single thread
- `newFixedPoolContext` - To use a dispatcher that internally uses a thread pool of a specific size
- `java.util.concurrent.Executor` - as coroutine dispatcher

Example
```kotlin
fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job
```

- When we start a coroutine with `launch` the context from the `CoroutineScope` from which `launch` was called is inherited for the new coroutine
#### Coroutine Scope vs Coroutine Context
- A coroutine scope has a single property, a coroutine context
- Every coroutine needs a specific coroutine scope
- Several coroutines can be started within the same coroutine scope
	- Each coroutine inherits the context of the scope
- Many coroutines can run in the same coroutine scope, but in different coroutine contexts
---
## Switching Contexts
- Switching the context of a coroutine allows you to change the dispatcher or other context elements while the coroutine is running. 
	- This is useful when you want to perform different parts of a task on different threads or thread pools.

Example: Switching Between Dispatchers
```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    println("Main program starts on thread: ${Thread.currentThread().name}")

    launch(Dispatchers.Default) {
        println("Running on Default Dispatcher: ${Thread.currentThread().name}")
        
        // Switch to IO Dispatcher
        withContext(Dispatchers.IO) {
            println("Switched to IO Dispatcher: ${Thread.currentThread().name}")
            // Simulate some IO operation
            delay(1000L)
            println("IO Operation completed on: ${Thread.currentThread().name}")
        }

        // Switch back to Default Dispatcher
        println("Back to Default Dispatcher: ${Thread.currentThread().name}")
    }

    println("Main program ends on thread: ${Thread.currentThread().name}")
}
```


---

## Coroutine Scopes in Kotlin

In Kotlin, coroutines are always launched within a scope, which defines the lifecycle and context of the coroutines. Different coroutine scopes are used depending on the context in which you're working (e.g., in an Android application, within a ViewModel, etc.). Below is a list of commonly used coroutine scopes:

### 1. **GlobalScope**
   - **Description**: `GlobalScope` is a global coroutine scope that lives for the entire duration of the application. It's not tied to any specific lifecycle and is generally used for top-level coroutines that are intended to run for the entire lifetime of the application or process.
   - **Usage**:
     ```kotlin
     GlobalScope.launch {
         // Code here runs in GlobalScope
     }
     ```
   - **Caution**: Using `GlobalScope` can lead to memory leaks or unintended behaviors because it’s not bound to any specific lifecycle.

### 2. **runBlocking**
   - **Description**: `runBlocking` creates a coroutine scope and blocks the current thread until all coroutines inside it have completed. It's often used in main functions or for testing purposes.
   - **Usage**:
     ```kotlin
     runBlocking {
         // Code here runs in a blocking coroutine scope
     }
     ```
   - **Common Use Case**: Testing, simple scripts, or to bridge blocking and non-blocking code.

### 3. **CoroutineScope**
   - **Description**: `CoroutineScope` is an interface that can be implemented to define a custom coroutine scope. It’s often used in classes that manage their own lifecycle, such as ViewModels, activities, or custom components.
   - **Usage**:
     ```kotlin
     class MyClass : CoroutineScope {
         private val job = Job()
         override val coroutineContext: CoroutineContext
             get() = Dispatchers.Main + job

         fun doSomething() {
             launch {
                 // Code here runs in MyClass' coroutine scope
             }
         }

         fun clear() {
             job.cancel() // Cancel the scope's coroutines
         }
     }
     ```

### 4. **viewModelScope**
   - **Description**: `viewModelScope` is a coroutine scope provided by the Android Jetpack library specifically for use within a ViewModel. Coroutines launched in this scope are automatically canceled when the ViewModel is cleared.
   - **Usage**:
     ```kotlin
     class MyViewModel : ViewModel() {
         fun fetchData() {
             viewModelScope.launch {
                 // Code here runs in the ViewModel's coroutine scope
             }
         }
     }
     ```

### 5. **lifecycleScope**
   - **Description**: `lifecycleScope` is a coroutine scope tied to the Android `Lifecycle` object, such as an activity or fragment. Coroutines launched in this scope are automatically canceled when the associated lifecycle is destroyed.
   - **Usage**:
     ```kotlin
     class MyActivity : AppCompatActivity() {
         override fun onCreate(savedInstanceState: Bundle?) {
             super.onCreate(savedInstanceState)
             
             lifecycleScope.launch {
                 // Code here runs in the activity's lifecycle scope
             }
         }
     }
     ```

### 6. **MainScope**
   - **Description**: `MainScope` is a convenience scope that creates a `CoroutineScope` with `Dispatchers.Main` and a `Job`. It's commonly used in UI components to run coroutines on the main thread.
   - **Usage**:
     ```kotlin
     class MyComponent {
         private val scope = MainScope()

         fun start() {
             scope.launch {
                 // Code here runs on the Main dispatcher
             }
         }

         fun stop() {
             scope.cancel() // Cancel all coroutines in this scope
         }
     }
     ```

### 7. **supervisorScope**
   - **Description**: `supervisorScope` is similar to `coroutineScope`, but it doesn't propagate exceptions from child coroutines to the parent. This allows other children to continue running even if one fails.
   - **Usage**:
     ```kotlin
     supervisorScope {
         launch {
             // Failing child
             throw Exception("Failed")
         }
         launch {
             // This will continue even if the first coroutine fails
         }
     }
     ```

### 8. **coroutineScope**
   - **Description**: `coroutineScope` creates a coroutine scope and waits for all child coroutines to complete. It is similar to `runBlocking`, but it doesn’t block the underlying thread.
   - **Usage**:
     ```kotlin
     suspend fun doWork() = coroutineScope {
         launch {
             // Code here runs in a coroutine scope
         }
     }
     ```

### Summary

- **`GlobalScope`**: Unbound to any lifecycle, use with caution.
- **`runBlocking`**: Blocking scope for testing or bridging blocking/non-blocking code.
- **`CoroutineScope`**: Custom scopes, commonly used in classes managing their lifecycle.
- **`viewModelScope`**: Lifecycle-aware scope for ViewModels in Android.
- **`lifecycleScope`**: Lifecycle-aware scope for Android activities/fragments.
- **`MainScope`**: Convenience scope for UI components on the main thread.
- **`supervisorScope`**: Error handling scope that allows other coroutines to continue even if one fails.
- **`coroutineScope`**: A suspending function to create a scope that waits for all child coroutines to finish.

---
## Structured Concurrency

### Questions
- If the user leaves the screen before the coroutines have completed, what happens?
	- Do they get cancelled?
	- They should, because they will use a lot of resources. like maybe CPU resources for big expensive calculations and that can drain the battery
	- Uncanceled coroutines would also waste memory, because they would still reference to a view model for an instance. Therefore garbage collector cannot clear up the view model until the coroutine is completed or cancelled. In the worst case the application may crash, because of unnecessary allocated objects and runs out of memory.
### Structured Concurrency
- In structured concurrency:
	1. Every coroutine needs to be started in a logical scope with a limited lifetime. When the lifetime of a coroutine ends and the coroutine has not yet been completed, it will be cancelled.
	2. Coroutines started from the same scope form a hierarchy.
	3. A parent `Job` won't complete until all children are complete
	4. Cancelling a parent will cancel all children recursively. But cancelling a child will not cancel the parent or siblings.
	5. If a child coroutine fails, the exception is propagated upwards and depending on the `Job` type either all siblings are cancelled or not.

[structured concurrency]()

### Example of structure concurrency:
```kotlin
val scope = CoroutineScope(Dispatchers.Default)
fun main() = runBlocking {
	val job = scope.launch {
		delay(100)
		println("Coroutine completed")
	}
	job.invokeOnCompletion { throwable ->
		if(throwable is cancellationException){
			println("Coroutine was cancelled")
		}
	}
	delay(60)
	onDestroy()
}

fun onDestroy(){
	println("Life-time of scope ends")
	scope.cancel()
}
```

Result once your run this:
```
Life-time of scope ends
Coroutine was cancelled
```

| **Structured Concurrency**                                                                                                                | **Unstructured Concurrency**                                                            |
| ----------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Every coroutine needs to be started in a logical scope with a lifetime                                                                    | Threads are started globally  developers responsibility to keep track of their lifetime |
| Coroutines started in the same scope form a hierarchy                                                                                     | No hierarchy. Threads run in isolation without any relationship to each other           |
| A parent `Job` won't complete until children have completed                                                                               | All threads run completely independent of one another                                   |
| Cancelling parents cancels children                                                                                                       | No automatic cancellation mechanism                                                     |
| If a child coroutine fails, the exception is propagated upwards and depending on the `Job` type, either all siblings are cancelled or not | No automatic exception handling and cancelation mechanism                               |

---

## Canceling Coroutines
- `val job = CoroutineScope.launch { ... }`
	- `job.cancel()` cancels the coroutine
- If a `delay(100)` is being used and you cancel the coroutine using it, `delay` throws a `CancellationException`. Because it's coroutine got cancelled.
- Every suspend function of the Kotlin coroutine library also checks if the coroutine from which it is called is still active, otherwise throws `CancellationException`
	- Called cooperative cancellation
### Cooperative Cancellation
### `yield`

- **Purpose**: `yield` is a suspending function that pauses the coroutine to give other coroutines a chance to run. It is also a cooperative check for cancellation, meaning it will throw a `CancellationException` if the coroutine is canceled when `yield` is called.
- **Use Case**: Use `yield` when you want to explicitly give up the current thread's execution to allow other coroutines to run.

### `ensureActive`

- **Purpose**: `ensureActive` is a function that checks whether the coroutine is still active. If the coroutine is canceled, `ensureActive` will throw a `CancellationException`.
- **Use Case**: Use `ensureActive` to periodically check for cancellation within a long-running loop or operation.

### Example: `yield` vs `ensureActive`
```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = launch {
        repeat(5) { i ->
            println("Job: I'm working on $i...")
            yield() // Yield the coroutine to check for cancellation and allow other coroutines to run
        }
    }

    delay(1000L) // Let the job run for a while
    println("Main: I'm tired of waiting!")
    job.cancelAndJoin() // Cancel the job and wait for it to complete
    println("Main: Now I can quit.")
}

suspend fun longRunningTask() = coroutineScope {
    repeat(1000) { i ->
        ensureActive() // Check if the coroutine is still active
        println("Task: Working on part $i")
        // Some long-running computation
    }
}
```

## Non Cancellable `Job`
- In Kotlin coroutines, you can use `withContext(NonCancellable)` to execute a block of code that cannot be canceled, even if the coroutine it is part of is canceled. This is useful in situations where you need to perform certain cleanup or finalization tasks that should not be interrupted by cancellation.
```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = launch {
        try {
            println("Job: Starting work...")
            delay(1000L) // Simulate some work
            println("Job: Work completed!")
        } finally {
            withContext(NonCancellable) {
                println("Job: Cleaning up resources...")
                delay(500L) // Simulate cleanup that cannot be canceled
                println("Job: Cleanup completed.")
            }
        }
    }

    delay(500L)
    println("Main: Cancelling job...")
    job.cancelAndJoin() // Cancel the job and wait for it to complete
```

---
## Exception Handling With Coroutines
- `val handler = CoroutineExceptionHandler{ coroutineContext, throwable ->}`
	- `CoroutineExceptionHandler` is an interface that allows you to define a handler for uncaught exceptions in coroutines.
	- The `CoroutineExceptionHandler` is a context element so we can install it into our coroutine. It is applied to the `CoroutineScope` and can catch exceptions that are not handled by the coroutine itself.
- `val scope = CoroutineScope(Job() + handler)` we can also pass it into our coroutine builders like `launch` and `async`

## Coroutine Exception Handling Example:
```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    // Define a CoroutineExceptionHandler
    val handler = CoroutineExceptionHandler { _, exception ->
        println("Caught $exception")
    }

    val job = launch(handler) {
        println("Job: Starting work...")
        throw RuntimeException("Something went wrong!") // This exception will be caught by the handler
    }

    job.join()
    println("Main: Job has completed.")
}
```

**Explanation**:
- **CoroutineExceptionHandler**:
    - A `CoroutineExceptionHandler` is created to handle uncaught exceptions.
    - The handler is associated with the `launch` coroutine.
- **Exception Propagation**:
    - When the `RuntimeException` is thrown inside the coroutine, it is caught by the `CoroutineExceptionHandler`, preventing the application from crashing.
    - The message `"Caught java.lang.RuntimeException: Something went wrong!"` is printed.
- **Job Completion**:
    - Even though the exception occurred, the coroutine completes after handling the exception, and the main function continues.
## Exception Handling in Coroutine Scope Example

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = launch {
        try {
            println("Job: Starting work...")
            delay(1000L) // Simulate some work
            throw Exception("Error occurred!")
        } catch (e: Exception) {
            println("Caught exception: $e")
        } finally {
            println("Job: Cleaning up...")
        }
    }

    job.join()
    println("Main: Job has completed.")
}
```
### Explanation:
- **Try-Catch Block**:
  - The exception is caught inside the coroutine using a `try-catch` block.
  - This allows the coroutine to handle the exception locally, preventing it from propagating to the parent.

- **Finally Block**:
  - The `finally` block ensures that cleanup code runs regardless of whether an exception was thrown.

## Exception Handling with `SupervisorJob`
- In a parent-child coroutine relationship, a `SupervisorJob` can be used to prevent exceptions in one child coroutine from canceling other sibling coroutines.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val supervisor = SupervisorJob()

    val scope = CoroutineScope(coroutineContext + supervisor)

    val child1 = scope.launch {
        try {
            println("Child1: Starting work...")
            delay(1000L)
            throw RuntimeException("Child1 failed!")
        } catch (e: Exception) {
            println("Child1 caught exception: $e")
        }
    }

    val child2 = scope.launch {
        delay(1500L)
        println("Child2: Completed work successfully.")
    }

    child1.join()
    child2.join()
    println("Main: All children completed.")
}
```

### Explanation:
- **`SupervisorJob`**:
  - The `SupervisorJob` allows `child2` to continue its work even if `child1` fails.
  - `child1` handles its exception locally, preventing it from affecting `child2`.

## Summary
- **`CoroutineExceptionHandler`**: Handles uncaught exceptions at the coroutine scope level.
- **Try-Catch Blocks**: Used within coroutines to catch and handle exceptions locally.
- **`SupervisorJob`**: Prevents exceptions in one child coroutine from canceling other sibling coroutines.
- **Structured Concurrency**: Ensures that exceptions propagate up the hierarchy, allowing for centralized exception handling.
- These tools and patterns help you effectively manage exceptions in coroutines, ensuring that your application can recover gracefully from errors.