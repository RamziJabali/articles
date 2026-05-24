---
title: Theming In Compose
author: Ramzi Eljabali
date: 2026-05-24
tags:
  - themeing
  - compose
  - kotlin-multiplatform
  - android
  - material-3
  - compose-multiplatform
---
# Compose Multiplatform Theme and Resources Guide

## Key Points

- Creating `composeResources`
- Defining typography
- Defining colors and color schemes
- Defining extended theme colors
- Defining a custom theme
- Generating the `Res` class
- Troubleshooting common resource generation issues

---

## Creating `composeResources`

Skip this section if you're working in a regular Android-only module. Android modules can use the standard `src/main/res` directory.

If you're working in a Kotlin Multiplatform or Compose Multiplatform project, create a `composeResources` directory under `commonMain`:

```text
shared/
  src/
    commonMain/
      composeResources/
        drawable/
        font/
        values/
```

This makes the resources accessible across the platforms your shared module targets.

After syncing Gradle or running a build, Compose Multiplatform generates a `Res` object that lets you reference resources like:

```kotlin
Res.font.plusjakartasans_regular
Res.drawable.compose_multiplatform
```

---

## Typography

### Defining a Font Family

Browse [Google Fonts](https://fonts.google.com) or another font source and download the font family you want to use.

Place your `.ttf` font files under:

```text
src/commonMain/composeResources/font
```

Then sync Gradle or build the project so the generated `Res.font` accessors are available.

Example:

```kotlin
val PlusJakartaSans
    @Composable get() = FontFamily(
        Font(
            resource = Res.font.plusjakartasans_light,
            weight = FontWeight.Light,
        ),
        Font(
            resource = Res.font.plusjakartasans_regular,
            weight = FontWeight.Normal,
        ),
        Font(
            resource = Res.font.plusjakartasans_medium,
            weight = FontWeight.Medium,
        ),
        Font(
            resource = Res.font.plusjakartasans_semibold,
            weight = FontWeight.SemiBold,
        ),
        Font(
            resource = Res.font.plusjakartasans_bold,
            weight = FontWeight.Bold,
        ),
    )
```

---

### Defining Typography

You can build your app typography using Material 3's `Typography` class.

Reference: [Material 3 Typography](https://m3.material.io/styles/typography/overview)

Avoid naming your app typography value `Typography`, because that conflicts with the Material 3 `Typography` type name. A clearer name is `AppTypography`.

```kotlin
val AppTypography
    @Composable get() = Typography(
        titleLarge = TextStyle(
            fontFamily = PlusJakartaSans,
            fontWeight = FontWeight.SemiBold,
            fontSize = 30.sp,
            lineHeight = 36.sp,
        ),
        titleMedium = TextStyle(
            fontFamily = PlusJakartaSans,
            fontWeight = FontWeight.SemiBold,
            fontSize = 20.sp,
            lineHeight = 28.sp,
        ),
        titleSmall = TextStyle(
            fontFamily = PlusJakartaSans,
            fontWeight = FontWeight.SemiBold,
            fontSize = 16.sp,
            lineHeight = 24.sp,
        ),
        bodyLarge = TextStyle(
            fontFamily = PlusJakartaSans,
            fontWeight = FontWeight.Normal,
            fontSize = 18.sp,
            lineHeight = 26.sp,
        ),
        bodyMedium = TextStyle(
            fontFamily = PlusJakartaSans,
            fontWeight = FontWeight.Normal,
            fontSize = 16.sp,
            lineHeight = 24.sp,
        ),
        bodySmall = TextStyle(
            fontFamily = PlusJakartaSans,
            fontWeight = FontWeight.Normal,
            fontSize = 14.sp,
            lineHeight = 20.sp,
        ),
        labelMedium = TextStyle(
            fontFamily = PlusJakartaSans,
            fontWeight = FontWeight.Medium,
            fontSize = 16.sp,
            lineHeight = 24.sp,
        ),
        labelSmall = TextStyle(
            fontFamily = PlusJakartaSans,
            fontWeight = FontWeight.SemiBold,
            fontSize = 14.sp,
            lineHeight = 20.sp,
        ),
        headlineSmall = TextStyle(
            fontFamily = PlusJakartaSans,
            fontWeight = FontWeight.SemiBold,
            fontSize = 14.sp,
            lineHeight = 18.sp,
        ),
        displaySmall = TextStyle(
            fontFamily = PlusJakartaSans,
            fontWeight = FontWeight.SemiBold,
            fontSize = 11.sp,
            lineHeight = 14.sp,
        ),
    )
```

---

### Defining Extra Typography Tokens

If you have typography tokens that are not covered by the `Typography` constructor, you can expose them as extension properties.

```kotlin
val Typography.labelXSmall: TextStyle
    @Composable get() = TextStyle(
        fontFamily = PlusJakartaSans,
        fontWeight = FontWeight.SemiBold,
        fontSize = 11.sp,
        lineHeight = 14.sp,
    )

val Typography.titleXSmall: TextStyle
    @Composable get() = TextStyle(
        fontFamily = PlusJakartaSans,
        fontWeight = FontWeight.SemiBold,
        fontSize = 14.sp,
        lineHeight = 18.sp,
    )
```

Usage:

```kotlin
Text(
    text = "Small label",
    style = MaterialTheme.typography.labelXSmall,
)
```

---

## Color

### Gathering Colors

Start by defining your raw color tokens. This makes it easier to understand what each color represents before plugging them into a Material color scheme.

Reference: [Material 3 Color System](https://m3.material.io/styles/color/system/overview)

You do not need to define every Material 3 color role immediately. Start with the core roles your UI needs, then add more as your design system grows.

---

### Defining Color Tokens

```kotlin
// ---------- Light Colors ----------

val LightPrimary = Color(0xFF4F46E5)
val LightOnPrimary = Color(0xFFFFFFFF)
val LightPrimaryContainer = Color(0xFFE0E7FF)
val LightOnPrimaryContainer = Color(0xFF312E81)

val LightSecondary = Color(0xFF06B6D4)
val LightOnSecondary = Color(0xFFFFFFFF)

val LightBackground = Color(0xFFF8FAFC)
val LightOnBackground = Color(0xFF0F172A)

val LightSurface = Color(0xFFFFFFFF)
val LightOnSurface = Color(0xFF1E293B)

val LightError = Color(0xFFDC2626)
val LightOnError = Color(0xFFFFFFFF)

// ---------- Dark Colors ----------

val DarkPrimary = Color(0xFF818CF8)
val DarkOnPrimary = Color(0xFF0F172A)
val DarkPrimaryContainer = Color(0xFF3730A3)
val DarkOnPrimaryContainer = Color(0xFFE0E7FF)

val DarkSecondary = Color(0xFF22D3EE)
val DarkOnSecondary = Color(0xFF082F49)

val DarkBackground = Color(0xFF020617)
val DarkOnBackground = Color(0xFFF8FAFC)

val DarkSurface = Color(0xFF0F172A)
val DarkOnSurface = Color(0xFFE2E8F0)

val DarkError = Color(0xFFF87171)
val DarkOnError = Color(0xFF450A0A)

// ---------- Additional Error Colors ----------

val Red600 = Color(0xFFAA142A)
val Red500 = Color(0xFFDA233E)
val Red200 = Color(0xFFFF7987)

// ---------- Light Theme Success Colors ----------

val LightSuccess = Color(0xFF16A34A)
val LightOnSuccess = Color(0xFFFFFFFF)

val LightSuccessContainer = Color(0xFFDCFCE7)
val LightOnSuccessContainer = Color(0xFF14532D)

// ---------- Dark Theme Success Colors ----------

val DarkSuccess = Color(0xFF4ADE80)
val DarkOnSuccess = Color(0xFF052E16)

val DarkSuccessContainer = Color(0xFF166534)
val DarkOnSuccessContainer = Color(0xFFDCFCE7)
```

---

## Defining Color Schemes

Avoid naming your color scheme values `LightColorScheme` and `DarkColorScheme` if you want to make it clear they belong to your app. Prefer names like `AppLightColorScheme` and `AppDarkColorScheme`.

```kotlin
val AppLightColorScheme = lightColorScheme(
    primary = LightPrimary,
    onPrimary = LightOnPrimary,
    primaryContainer = LightPrimaryContainer,
    onPrimaryContainer = LightOnPrimaryContainer,

    secondary = LightSecondary,
    onSecondary = LightOnSecondary,

    background = LightBackground,
    onBackground = LightOnBackground,

    surface = LightSurface,
    onSurface = LightOnSurface,

    error = LightError,
    onError = LightOnError,
)

val AppDarkColorScheme = darkColorScheme(
    primary = DarkPrimary,
    onPrimary = DarkOnPrimary,
    primaryContainer = DarkPrimaryContainer,
    onPrimaryContainer = DarkOnPrimaryContainer,

    secondary = DarkSecondary,
    onSecondary = DarkOnSecondary,

    background = DarkBackground,
    onBackground = DarkOnBackground,

    surface = DarkSurface,
    onSurface = DarkOnSurface,

    error = DarkError,
    onError = DarkOnError,
)
```

---

## Defining Extended Colors

Some design tokens do not map directly to Material 3's built-in `ColorScheme` roles.

For example, Material 3 has `error`, but it does not have a built-in `success` color role. For app-specific roles like `success`, create an extended color model.

```kotlin
@Immutable
data class ExtendedColors(
    val success: Color,
    val onSuccess: Color,
    val successContainer: Color,
    val onSuccessContainer: Color,
)
```

Then define a light and dark version:

```kotlin
val LightExtendedColors = ExtendedColors(
    success = LightSuccess,
    onSuccess = LightOnSuccess,
    successContainer = LightSuccessContainer,
    onSuccessContainer = LightOnSuccessContainer,
)

val DarkExtendedColors = ExtendedColors(
    success = DarkSuccess,
    onSuccess = DarkOnSuccess,
    successContainer = DarkSuccessContainer,
    onSuccessContainer = DarkOnSuccessContainer,
)
```

---

### Providing Extended Colors with a CompositionLocal

Use a `CompositionLocal` to provide your extended colors throughout the composable tree.

```kotlin
val LocalExtendedColors = staticCompositionLocalOf {
    LightExtendedColors
}
```

A clean way to expose these colors is through an `ColorScheme` extension:

```kotlin
val ColorScheme.extended: ExtendedColors  
    @ReadOnlyComposable  
    @Composable    
    get() = LocalExtendedColors.current
```

Usage:

```kotlin
Text(
    text = "Success",
    color = MaterialTheme.colorScheme.extended.error
)
```

You could also expose extended colors from `ColorScheme`, but that can be slightly misleading because the extended colors are not actually stored inside `ColorScheme`; they are provided through `LocalExtendedColors`.

---

## Defining Your Theme

Create a reusable theme composable that decides whether to use the light or dark color scheme.

```kotlin
@Composable
fun CustomTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit,
) {
    val colorScheme = if (darkTheme) {
        AppDarkColorScheme
    } else {
        AppLightColorScheme
    }

    val extendedColors = if (darkTheme) {
        DarkExtendedColors
    } else {
        LightExtendedColors
    }

    CompositionLocalProvider(
        LocalExtendedColors provides extendedColors,
    ) {
        MaterialTheme(
            colorScheme = colorScheme,
            typography = AppTypography,
            content = content,
        )
    }
}
```

---

## Example Usage

```kotlin
CustomTheme {
    var showContent by remember { mutableStateOf(false) }

    Column(
        modifier = Modifier
            .background(MaterialTheme.colorScheme.primaryContainer)
            .safeContentPadding()
            .fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally,
    ) {
        Button(onClick = { showContent = !showContent }) {
            Text("Click me!")
        }

        AnimatedVisibility(showContent) {
            val greeting = remember { Greeting().greet() }

            Column(
                modifier = Modifier.fillMaxWidth(),
                horizontalAlignment = Alignment.CenterHorizontally,
            ) {
                Image(
                    painter = painterResource(Res.drawable.compose_multiplatform),
                    contentDescription = null,
                )

                Text("Compose: $greeting")
            }
        }
    }
}
```

---

## Generating `composeResources`

If your `Res` object is not being generated, update your Gradle setup.

In your module `build.gradle.kts`, add the Compose resources dependency to `commonMain.dependencies`:

```kotlin
kotlin {
    sourceSets {
        commonMain.dependencies {
            implementation(compose.components.resources)
        }
    }
}
```

Then configure resource class generation:

```kotlin
import org.jetbrains.compose.resources.ResourcesExtension.ResourceClassGeneration.Always

compose.resources {
    generateResClass = Always
    packageOfResClass = "com.yourpackage.generated.resources"
}
```

Replace `com.yourpackage.generated.resources` with the package where you want the generated `Res` class to live.

For example:

```kotlin
packageOfResClass = "com.ramzi.theme.generated.resources"
```

Then import it where needed:

```kotlin
import com.ramzi.theme.generated.resources.Res
```

---

## Common Issues

### `Unresolved reference: Res`

Try the following:

1. Confirm your resources are under `src/commonMain/composeResources`.
2. Confirm `implementation(compose.components.resources)` is added to `commonMain.dependencies`.
3. Sync Gradle.
4. Run a clean build.
5. Confirm your `packageOfResClass` matches the import you are using.

---

### `Unresolved reference: resources`

If this does not resolve:

```kotlin
compose.resources {
    generateResClass = Always
}
```

Make sure the Compose Gradle plugin is applied in the module where you are configuring resources.

Example:

```kotlin
plugins {
    id("org.jetbrains.compose")
}
```

Depending on your project setup, this may be applied directly in the module or through a convention plugin.

---

### `Annotation argument must be a compile-time constant`

This usually happens when a generated resource reference is used somewhere that requires a compile-time constant.

Generated Compose resources are not the same as Android `R` constants. Avoid using `Res.font`, `Res.drawable`, or other generated Compose resource values in annotation arguments or places that require compile-time constants.

---

### Font resource is not generated

Check that:

1. The file is inside `src/commonMain/composeResources/font`.
2. The file name uses lowercase letters, numbers, and underscores.
3. The file extension is supported, such as `.ttf`.
4. Gradle has been synced after adding the file.
5. A clean build has been run if the generated accessor still does not appear.

Example valid file name:

```text
plusjakartasans_regular.ttf
```

Example generated reference:

```kotlin
Res.font.plusjakartasans_regular
```
