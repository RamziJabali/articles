## Foreground Service with persistant notification

### how to start a foreground service

1. Create Foreground Service class
 
```kotlin
class ForegroundService : Service() { ... }
```

2. Add to manifest
    - You will have to Define it in the manifest
    - Your foreground service can be of a different type, mine is a location foreground service

```manifest
     <service
            android:foregroundServiceType="location"
            android:exported="false"
            android:name=".loactionservice.ForegroundService">
     </service>
```

3. Override onStartCommand function
    * You can define an expression to handle the different actions tht are permitted within your service
    * In this case I have an actions enum defined with the service `enum class Actions { START, STOP}`

```kotlin
override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        when (intent?.action) {
            Actions.START.name -> {
                Log.i("ForegroundService::Class", "Starting service")
                start()
            }
            Actions.STOP.name -> {
                Log.i("ForegroundService::Class", "Stopping service")
                stop()
            }
        }
        return super.onStartCommand(intent, flags, startId)
    }

```

4. Create notifications and notification channels
    - Notification channels can be created in a seprate class and initialized on onCreate()
    - My notification includes Pending Intents


    * The first PendingIntent(Action that will take place at a later time) makes it so if the user clicks on the notification the MainActivity::class.java is started
        * `PendingIntent.getActivity()` ~ It is used to create a PendingIntent that will start an activity when triggered.
        * This happens because we pass it an Intent `Intent(this, MainActivity::class.java)` to start the MainActivity::class
        * `PendingIntent.FLAG_IMMUTABLE` This flag indicates that the PendingIntent should be immutable, meaning that its configuration cannot be changed after it is created. 

```kotlin
private val pendingIntent: PendingIntent by lazy {
        PendingIntent.getActivity(
            this, 0, Intent(this, MainActivity::class.java),
            PendingIntent.FLAG_IMMUTABLE
        )
    }
```

* The Second PendingIntent(Action that will take place at a later time) makes it so if the user clicks on the notification action button the service is stopped
    * `PendingIntent.getService()` ~ Similar to getActivity(), it is used to create a PendingIntent, but instead of starting an activity, it starts a service when triggered. 
    * This happens because we pass it an Intent `Intent(this, ForegroundService::class.java).apply {action = Actions.STOP.name}` to start the ForegroundService::class.java
    * `PendingIntent.FLAG_IMMUTABLE` This flag indicates that the PendingIntent should be immutable, meaning that its configuration cannot be changed after it is created. 
    * Recall `onStartCommand()` contains an expression that evaluates what is passed into the intent

```kotlin
    private val stopServicePendingIntent: PendingIntent by lazy {
        PendingIntent.getService(
            this, 0, Intent(this, ForegroundService::class.java).apply {
                action = Actions.STOP.name
            },
            PendingIntent.FLAG_CANCEL_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )
    }
```

```kotlin
private fun getNotification() =
        NotificationCompat.Builder(applicationContext, CHANNEL_ID_1)
            .setSmallIcon(R.mipmap.just_jog_icon_foreground)
            .setContentTitle(ContextCompat.getString(applicationContext, R.string.just_jog))
            .setContentText(ContextCompat.getString(applicationContext, R.string.notification_text))
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setVisibility(NotificationCompat.VISIBILITY_PUBLIC)
            .setOngoing(true)
            .setAutoCancel(false)
            .setStyle(NotificationCompat.BigTextStyle())
            // Set the intent that fires when the user taps the notification.
            .setContentIntent(pendingIntent)
            .addAction(
                R.mipmap.cross_monochrome,
                getString(R.string.stop_jog),
                stopServicePendingIntent
            )
            .build()
```

5. Define your logic that is used when your service is started
    * Using a LocationManager and LocationListener
      
```kotlin
private val locationManager by lazy {
        ContextCompat.getSystemService(application, LocationManager::class.java) as LocationManager
    }

    private val locationListener: LocationListener by lazy {
        LocationListener { location ->
            Log.d(
                "ForegroundService::Class",
                "Time:${ZonedDateTime.now()}\nLatitude: ${location.latitude}, Longitude:${location.longitude}"
            )
        }
    }
```


```kotlin
 private fun stop() {
        locationManager.removeUpdates(locationListener)
        stopSelf()
    }

    private fun start() {
        Log.i("ForegroundService::Class", "Checking permissions")
        for (permission in permissions.list) {
            val currentPermission = ContextCompat.checkSelfPermission(this, permission)
            if (currentPermission == PackageManager.PERMISSION_DENIED) {
                // Without these permissions the service cannot run in the foreground
                // Consider informing user or updating your app UI if visible.
                Log.d("ForegroundService::Class", "Permissions were not given, stopping service!")
                stopSelf()
            }
        }

        try {
            ServiceCompat.startForeground(
                this,
                NOTIFICATION_ID, // Cannot be 0
                getNotification(),
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
                    ServiceInfo.FOREGROUND_SERVICE_TYPE_LOCATION
                } else {
                    0
                },
            )
            // getting user location
            locationManager.requestLocationUpdates(
                LocationManager.GPS_PROVIDER,
                LOCATION_REQUEST_INTERVAL_MS,
                0F,
                locationListener
            )
        } catch (e: Exception) {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S
                && e is ForegroundServiceStartNotAllowedException
            ) {
                // App not in a valid state to start foreground service
                // (e.g. started from bg)
                Log.d("ForegroundService::Class", e.message.toString())
            }
        }
    }
```
7. You have now defined a general skeleton for your foreground service
    * You created a means to handle intents to start your foreground service
    * You created your own notification
    * You created pending intents to go with your notification
        * Starts MainActivity::Class.java and stops the foreground service    
    * You created logic that is executed when the service is started