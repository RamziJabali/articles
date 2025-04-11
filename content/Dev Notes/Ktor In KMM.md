---
title: Ktor In KMM
author: Ramzi Eljabali
date: 2025-02-08
tags:
  - article
  - ktor
  - kotlin
  - kotlin-multiplatform
  - kotlin-serialization
  - api
  - incomplete
---

# Ktor In KMM

## Introduction
While working on my personal Kotlin multiplatform project, I was looking for a multiplatform http client to help with putting my project together. I came across Ktor a http client built with Kotlin and coroutines that works on multiplatform projects.

## Key Points
- **Point 1:** Ktor project setup
- **Point 2:** Http client configuration
- **Point 3:** Making API call

## Section 1: Ktor project setup
### Adding Dependencies 
- We want to add the following dependencies 
```kotlin
ktor-client-core = { module = "io.ktor:ktor-client-core", version.ref = "ktor" } 

ktor-client-okhttp = { module = "io.ktor:ktor-client-okhttp", version.ref = "ktor" }  

ktor-client-darwin = { module = "io.ktor:ktor-client-darwin", version.ref = "ktor" }  

ktor-client-logging = {module = "io.ktor:ktor-client-logging", version.ref = "ktor"}  

ktor-client-content-negotiation = {module = "io.ktor:ktor-client-content-negotiation", version.ref = "ktor"}  

ktor-serialization-json = {module = "io.ktor:ktor-serialization-kotlinx-json", version.ref = "ktor"}

kotlinxSerialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
```
-  With those dependencies we want to start adding them to our gradle 
	- When we're targeting `iOS` we use the `darwin` engine dependency,  when targeting `android` we use the `okhttp` engine dependency, and for `commonMain` source set we will use the `core` http client dependency.
	- We also need to add ktor JSON serialization plugin to be able to deserialize the JSON response into our Kotlin objects. We need to install the `ContentNegotiation` plugin with `KotlinxSerialization`, and we also want to install `Logging` so we need to use logging dependency.
```kotlin
plugins {
	alias(libs.plugins.kotlinxSerialization)
}

sourceSets {  
	iosMain.dependencies {  
	    implementation(libs.ktor.client.darwin)  
	}  
	androidMain.dependencies {  
	    implementation(libs.ktor.client.okhttp)  
	}  
  
	commonMain.dependencies {  
	    //put your multiplatform dependencies here  
	    implementation(libs.ktor.client.core)  
	    implementation(libs.ktor.client.logging)  
	    implementation(libs.ktor.client.content.negotiation)  
	    implementation(libs.ktor.serialization.json)  
	}
}
```

## Section 2: Http client configuration
```kotlin
import io.ktor.client.HttpClient  
import io.ktor.client.engine.HttpClientEngine  
import io.ktor.client.plugins.contentnegotiation.ContentNegotiation  
import io.ktor.client.plugins.logging.DEFAULT  
import io.ktor.client.plugins.logging.LogLevel  
import io.ktor.client.plugins.logging.Logger  
import io.ktor.client.plugins.logging.Logging  
import io.ktor.serialization.kotlinx.json.json  
import kotlinx.serialization.json.Json  
  
fun createHttpClient(client: HttpClientEngine): HttpClient {  
    return HttpClient(client) {  
        install(Logging){  
            logger = Logger.DEFAULT  
            level = LogLevel.ALL  
        }  
        install(ContentNegotiation){  
            json(  
                json = Json {  
                    prettyPrint = true  
                    isLenient = true  
                    // if API returns JSON fields we don't have  
                    // defined it will ignore it and will not crash our app                    ignoreUnknownKeys = true  
                },  
                contentType =  
            )  
        }  
    }
}
```
## Section 3: Making API call


## Conclusion
<!-- Summarize the article, restating the key ideas. You can also end with a call to action or closing thoughts. -->

## References
<!-- If you have used any sources, list them here for further reading or citation purposes. -->
- [Content Negotiation and Serialization](https://ktor.io/docs/client-serialization.html#serialization_dependency)
- [# Creating a cross-platform mobile application](https://ktor.io/docs/client-create-multiplatform-application.html#ktor-dependencies)

