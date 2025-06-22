---
title: Dynamic Padding with Window Insets
author: Ramzi Eljabali
date: 2025-06-08
tags:
  - article
  - compose
  - window-insets
  - design-tokens
  - ui
  - status-bar
  - navigation-bar
---

# Dynamic Padding with Window Insets

## Introduction

 With Android 15 enforcing apps to be edge to edge, I needed to find the best way of implementing it without ruining how the app looks and functions. This lead to me discovering `WindowInsets`, window insets provide information about the system UI. This allows us to   access the values of those system UI elements like the `navigation bar` or the `status bar`. 

##### How does this allows us to implement edge to edge appropriately?
With access to the values of system UI elements, that dynamically change, we will be able to use those values to extend our background behind the system UI elements while also ensuring our UI elements are NOT obstructed. 

## Example: 

```kotlin
@Composable  
fun JustJogScaffold() {  
    Box {  
        Scaffold(  
            modifier = Modifier.fillMaxSize(),  
            bottomBar = {  
                JustJogBottomNavigation(  
                    modifier = Modifier.navigationBarsPadding()  
                )            },  
            floatingActionButton = { /* future requirement */ }  
        ) { padding ->  
        }  
    }
}
```

```kotlin
@Composable  
fun JustJogBottomNavigation(  
    modifier: Modifier = Modifier  
) {  
    var bottomNavCurrentIndex: Int by remember { mutableStateOf(0) }  
    BottomNavigation(  
        backgroundColor = Color.DarkGray,  
        contentColor = Color.White,  
        elevation = BottomNavigationDefaults.Elevation  
    ) {  
        JustJogBottomNavigationItems.entries.forEach { bottomNavItem ->  
            BottomNavigationItem(  
                selected = bottomNavItem.index == bottomNavCurrentIndex,  
                onClick = {  
                    bottomNavCurrentIndex = bottomNavItem.index  
                },  
                icon = {  
                    Icon(  
                        painterResource(bottomNavItem.icon),  
                        contentDescription = "${bottomNavItem.itemName} Icon"                    )  
                },  
                modifier = modifier,  
                enabled = true,  
                label = { Text(bottomNavItem.itemName) },  
                alwaysShowLabel = false,  
                selectedContentColor = Color.Yellow,  
                unselectedContentColor = Color.LightGray  
            )  
        }  
    }
}
```

Passing that modifier into the `BottomNavigationItem` makes it so it adds the necessary padding 
to bottom or left/right if you're using 3 button navigation. 

There is also means of accessing the values of the hardware cutouts in order to adapt your UI to it.
## Conclusion

`WindowInsets` and it's functions makes accessing system UI element properties easy and give you a means of dynamically adapting your views to the system UI.

## References
- [Insets](https://developer.android.com/develop/ui/compose/system/insets)


