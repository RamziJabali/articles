---
tags:
  - android
  - aidl
  - article
  - kotlin
date: 2024-07-12
title: AIDL Guide
---
# What is AIDL?

AIDL stands for Android Interface Definition Language. As the name says it's an interface. It requires the using the Java Programming Language.

It is also requires a service to be able to use between applications.

# What does it do?

AIDL allows for IPC(InterProcess Communication). Allowing for code to be accessible between different process, even different applications.

# How do you create an AIDL file

1. In Android Studio you would want to hover to your `app` package and right click and go into New.
	1. You should be able to see a variety of different options, you will choose to AIDL.
	2. If it is not clickable
		1. Go to `build.gradle` on the module level and add `aidl = true`
			```
			buildFeatures {  
		    compose = true  aidl = true  
		    }
			```
		3. Now you should have a AIDL file created!
	3. You will have to build your application to generate the Java files that you will be using to create the `Stub`, a type of `IBinder`, that you will use when you call `onBind()` from client!
# How do I use my AIDL Interface from a different application

Steps:
1. Implement AIDL interface within a service class
	1. Add logic
2. Update Service Side Manifest 
	1. Permission
	2. Uses-Permission
	3. Service
3. Add the same exact AIDL file into your client application
	1. Same package name and file contents
4. Update Client Side Manifest 
5. Add `ServiceConnection` code to be able establish a connection with AIDL

## 1. Implement AIDL Interface within a service class
### AIDL file
```Java
// IMyAidlInterface.aidl  
package ramzi.eljabali.aidlmockapp;  
  
// Declare any non-default types here with import statements  
  
interface IMyAidlInterface {  
    /**  
     * Demonstrates some basic types that you can use as parameters     * and return values in AIDL.     */    
    void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,  
            double aDouble, String aString);  
  
    int add(int x, int y);  
  
    String greet(String name);  
}
```

### Service implementing AIDL
```Kotlin
package ramzi.eljabali.aidlmockapp  
  
import android.app.Service  
import android.content.Intent  
import android.os.IBinder  
import android.util.Log  
  
class MyAidlService : Service() {  
  
    override fun onBind(p0: Intent?): IBinder {  
        return binder  
    }  
  
    private val binder = object : IMyAidlInterface.Stub() {  
        override fun basicTypes(  
            anInt: Int,  
            aLong: Long,  
            aBoolean: Boolean,  
            aFloat: Float,  
            aDouble: Double,  
            aString: String?  
        ) {  
            Log.i(  
                "IMyAidlInterface -> basicTypes()",  
                "AIDL supports the following types: Int, Long, Boolean, Float, Double, String."  
            )  
        }  
  
        override fun add(x: Int, y: Int): Int {  
			Log.i("IMyAidlInterface -> add()", "x value $x, y value $y, total ${x + y}")  
            return x + y  
        }  
  
        override fun greet(name: String?): String {  
            Log.i("IMyAidlInterface -> greet()", "hello $name, welcome to AIDL!")  
            return "Hello $name, welcome to AIDL!"  
        }  
  
    }  
  
}
```

## 2. Update Service Side Manifest  

### What do you need to add?

- `<permission>` tag with the following properties
	1. `android:name=`
	2. `android:protectionLevel=
-  `<uses-permission>` tag with the following properties
	- `android:name=`
- `<service>` tag with the following properties
	- `android:name=`
	- `android:exported=`
	- `android:permission=`
	- `<intent-filter>`

1. A `<permission>` tag is used when you need to declare a security permission.
	- This security permission is what is used to limit what components or features applications can access.
	- Within the `<permission>` tag we need to define the name(`android:name`) of this permission and the protection level(`android:protectionLevel`) for our application.
	- `android:name=ramzi.eljabali.aidlmockapp.permission.BIND_MY_AIDL_SERVICE`
		- In our case our permission will be a custom.
			- I chose to go with the package name and the type of feature I am wanting to share.
	- `android:protectionLevel:signature`
		- `signature` means that the system will only grant this permission if the requesting application is signed with same certificate as the application that declared the permission.
2. A `<uses-permission>` tag specifies a system permission that you must grant for your application to operate correctly. This is granted upon installing your application(Android 5.1 and lower), or upon running your application(Android 6.0 and higher).
	- `android:name` property is used here to specify the permission name.
		- Could be defined within `<permission>` element, or a permission defined by another application, or just regular system permissions like `"android.permission.CAMERA"`
3. A `<service>` tag is used to declare a `Service` subclass, as one of your applications components. Any service you have created within your application needs to be defined within your manifest using the `<service>`tag otherwise, it will not be seen by the system and ultimately will not run.
	- `android:name` property is used to define which `Service` subclass we are using within our application.
		- In this case it's just `MyAidlService`
		- `android:name=".MyAidlService"`
	- `android:exported` property is used to define whether components of other applications are allowed to use our service or interact with it.
		- In our case we do want other applications to be able to use our service so we will set `android:exported=true`, otherwise we would have set it to `false`
	- `android:permission` property is used to define the name of a permission that an entity needs in order to launch the service or bind to it. 
		- If a caller of `startService()`, `bindService()`, or `stopService()` hasn't granted this permission, the method doesn't work and the `Intent` object isn't delivered to the service. Client will need this permission in their manifest to be able to call on our service.
		- `android:permission="ramzi.eljabali.aidlmockapp.permission.BIND_MY_AIDL_SERVICE"`
	- An `<intent-filter>` is required to be able to start this service. Because the `intent-filter` specifies the type of `Intent`s that an `Activity`, `Service`, or `Broadcast Reciever` can respond to.
		- ```<intent-filter>
		    <action android:name="ramzi.eljabali.aidlmockapp.MyAidlService" />  
	    </intent-filter>```

Service Side AndroidManifest.xml
```XML AndroidManifest.xml
<permission  
    android:name="ramzi.eljabali.aidlmockapp.permission.BIND_MY_AIDL_SERVICE"  
    android:protectionLevel="signature" />

<uses-permission android:name="ramzi.eljabali.aidlmockapp.permission.BIND_MY_AIDL_SERVICE" />

<service  
    android:name=".MyAidlService"  
    android:exported="true"  
android:permission="ramzi.eljabali.aidlmockapp.permission.BIND_MY_AIDL_SERVICE"> 
	    <intent-filter>
		    <action android:name="ramzi.eljabali.aidlmockapp.MyAidlService" />  
	    </intent-filter>
    </service>
    
```

## 3. Add the same exact AIDL file into your client application

- This acts as a contract between the applications that want to use this service.

In Android Studio you would want to hover to your `app` package and right click and go into New.
	1. You should be able to see a variety of different options, you will choose to AIDL.
	2. If it is not clickable
		1. Go to `build.gradle` on the module level and add `aidl = true`
			```
			buildFeatures {  
		    compose = true  aidl = true  
		    }
			```
		3. Now you should have a AIDL file created!
	3. You will refactor the aidl package name to match the service side aidl package name.
		1. In my case it was `ramzi.eljabali.cientapplication` changed to `ramzi.eljabali.aidlmockapp` to match with service side.
	4. You will be build to generate the Java files necessary to be able to use the service.

## 4. Update Client Side Manifest 
### What do you need to add?
1. `<uses-permission>`
	- As previously discussed `<uses-permission>` tag is required to specify the system permission we need to grant for our application to work.
	- `android:name` is the name of the permission we are wanting to grant.

```XML 
<uses-permission android:name="ramzi.eljabali.aidlmockapp.permission.BIND_MY_AIDL_SERVICE" />
```

## 5. Connect to the Service

1. Implement the `ServiceConnection` interface within your code to be able to know when you have connected to your service and when you have abruptly lost connection to your service.

Here's how implementing the `ServiceConnection` interface would look like:
```kotlin
private var iMyAidlInterface: IMyAidlInterface? = null  
    private var isBound = false  
  
    private val mConnection = object : ServiceConnection {  
        override fun onServiceConnected(className: ComponentName, service: IBinder) {  
            Log.i("Service-MainActivity", "Connection to service established")  
            iMyAidlInterface = IMyAidlInterface.Stub.asInterface(service)  
            isBound = true  
        }  
  
        override fun onServiceDisconnected(className: ComponentName) {  
            Log.e("Service-MainActivity", "Service has unexpectedly disconnected")  
            iMyAidlInterface = null  
            isBound = false  
        }  
    }
```

2. You want to start the service and then bind to it.
	1. To start the service you will need to create an `Intent` 
		- We previously discussed the use of the `<intent-filter>` within the service side `AndroidManifest.xml`
		- Now the `action` attribute within the `Intent` we are creating needs to be that of the `intent-filter` we previously defined on the service side.
		- ```<intent-filter>  
		    <action android:name="ramzi.eljabali.aidlmockapp.MyAidlService" />  
		 </intent-filter>```
		- `val intent = Intent("ramzi.eljabali.aidlmockapp.MyAidlService")`
		- We then want to set the package name to the package name of the service.
		- `intent.setPackage("ramzi.eljabali.aidlmockapp")`

```Kotlin
val intent = Intent("ramzi.eljabali.aidlmockapp.MyAidlService")  
intent.setPackage("ramzi.eljabali.aidlmockapp")
```

1. Now that we have created the `Intent` we can start the service.
		- `startService(intent)` 
	2. After Starting the service we want to bind to it.
		- `bindService(intent, mConnection, BIND_AUTO_CREATE)`
			- `BIND_AUTO_CREATE` is an `Int` flag that specifies that it will automatically create the service as long as the binding exists.

```Kotlin
startService(intent)  
  
val isBinding = bindService(intent, mConnection, BIND_AUTO_CREATE) 

Log.i("Service-MainActivity", "Binding service: $isBinding")  
Log.i("Service-MainActivity", "Is bound to service: $isBound")
```


Putting it all together we get:

```kotlin
class MainActivity : ComponentActivity() {  
    private var iMyAidlInterface: IMyAidlInterface? = null  
    private var isBound = false  
  
    private val mConnection = object : ServiceConnection {  
        override fun onServiceConnected(className: ComponentName, service: IBinder) {  
            Log.i("Service-MainActivity", "Connection to service established")  
            iMyAidlInterface = IMyAidlInterface.Stub.asInterface(service)  
            isBound = true  
        }  
  
        override fun onServiceDisconnected(className: ComponentName) {  
            Log.e("Service-MainActivity", "Service has unexpectedly disconnected")  
            iMyAidlInterface = null  
            isBound = false  
        }  
    }  
  
    override fun onCreate(savedInstanceState: Bundle?) {  
        super.onCreate(savedInstanceState)  
        enableEdgeToEdge()  
  
        val intent = Intent("ramzi.eljabali.aidlmockapp.MyAidlService")  
        intent.setPackage("ramzi.eljabali.aidlmockapp")  
  
        startService(intent)  
  
        val isBinding = bindService(intent, mConnection, BIND_AUTO_CREATE)  
        Log.i("Service-MainActivity", "Binding service: $isBinding")  
        Log.i("Service-MainActivity", "Is bound to service: $isBound")  
  
        setContent {  
            var greetingText by remember { mutableStateOf("Hello!") }  
            var mathText by remember { mutableStateOf("") }  
            ClientApplicationTheme {  
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->  
                    innerPadding  
                    Column(  
                        Modifier.fillMaxSize(),  
                        verticalArrangement = Arrangement.Center,  
                        horizontalAlignment = Alignment.CenterHorizontally  
                    ) {  
                        Text(text = greetingText)  
                        Button(onClick = {  
                            if (isBound) {  
                                greetingText = iMyAidlInterface?.greet("Ramzi") ?: "Couldn't call service"  
                                mathText = iMyAidlInterface?.add(35, 7).toString()  
                            } else {  
                                greetingText = "Service not bound"  
                            }  
                        }) {  
                            Text("Click me")  
                        }  
                        Text(text = mathText)  
                    }  
                }            }        }    }  
  
    override fun onDestroy() {  
        super.onDestroy()  
        if (isBound) {  
            unbindService(mConnection)  
            Log.i("Service-MainActivity", "Service unbound")  
            isBound = false  
        }  
    }  
}
```

# Results:
1. You created an `AIDL` file
2. You implemented it as a service
3. Added a custom `permission` to your `AndroidManifest.xml`
4. Added a custom `uses-permission` to your `AndroidManifest.xml`
5. Defined your `service` within your `AndroidManifest.xml`
	1. Created an `intent-filter` to allow for it to be started 
6. Client side we implemented the `ServiceConnection` interface to know when we have established a connection to the service and when it's abruptly gone.
	1. `override fun onServiceConnected()`
	2. `override fun onServiceDisconnected()`
7. Client side created an `Intent` that would be used to start and bind the service
8. Client side started the service using `startService(intent)`
9. Client side was then bound to the service using `bindService(intent, mConnection, BIND_AUTO_CREATE)`

