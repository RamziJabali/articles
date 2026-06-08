---
title: Building a type-safe networking layer in KMM with Ktor
author: Ramzi Eljabali
date: 2026-06-07
tags:
  - article
  - kotlin-multiplatform
  - compose-multiplatform
  - ktor
  - http-client
  - http
  - architecture
  - networking
---
## Introduction

When working with HTTP in KMM, we want a structure that makes responses, errors, and platform differences predictable rather than something we re-solve on every request. Ktor's Android and iOS engines throw different exception types, and figuring out which engine runs where can be a hassle on its own. So let's spend some time building tools that standardize how we handle HTTP responses and errors, paired with extensions that streamline the request  process.

## Key Points

* Declaring Our HTTP Response Object
* Building Our `Result` DSL
* Building Http Extensions
* Creating HTTP Client Factory With Ktor

## Declaring Our HTTP Response Object

There are a couple of things we need before declaring our response object:

1. Error type
2. Success type

That's pretty much it.

Let's declare an `Error` interface that we'll use to type our errors:

```kotlin
interface Error
```

Let's make another interface to contain all the remote and local errors we need to track (e.g., repository-level requests):

```kotlin
sealed interface DataError: Error {  
    enum class Remote(): DataError {  
        BAD_REQUEST,  
        REQUEST_TIME_OUT,  
        UNAUTHORIZED,  
        FORBIDDEN,  
        NOT_FOUND,  
        CONFLICT,  
        TOO_MANY_REQUESTS,  
        NO_INTERNET,  
        PAYLOAD_TOO_LARGE,  
        SERVER_ERROR,  
        SERVICE_UNAVAILABLE,  
        SERIALIZATION,  
        UNKNOWN  
    }  
  
    enum class Local(): DataError {  
        DISK_FULL,  
        NOT_FOUND,  
        UNKNOWN  
    }  
}
```

Our success case will be generic - we'll call the response object `Result` since the shape of a successful response differs from request to request.

Since we already have our error type abstracted, we can build our `Result` interface that we'll use throughout our project.

```kotlin
sealed interface Result<out D, out E : Error> {  
    data class Success<out D>(val data: D) : Result<D, Nothing>  
    data class Failure<out E : Error>(val error: E) : Result<Nothing, E>  
}
```

* This declares two generic type parameters:
   * `D` - our success case
   * `E` - our error case, which requires all errors to be of type `Error` as declared above
* `out` on both parameters marks them as covariant. This means if `Cat` is a subtype of `Animal`, then `Result<Cat, SomeError>` is treated as a subtype of `Result<Animal, SomeError>`.
* This is what lets you safely return a more specific `Result` type where a more general one is expected.

## Building Our `Result` DSL

With `Result` defined, we can build extension functions that let us work with it just like any other response object.

```kotlin
inline fun <T, E : Error, R> Result<T, E>.map(map: (T) -> R): Result<R, E> {  
    return when(this){  
        is Failure -> Failure(error)  
        is Success -> Success(map(data))  
    }  
}  
  
inline fun <T, E : Error> Result<T, E>.onFailure(onAction: (E) -> Unit): Result<T, E> {  
    return when(this){  
        is Failure -> {  
            onAction(this.error)  
            this  
        }  
        is Success -> this  
    }  
}  
  
fun <T, E : Error> Result<T, E>.asEmptyResult(): EmptyResult<E> {  
    return map{ }  
}  
  
typealias EmptyResult<E> = Result<Unit, E>  
  
inline fun <T, E : Error> Result<T, E>.onSuccess(onAction: (T) -> Unit): Result<T, E> {  
    return when(this){  
        is Failure -> this  
        is Success -> {  
            onAction(this.data)  
            this  
        }  
    }  
}
```

This can look intimidating at first, but here's the approach I use to make sense of any function in this `Result` DSL, a reliable order to parse a function signature:

* Modifiers first (`inline`, `suspend`, `private`, etc.)
* Generic type parameters in `<...>` - note their bounds (`: Error`)
* The receiver, if there's a `Type.` before the function name - that tells you it's an extension function
* The function name and parameters - check if any parameter is itself a function type (lambda)
* The return type after `:`

With that checklist in mind, let's build one last extension function to round out the `Result` DSL:

* A simple `onComplete` extension function
   * Modifier: `inline`
   * Generic type parameter: `<T, E: Error>`
   * Receiver type: `Result<T, E>`
   * Function name: `onComplete`
   * Parameter: `onAction: (Result<T, E>) -> Unit`
   * Return Type: `Result<T, E>`
* Put it all together:

```kotlin
inline fun <T, E: Error> Result<T, E>.onComplete(onAction: (Result<T, E>) -> Unit) : Result<T, E> {
    onAction(this)
    return this
}
```

## Building Http Extensions

Now let's dive into the remote side of things, which we'll handle using Ktor's `HttpClient`.

For help setting up Ktor, check out my guide, [Ktor In KMM](https://ramzijabali.github.io/articles/Dev-Notes/Ktor-In-KMM).

We'll use KMM's `expect`/`actual` pattern to handle platform-specific differences.

Let's start by declaring an `expect`/`actual` function in `commonMain` that each platform will implement to perform API calls:

```kotlin
expect suspend fun <T> platformSafeCall(  
    execute: suspend () -> HttpResponse,  
    handleResponse: suspend (HttpResponse) -> Result<T, DataError.Remote>  
): Result<T, DataError.Remote>
```

We pass in two lambdas: one to execute the call, and one to handle its response.

This is where the engine differences between Android and iOS come into play:

* Android uses the `okHttp` HttpClientEngine
* iOS uses the `Darwin` HttpClientEngine

Since they don't share the same exception types, each platform needs its own handling. Which is exactly what our `expect`/`actual` pattern is for.

### Android Actual Implementation

```kotlin
actual suspend fun <T> platformSafeCall(  
    execute: suspend () -> HttpResponse,  
    handleResponse: suspend (HttpResponse) -> Result<T, DataError.Remote>  
): Result<T, DataError.Remote> {  
   return try {  
        val response = execute()  
        handleResponse(response)  
    } catch (e: UnknownHostException) {  
        Result.Failure(DataError.Remote.NO_INTERNET)  
    } catch (e: UnresolvedAddressException) {  
        Result.Failure(DataError.Remote.NO_INTERNET)  
    } catch (e: ConnectException) {  
        Result.Failure(DataError.Remote.NO_INTERNET)  
    } catch (e: SocketTimeoutException) {  
        Result.Failure(DataError.Remote.REQUEST_TIME_OUT)  
    } catch (e: HttpRequestTimeoutException) {  
        Result.Failure(DataError.Remote.REQUEST_TIME_OUT)  
    } catch (e: SerializationException) {  
        Result.Failure(DataError.Remote.SERIALIZATION)  
    } catch (e: Exception) {  
       currentCoroutineContext().ensureActive()  
        Result.Failure(DataError.Remote.UNKNOWN)  
    }  
}
```

### iOS Actual Implementation

Since iOS uses the `Darwin` `HttpClientEngine`, we can't mirror the Android implementation 1:1. We need to handle its distinct exception types instead.

```kotlin
actual suspend fun <T> platformSafeCall(  
    execute: suspend () -> HttpResponse,  
    handleResponse: suspend (HttpResponse) -> Result<T, DataError.Remote>  
): Result<T, DataError.Remote> {  
    return try {  
        val response = execute()  
        handleResponse(response)  
    } catch (e: DarwinHttpRequestException) {  
        handleDarwinException(e)  
    } catch (e: UnresolvedAddressException) {  
        Result.Failure(DataError.Remote.NO_INTERNET)  
    } catch (e: SocketTimeoutException) {  
        Result.Failure(DataError.Remote.REQUEST_TIME_OUT)  
    } catch (e: HttpRequestTimeoutException) {  
        Result.Failure(DataError.Remote.REQUEST_TIME_OUT)  
    } catch (e: SerializationException) {  
        Result.Failure(DataError.Remote.SERIALIZATION)  
    } catch (e: Exception) {  
        currentCoroutineContext().ensureActive()  
        Result.Failure(DataError.Remote.UNKNOWN)  
    }  
}  
  
private fun handleDarwinException(e: DarwinHttpRequestException): Result<Nothing, DataError.Remote> {  
    val nsError = e.origin  
    return if (nsError.domain == NSURLErrorDomain){  
        when(nsError.code) {  
            NSURLErrorNotConnectedToInternet,  
            NSURLErrorNetworkConnectionLost,  
            NSURLErrorCannotFindHost,  
            NSURLErrorDNSLookupFailed,  
            NSURLErrorResourceUnavailable,  
            NSURLErrorInternationalRoamingOff,  
            NSURLErrorCallIsActive,  
            NSURLErrorDataNotAllowed -> Result.Failure(DataError.Remote.NO_INTERNET)  
            NSURLErrorTimedOut -> Result.Failure(DataError.Remote.REQUEST_TIME_OUT)  
            else -> Result.Failure(DataError.Remote.UNKNOWN)  
        }  
    } else Result.Failure(DataError.Remote.UNKNOWN)  
}
```

This lets both platforms make requests while returning the same streamlined result types back to our common code.

## Common Http Utilities

1. Let's define a function that converts Ktor's `HttpResponse` into our `Result` type:

```kotlin
suspend inline fun <reified T> responseToResult(response: HttpResponse): Result<T, DataError.Remote> {  
    return when(response.status.value){  
        in 200 .. 299 -> {  
            try {  
                Result.Success(response.body<T>())  
            } catch (e: NoTransformationFoundException) {  
                Result.Failure(DataError.Remote.SERIALIZATION)  
            }  
        }  
        400 -> Result.Failure(DataError.Remote.BAD_REQUEST)  
        401 -> Result.Failure(DataError.Remote.UNAUTHORIZED)  
        403 -> Result.Failure(DataError.Remote.FORBIDDEN)  
        404 -> Result.Failure(DataError.Remote.NOT_FOUND)  
        408 -> Result.Failure(DataError.Remote.REQUEST_TIME_OUT)  
        413 -> Result.Failure(DataError.Remote.PAYLOAD_TOO_LARGE)  
        429 -> Result.Failure(DataError.Remote.TOO_MANY_REQUESTS)  
        500 -> Result.Failure(DataError.Remote.SERVER_ERROR)  
        503 -> Result.Failure(DataError.Remote.SERVICE_UNAVAILABLE)  
        else -> Result.Failure(DataError.Remote.UNKNOWN)  
    }  
}
```

2. Now that we have `platformSafeCall`, let's wrap it in a function that supplies the missing `handleResponse` parameter:

```kotlin
suspend inline fun <reified T> safeCall(  
    noinline execute: suspend () -> HttpResponse  
) : Result<T, DataError.Remote> {  
    return platformSafeCall(  
        execute = execute,  
        handleResponse = { response ->  
            responseToResult(response)  
        }  
    )  
}
```

3. Let's add a `constructRoute` function that uses your `BASE_URL` to build full API URLs:

```kotlin
fun constructRoute(route: String): String =  
    when {  
        route.contains(BASE_URL) -> route  
        route.startsWith("/") -> "$BASE_URL$route"  
        else -> "$BASE_URL/$route"  
    }
```

4. Finally, let's wrap our four HTTP methods as extension functions on Ktor's `HttpClient`, so you don't have to rewrite the same boilerplate every time you make a request.

```kotlin
suspend inline fun <reified Request, reified  Response: Any> HttpClient.post(  
    route: String,  
    queryParams: Map<String, Any> = mapOf(),  
    body: Request,  
    crossinline builder : HttpRequestBuilder.() -> Unit = {}  
): Result<Response, DataError.Remote> {  
    return safeCall {  
        post {  
            url(constructRoute(route))  
            queryParams.forEach { (key, value) ->  
                parameter(key, value)  
            }  
            setBody(body)  
            builder()  
        }  
    }
}  
  
suspend inline fun <reified  Response: Any> HttpClient.get(  
    route: String,  
    queryParams: Map<String, Any> = mapOf(),  
    crossinline builder : HttpRequestBuilder.() -> Unit = {}  
): Result<Response, DataError.Remote> {  
    return safeCall {  
        get {  
            url(constructRoute(route))  
            queryParams.forEach { (key, value) ->  
                parameter(key, value)  
            }  
            builder()  
        }  
    }
}  
  
suspend inline fun <reified  Response: Any> HttpClient.delete(  
    route: String,  
    queryParams: Map<String, Any> = mapOf(),  
    crossinline builder : HttpRequestBuilder.() -> Unit = {}  
): Result<Response, DataError.Remote> {  
    return safeCall {  
        delete {  
            url(constructRoute(route))  
            queryParams.forEach { (key, value) ->  
                parameter(key, value)  
            }  
            builder()  
        }  
    }
}  
  
suspend inline fun <reified Request, reified  Response: Any> HttpClient.put(  
    route: String,  
    queryParams: Map<String, Any> = mapOf(),  
    body: Request,  
    crossinline builder : HttpRequestBuilder.() -> Unit = {}  
): Result<Response, DataError.Remote> {  
    return safeCall {  
        put {  
            url(constructRoute(route))  
            queryParams.forEach { (key, value) ->  
                parameter(key, value)  
            }  
            setBody(body)  
            builder()  
        }  
    }
}
```

## Creating HTTP Client Factory With Ktor

This factory is built to be customizable - for example, you can swap in your own `Logger` in place of Ktor's default.

```kotlin
class HttpClientFactory(private val justMessageLogger: JustMessageLogger) {  
    fun create(engine: HttpClientEngine): HttpClient {  
        return HttpClient(engine) {  
            install(ContentNegotiation) {  
                json(  
                    json = Json {  
                        ignoreUnknownKeys = true  
                    }  
                )  
            }  
            install(HttpTimeout) {  
                socketTimeoutMillis = 20_000L  
                requestTimeoutMillis = 20_000L  
            }  
            install(Logging) {  
                logger = object : Logger {  
                    override fun log(message: String) {  
                        justMessageLogger.debug(message)  
                    }  
                }  
                level = LogLevel.ALL  
            }  
            install(WebSockets) {  
                pingIntervalMillis = 20_000L  
            }  
            defaultRequest {  
                header("x-api-key", BuildKonfig.API_KEY)  
                contentType(ContentType.Application.Json)  
            }  
        }    
    }  
}
```
