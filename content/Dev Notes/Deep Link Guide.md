---
tags:
  - android
  - article
  - deep-link
  - github
date: 2024-07-14
title: Deep Link Guide
---
# What is Deep Link?
- A Deep Link is a URI of any scheme that takes the user directly to a specific part of your application.
- Using this URL we can send a message to our application with parameters.

# What is it used for?
- Deep Links are used for directing users to specific parts of your application, or other applications all together.
	- This allows for the user to bypass certain parts of your application allowing for an easier experience.

# How to create Deep Links
1. You will have to first define an `intent-filter` for it within your `AndroidManifest.xml` file.
	- These drive users to the right activity in your application.

Looks like this:
```xml
<activity    
	android:name=".MyMapActivity"    
	android:exported="true"    
	...>    
	<intent-filter android:autoVerfiy="true">        
		  <action android:name="android.intent.action.VIEW" />        
		  <category android:name="android.intent.category.DEFAULT" />
		  <category android:name="android.intent.category.BROWSABLE" />
		<!-- Scheme and host for the deep link -->
		  <data android:scheme="example" android:host="content"                                  android:pathPrefix="/path/to/content" />  
	</intent-filter>  
</activity>
```


Let's go through what each tag and it's attribute means.
1. The `<activity>` tag is defines the activities we have within our application.
	- `android:name` attribute specifies the name of the activity.
	- `android:exported` attribute declares that this activity is accessible outside this application.
2. The `<intent-filter>` tag declares what type of intents this application can respond to.
	- `android:autoVerify` attribute ensures that the deep link is verified within our system.
		- If it's set to `true`, the system tries to verify that the specified domain is associated with your application.
	- `<action>` tag specified the action that your application can handle.
		- `android:name` attribute defines the action name
		- `android:intent.action.VIEW` tells the system that this activity can respond to a `view` actions.
	- `<category>` tag adds a category to our `intent-filter`, which specifies additional information about the kinds of `Intents` the activity can handle.
		- `android.intent.category.DEFAULT` declares our activity as an entry point to our application
		- `android.intent.category.BROWSABLE` declares that our activity is capable of being reached via a web link. This makes it accessible to web browsers and other applications.
	- `<data>` tag specifies the data that the `URI` pattern can handle.
		- `android:scheme` attribute defines the URI scheme(e.g. `HTTP`, `HTTPS`, or a custom scheme like `example`).
		- `android:host` attribute specifies the host part of the URI(e.g., `www.example.com` or a custom host like `content`).
		- `android:pathPrefix` attribute defines the initial part of the path that the `URI` must match. 
			- In this case, it matches with URIs starting with `/path/to/content`
			- So with our given configuration our `MainActivity` can handle deep links that match the following pattern
				- `example://content/path/to/content`
					- `example` being the custom scheme 
					- `content` being the custom host
					- `/path/to/content` being the path 

# Deep Linking to launch other applications
1. Configure Deep Linking in Your Application 
2. Handle the Deep Link in Your Activity
3. Launch Your App via Deep Link from Another Application

## 1. Configure Deep Linking in Your Application
- Set up deep link in your applications `AndroidManifest.xml`
Example:
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.yourapp">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">

        <!-- Main entry point of the app -->
        <activity android:name=".MainActivity">
            <intent-filter android:autoVerify="true">
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />

                <!-- Scheme and host for the deep link -->
                <data android:scheme="example" android:host="content"                                  android:pathPrefix="/path/to/content" />
            </intent-filter>
        </activity>
    </application>
</manifest>

```

In this configuration:
- The `android:scheme` is set to `example`.
- The `android:host` is set to `content`.
- The `android:pathPrefix` is set to `/path/to/content`.

## 2. Handle the Deep Link in Your Activity]

In your `MainActivity`, handle the incoming deep link.

Example:

```kotlin
package com.example.yourapp

import android.content.Intent
import android.net.Uri
import android.os.Bundle
import android.util.Log
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        handleIntent(intent)
    }

    override fun onNewIntent(intent: Intent) {
        super.onNewIntent(intent)
        handleIntent(intent)
    }

    private fun handleIntent(intent: Intent) {
        val action = intent.action
        val data: Uri? = intent.data

        if (Intent.ACTION_VIEW == action && data != null) {
            val path = data.path
            // Parse the URI and navigate to the appropriate content in your app
            when (path) {
                "/path/to/content" -> {
                    // Navigate to the specific content
                    Log.i("MainActivity", "Deep link received with path: $path")
                }
                else -> {
                    // Handle other paths
                    Log.i("MainActivity", "Unknown path: $path")
                }
            }
        }
    }
}
```

## 3. Launch Your App via Deep Link from Another Application

In the other application, create an `Intent` that uses a deep link URI to launch your application.

Example code from other application:
```kotlin
package com.example.otherapp

import android.content.Intent
import android.net.Uri
import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Create an intent with the deep link URI
        val deepLinkUri = Uri.parse("example://content/path/to/content")
        val intent = Intent(Intent.ACTION_VIEW, deepLinkUri)

        // Check if there is an app that can handle this intent
        val packageManager = packageManager
        val resolveInfo = packageManager.resolveActivity(intent, 0)
        if (resolveInfo != null) {
            startActivity(intent)
        } else {
            // Handle the case where no app can handle the intent
            // Optionally, redirect to the Play Store or show an error message
        }
    }
}
```

# Summary

1. **Configure Deep Linking in Your App**:
    - Set up the `intent-filter` in your app's `AndroidManifest.xml` to handle the specific deep link URI.

1. **Handle the Deep Link in Your Activity**:
    - Implement logic in your activity to parse and respond to the deep link.

1. **Launch Your App via Deep Link from Another App**:    
    - Create an intent with the deep link URI and use it to launch your app from another application.

# Full example: GitHub Link!
[Service-Client Deep Linking Example](https://github.com/RamziJabali/aidl-client-service-demo-android)