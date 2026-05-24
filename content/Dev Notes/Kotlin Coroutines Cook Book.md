---
title: Kotlin Coroutines Cookbook
author: Ramzi Eljabali
date: 2026-05-22
tags:
  - kotlin
  - coroutines
  - corourine-flow
  - cook-book
---

# Cook Book

1. [[#Onetime events observation using coroutine channels.|Onetime events observation using coroutine channels.]]
	- Using `Main.immediate` since we don't want to miss any events that may be missed due to recomposition/navigation timing issues.

## Onetime events observation using coroutine channels.

```kotlin
@Composable  
fun <T> ObserveAsEvents(  
    flow: Flow<T>,  
    key1: Any? = null,  
    key2: Any? = null,  
    onEvent: (T) -> Unit  
) {  
    val lifecycleOwner = LocalLifecycleOwner.current  
    LaunchedEffect(lifecycleOwner, key1, key2) {  
        lifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {  
            withContext(Dispatchers.Main.immediate) {  
                flow.collect(onEvent)  
            }  
        }    
    }
}
```

```kotlin
class MainViewModel: ViewModel() {
	private val navigationChannel = Channel<NavigationEvent>
	val navigationEventsChannelFlow = navigationChannel.receiveAsFlow()
	
	fun login() {
		navigationChannel.send(NavigationEvent.navigateToProfile)
	}
}
```

---

