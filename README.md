# AllUnite Native Android SDK

What it is and why you need it?
===============================

AllUnite offers a technological platform to measure the effect of various marketing channels to visits in local stores. 
The measurement is done via anonymous consumers who have accepted permission.
AllUnite measure the anonymous consumer in the local store using AllUnite WiFi and iBeacon technology.

The purpose of this document is to describe how to integrate AllUnite SDK in an app.

AllUnite SDK designed to match consumer's web profile to their physical device and to track
consumers in the physical world.

![Overal schema](https://s3-eu-west-1.amazonaws.com/allunite-main/doc/scheme_v3.2.png)

How it works (workflow)
===============================

##### 1. Request consumer's permission (**REQUIRED**)
Tracking technology requires location permission that should be requested from a consumer.
AllUnite SDK neither does device matching nor tracks the device in case location permission is not provided.

You can enable or disable SDK explicitly using application URI (deep link) <app_schema>://allunite-sdk-mode?enable=[true/false].
To make it possible you would need to use AllUniteSdk.parseIncomingIntent() method as it described in "Add code for intercepting users choise at web Terms&Conditions using deeplink" below.

It can be useful if Application shows Terms & Conditions page that has "OK" and "No Thanks" options.
In this case "No Thanks" button opens <app_schema>://allunite-sdk-mode?enable=false that disables AllUnite SDK functionality, 
"OK" button can starts matching process directly in the browser (we need to customize it first) or using AllUniteSdk.bindDevice() method (see next paragraph).

##### 2. Match phisycal device (**REQUIRED**)
To match the device we need to open web browser and do a few redirects to link consumer's cookies with device ID.
Finally consumer's journey ends on a landing page that tries to open Application at once or makes it possible to open Application later.

Method AllUniteSdk.bindDevice() performs matching process described above.

##### 3. Start beacon tracking (**REQUIRED**)
To start tracking you need to call AllUniteSdk.addDidFindBeaconListener() method.
AllUnite SDK will track device in foreground and background until AllUniteSdk.removeDidFindBeaconListener() method is called.
You can override didFindBeacon(String id, Integer major, Integer minor) that will be called each time when beacon detected (see "Beacons listening" below).
It can be used to show a message or do any other actions when consumer enters an area of the beacon vicinity.

##### 4. Custom actions tracking (**OPTIONAL**)
You can register any custom actions using AllUniteSdk.track() method and use it next to location tracking info on the AllUnite Campaign Dashboard.

Android SDK - Quick Start Guide
===============================

#### 1. Add dependencies to *build.gradle*
```
    allprojects {
    repositories {
        maven { url "https://jitpack.io" }
    }
    }
    
    apply plugin: 'com.android.application'

    dependencies {
        compile 'com.github.allunite:mobile-unity-sdk:1.2.19'
        compile('com.bluecats:bluecats-android-sdk:2.0.7', {
        	exclude group: 'com.google.code.gson', module: 'gson'
	        exclude group: 'com.android.support', module: 'support-compat'
        	exclude group: 'com.android.support', module: 'support-core-utils'
    })
    }

    android {
        compileSdkVersion 22
        buildToolsVersion "22.0.1"

        defaultConfig {
            applicationId "com.allunite.example.myapplication"
            minSdkVersion 15
            targetSdkVersion 22
            versionCode 1
            versionName "1.0"
        }
    }
```
If you want to use your own dependencies on Gson, Retrofit or OkHttp logging interceptor with another version, exclude the transitive dependencies like this:
```
    compile('com.github.allunite:mobile-unity-sdk:1.1.2', {
        exclude group: 'com.google.code.gson', module: 'gson'
        exclude group: 'com.squareup.retrofit2', module: 'retrofit'
        exclude group: 'com.squareup.retrofit2', module: 'converter-gson'
        exclude group: 'com.squareup.okhttp3', module: 'logging-interceptor'
    })
```

#### 2. Add permissions to *src/main/AndroidManifest.xml*
```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="com.allunite.example.myapplication">
	<uses-permission android:name="android.permission.INTERNET" />
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
	<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
	<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

	<uses-permission android:name="android.permission.BLUETOOTH" />
	<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
	<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />

	<application android:allowBackup="true" android:icon="@mipmap/ic_launcher" android:label="@string/app_name" android:theme="@style/AppTheme" >
	<receiver android:name="com.allunite.sdk.startup.StartupBroadcastReceiver">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
                <action android:name="android.intent.action.ACTION_POWER_CONNECTED" />
                <action android:name="android.intent.action.ACTION_POWER_DISCONNECTED" />
                <action android:name="com.allunite.sdk.startup.start" />
            </intent-filter>
        </receiver>

        <service
            android:name="com.bluecats.sdk.BlueCatsSDKService"
            android:enabled="true" />
        <service android:name=".service.BCService" />
</manifest>
```
And add two <meta-data> entries with your app credentials:
```
        <meta-data
            android:name="AllUniteId"
            android:value="YOUR_ACCOUNT_ID" />
        <meta-data
            android:name="AllUniteKey"
            android:value="YOUR_ACCOUNT_KEY" />
```
 

#### 3. Add AllUnite to your main activity *src/main/java/com/allunite/example/myapplication/MainActivity.java*

On Android, the SDK's app activation helper should be invoked once when the application is created, so in your Application class's onCreate method, place the following:

```
    package com.allunite.example.myapplication;

    import android.os.Bundle;
    import android.support.v7.app.ActionBarActivity;

    import com.allunite.sdk.AllUniteSdk;

    public class MainActivity extends AppCompatActivity {

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main
	    
	    AllUniteSdk.init(AllUniteApplication.this);
        }
    }
```
 

Make sure that you've replaced "YOUR\_ACCOUNT\_ID" and "YOUR\_ACCOUNT\_KEY" with your customer account and key and run your app.

If you need to intercept AllUniteSDK init result you can to use method init(final Context context, IActionListener listener). For example:

```
AllUniteSdk.init(MainActivity.this, new IActionListener() {
                @Override
                public void onStart() {
                    
                }

                @Override
                public void onSuccess() {

                }

                @Override
                public void onError(Throwable throwable) {

                }
            });
```

#### 4. Add code to bind device to customer's profile

```
	...

	final SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);
	boolean isAgreed = sharedPreferences.getBoolean("agreed", false);
	if (!isAgreed) {
		ModalResult result = showEula();
		if (result == ModalResult.Accepted)
			AllUniteSdk.bindDevice();
	}
```
<span style="color: rgb(76,85,90);">Now this device is bound to customer's online analytics data collected from web browser on the device.</span>

#### 5. Add code for intercepting users choise at web Terms&Conditions using deeplink
At first you need to know your Uri scheme for deeplinking. Next you need to choose Activity where user will be redirecred from web page.
Add intent filter for this activity to AndroidManifest.xml:
```
	<intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />

                <data
                    android:host="allunite-sdk-mode"
                    android:scheme="(your Uri scheme)" />
        </intent-filter>
```
Add code to your Activity onCreate:
```
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
       	...
        AllUniteSdk.parseIncomingIntent(this, getIntent(), (your Uri scheme));
	...
    }
```

#### 6. SDK enabling
If you need to enable AllUniteSDK programmatically just use AllUniteSdk.setSdkEnabled(Context context, boolean enabled) method.

#### 7. Add code to track your first event
```
	...

	String actionCategory = "showCampaign";
	String actionId = "blackFriday2016";
	AllUniteSdk.track(actionCategory, actionId);
```
<span style="color: rgb(76,85,90);">You should now see the action "blackFriday2016" on this device in "showCampaign" category of your AllUnite reports.</span>

#### 8. Location permissions
For Beacons tracking you need to add ACCESS_COARSE_LOCATION permission to your app.
If your app uses targetSdkVersion 23 or above you need to implement [runtime location permissions requesting](https://developer.android.com/training/permissions/requesting.html).

#### 9. Beacons listening
If you want to intercept beacons by yourself you need to subscribe to AllUniteSDK using callback interface com.allunite.sdk.callbacks.IDidFindBeaconListener:

```
public class MainActivity extends AppCompatActivity implements IDidFindBeaconListener {
    ...
    @Override
    public void didFindBeacon(String id, Integer major, Integer minor) {
      ...
    }
    ...
    @Override
    protected void onResume() {
        super.onResume();
        AllUniteSdk.addDidFindBeaconListener(this);
    }

    @Override
    protected void onPause() {
        super.onPause();
        AllUniteSdk.removeDidFindBeaconListener(this);
    }
}
```

#### 10. Permission flow

Track when user accept location permission using AllUniteSdk.sendLocationPermissionsGranted(Context) at Runtime Permissions callback.

Example:
```
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);

        if (requestCode == REQUEST_LOCATION_PERMISSION
                && grantResults.length > 0
                && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            AllUniteSdk.sendLocationPermissionsGranted(this);
        }
    }
```

#### 11. Track current device status.
```
AllUniteSdk.trackDeviceStatus(context);
```

To track device status on receiving push notification by Google or Firebase Cloud Messages please put it into callback method:

```
public class MyGcmListenerService extends com.google.android.gms.gcm.GcmListenerService {

    ...

	@Override
	public void onMessageReceived(String from, Bundle data) {
	        // some code
	        AllUniteSdk.trackDeviceStatus(getApplication());
		}
	}
```

#### 12. That's it! AllUnite is now integrated with your app.

In order to ensure our library doesn't have an impact on user experience, we send events asynchronously.
