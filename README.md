## Cordova Plugin for Responsys SDK (Capacitor Compatible)

This plugin makes it easy to integrate your Cordova based mobile app with the Responsys SDK. 

### Table of Contents
- [Requirements](#requirements)
- [Setup](#setup)
  * [For Android](#for-android-1)
  * [For iOS](#for-ios-1)
- [Installation](#installation)
  * [Capacitor Push Plugin](#capacitor-push-plugin)
  * [Responsys Plugin](#responsys-plugin)
- [Integration](#integration)
  * [For Android](#for-android-2)
  * [For iOS](#for-ios-2)
- [Usage](#usage)
  * [Configure And Register](#configure-and-register)
  * [User Identification](#user-identification)
  * [Engagements And Conversion](#engagements-and-conversion)
  * [In-App Messages](#in-app-messages)
  * [Message Center](#message-center)
  * [Geofences And Beacons](#geofences-and-beacons)
  * [Notification Preferences](#notification-preferences)
- [Support](#support)
- [License](#license)

<br/>

<div id='requirements'/>
### Requirements

- Capacitor >= 3.5.1
- Ionic CLI >= 6.7.0
- Ionic Framework >= 5.0.5 (@ionic/angular)
- Android SDK Tools >= 26.1.1
- iOS 12 or later

<br/>

<div id='setup'/>
### Setup

Before installing the plugin, you must setup your app to receive push notifications.

<div id='for-android-1'/>
#### For Android 
- [Get FCM Credentials](https://docs.oracle.com/en/cloud/saas/marketing/responsys-develop-mobile/android/gcm-credentials) 
- Log in to the [Responsys Mobile App Developer Console](https://docs.oracle.com/en/cloud/saas/marketing/responsys-develop-mobile/dev-console/login/) and enter your FCM credentials (Project ID and Server API Key) for your Android app.
- Download the `pushio_config.json` file generated from your credentials and include it in your project's `android/src/main/assets` folder.
- Create a new directory  - `PushIOManager` inside your app's `android` directory.
- Download the SDK native binary from [here](https://www.oracle.com/downloads/applications/cx/responsys-mobile-sdk.html) and place the .aar file in this `android/PushIOManager/` directory.
- Inside the `android/PushIOManager` directory, create a file `build.gradle` with the following code:

	```gradle
	configurations.maybeCreate("default")
	artifacts.add("default", file('PushIOManager-6.52.aar'))
	```		

- Add the following to your project-wide `settings.gradle` file:

	```gradle
	include ':PushIOManager'
	project(':PushIOManager').projectDir = new File(rootProject.projectDir, './PushIOManager')
	```

<br/>

<div id='for-ios-1'/>
#### For iOS
- [Generate Auth Key](https://docs.oracle.com/en/cloud/saas/marketing/responsys-develop-mobile/ios/auth-key/) 
- Log in to the [Responsys Mobile App Developer Console](https://docs.oracle.com/en/cloud/saas/marketing/responsys-develop-mobile/dev-console/login/) and enter your Auth Key and other details for your iOS app.
- Download the `pushio_config.json` file generated from your credentials.
- Open the Xcode project workspace in your `platforms/ios` directory of cordova app. 
- Drag and Drop your `pushio_config.json` in Xcode project.
- Select the root project and Under Capabilites add the "Push Notifications" and "Background Modes". 
![Capabilty Image](./img/ios_add_capability.png "Capabilty Image")

<br/>

<div id='installation'/>
### Installation

<div id='capacitor-push-plugin'/>
#### Capacitor Push Plugin

- Follow the steps [here](https://capacitorjs.com/docs/apis/push-notifications) to install the Capacitor push plugin. This is necessary for correctly handling the incoming push notifications.

- Include following code in `capacitor.config.ts`,

	```shell
	import { CapacitorConfig } from '@capacitor/cli';
	
	const config: CapacitorConfig = {
	  cordova: {
	    preferences: {
	      'supportCapPushPlugin':'true'
	    }
	  }
	};
	
	export default
	```

<br/>

<div id='responsys-plugin'/>
#### Responsys Plugin

- Clone the plugin outside your app directory,

	```shell
	git clone -b capacitor https://github.com/oracle/pushiomanager-cordova-plugin.git
	```
	
- Create `frameworks` folder inside the plugin and place the [latest iOS PushIOManager.framework](https://www.oracle.com/downloads/applications/cx/responsys-mobile-sdk.html) inside `PATH_TO_cordova-plugin-pushiomanager_DIRECTORY/frameworks/` folder. 

After above these steps your framework directory should look like this.

![FrameworkCopy](./img/framework_copy.png "Framework Copy")

The plugin can be installed with the following command

```shell
npm install PATH_TO_pushiomanager-cordova-plugin_DIRECTORY 
```

then perform sync using with the following command

```shell
npx cap sync
```

or Ionic command:

```shell
ionic cap sync 
ionic cap sync <platform>
```

<br/>


<div id='integration'/>
### Integration

<div id='for-android-2'/>
#### For Android

- Open the `AndroidManifest.xml` file located at `android/src/main` and add the following,
	* Permissions above the `<application>` tag,

		```xml
		<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
		<uses-permission android:name="${applicationId}.permission.PUSHIO_MESSAGE" />
		<uses-permission android:name="${applicationId}.permission.RSYS_SHOW_IAM" />
		<permission android:name=".permission.PUSHIO_MESSAGE" android:protectionLevel="signature" />
		<permission android:name="${applicationId}.permission.RSYS_SHOW_IAM" android:protectionLevel="signature" />
		```
	
	* Intent-filter for launching app when the user taps on a push notification. Add it inside the `<activity>` tag of `MainActivity`,

		```xml
		<intent-filter>
			<action android:name="${applicationId}.NOTIFICATIONPRESSED" />
	   		<category android:name="android.intent.category.DEFAULT" />
		</intent-filter>
		```

	* (Optional) Intent-filter for [Android App Links](https://developer.android.com/training/app-links) setup. Add it inside the `<activity>` tag of `MainActivity`,

		```xml
		<intent-filter android:autoVerify="true">
			<action android:name="android.intent.action.VIEW" />
			<category android:name="android.intent.category.DEFAULT" />
			<category android:name="android.intent.category.BROWSABLE" />
			<data android:scheme="https" 
			      android:pathPrefix="/pub/acc"
			      android:host="@string/app_links_url_host" />
       </intent-filter>
		```
		
	* Add the following code inside `<application>` tag,

		```xml
		 <receiver android:enabled="true" 
		 			 android:exported="false" 
		 			 android:name="com.pushio.manager.PushIOUriReceiver" 
		 			 tools:node="replace">
		 			 
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <data android:scheme="@string/uri_identifier" />
            </intent-filter>
            
        </receiver>
        
        <activity android:name="com.pushio.manager.iam.ui.PushIOMessageViewActivity" 
        			  android:permission="${applicationId}.permission.SHOW_IAM" 
        			  android:theme="@android:style/Theme.Translucent.NoTitleBar">
            
            <intent-filter tools:ignore="AppLinkUrlError">
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.BROWSABLE" />
                <category android:name="android.intent.category.DEFAULT" />
                <data android:scheme="@string/uri_identifier" />
            </intent-filter>
            
        </activity>
        
       <service android:name=".MyFirebaseMessagingService">
		    <intent-filter>
		        <action android:name="com.google.firebase.MESSAGING_EVENT" />
		    </intent-filter>
		</service>
		```
		

- Open the `strings.xml` file located at `android/src/main/res/values` and add the following properties,

	* Custom URI scheme for displaying In-App Messages and Rich Push content,

		```xml
		<string name="uri_identifier">pio-YOUR_API_KEY</string>
		```
		You can find the API key in the `pushio_config.json` that was placed in `android/app/src/main/assets` earlier during setup.
		
	* (Optional) If you added the `<intent-filter>` for Android App Links in the steps above, then you will need to declare the domain name,
	
		```xml
		<string name="app_links_url_host">YOUR_ANDROID_APP_LINKS_DOMAIN</string>
		```

- Create a native Intent Service - `MyFirebaseMessagingService.java` in `android/app/src/main/java/<your_package_name>`, as follows,

	```java
	public class PIOFirebaseMessagingService extends FirebaseMessagingService {
	    @Override
	    public void onNewToken(String token) {
	    	super.onNewToken(token);
	        
	      	PushNotificationsPlugin.onNewToken(token);
	        
	      	PushIOManager.getInstance(getApplicationContext()).setDeviceToken(token);
	    }
	
	    @Override
	    public void onMessageReceived(@NonNull RemoteMessage remoteMessage) {
	        super.onMessageReceived(remoteMessage);
	        
	        PushIOManager pushIOManager = PushIOManager.getInstance(getApplicationContext());
	        
	        if (pushIOManager.isResponsysPush(remoteMessage)) {
	            
	            pushIOManager.handleMessage(remoteMessage);
	        
	        }else{
	        
	            PushNotificationsPlugin.sendRemoteMessage(remoteMessage);
	           
	        }
	    }
	}
	```
	
- Intercept incoming push notifications in your app (.ts) and redirect to any other push plugin (if any),

	```javascript
	constructor() {
		PushNotifications.register();
	  	PushNotifications.addListener('pushNotificationReceived',
	      (notification: PushNotificationSchema) => {
	      
	        alert("Push received: " + JSON.stringify(notification));
	        
	      }
	    );
    }
	```

<br/>

<div id='for-ios-2'/>
#### For iOS

- Add `-ObjC` linker flag to Cordova Plugin target in pods project.

	![linkerflag Image](./img/otherlinerflag.png "linkerflag Image")

- Add the following two methods in AppDelegate to handle the APNS device token registration event

	```shell
	func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
	  	  NotificationCenter.default.post(name: .capacitorDidRegisterForRemoteNotifications, object: token)
	}
	
	func application(_ application: UIApplication, didFailToRegisterForRemoteNotificationsWithError error: Error) {
	      NotificationCenter.default.post(name: .capacitorDidFailToRegisterForRemoteNotifications, object: error)
	}
	```
	
- In your app, Add the following in the AppDelegate file in App xcode project

	```shell
	import Capacitor
	
	func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool   
	       addPushNotificationListner()
	        return true
	    }
	
	public func addPushNotificationListner() {
	       
	       let callback =  { (response:[String:Any]?, error:NSError?) -> Void in
	           
	           if let response = response {
	               NotificationCenter.default.post(name: NSNotification.Name(rawValue: "PushIODidReceiveNotification"),object: response)
	           }
	           
	       }
	        
	       addListener(name: "pushNotificationReceived",closure: callback)
	       
	       addListener(name: "pushNotificationActionPerformed",closure: callback)
	 }
	    
	public func addListener(name:String,closure:@escaping([String:Any]?, NSError?)->Void) {
	        
	        DispatchQueue.main.async {
	            if let rootViewController:CAPBridgeViewController =   self.window?.rootViewController as? CAPBridgeViewController {
	                
	                if let plugin:CAPPlugin = rootViewController.bridge?.plugin(withName: "PushNotifications") {
	                    
	                    if let call  =  CAPPluginCall(callbackId: name, success: { result, c in
	                        closure(result?.data,nil);
	                    }, error: { error in
	                        closure(nil,error as? NSError);
	                    }) {
	                        let pcall =  call;
	                        pcall.options = JSTypes.coerceDictionaryToJSObject(call.options)
	                    plugin.addEventListener(name, listener: call)
	                  }
	                }
	            }
	        }
	 }
	```

- Run capacitor sync command. Now App is ready to build and run.

	```shell
	ionic cap sync
	```


- For In-App Messages and Rich Push Content follow the below steps :
  * To Enable Custom URI scheme for displaying In-App Messages and Rich Push content follow the [Step 1](https://docs.oracle.com/en/cloud/saas/marketing/responsys-develop-mobile/ios/in-app-msg/). You don't need to add the code.
  You can find the API key in the `pushio_config.json` that was placed in your Xcode project earlier during setup.
  
  * Follow  [Step 2](https://docs.oracle.com/en/cloud/saas/marketing/responsys-develop-mobile/ios/in-app-msg/) to  add the required capabilites in your Xcode project for In-App messages. You don't need to add the code.

- For Media Attachments you can follow the following [guide](https://docs.oracle.com/en/cloud/saas/marketing/responsys-develop-mobile/ios/media-attachments/). Copy and paste the code provided in guide in respective files.	

- For Carousel Push you can follow the following [guide](https://docs.oracle.com/en/cloud/saas/marketing/responsys-develop-mobile/ios/carousel-push/). Copy and paste the code provided in guide in respective files.    

<br/>

<div id='usage'/>
### Usage

The Responsys plugin can be accessed in JS code using the namespace `cordova.plugins.PushIOManager`,

```javascript
import { Platform } from '@ionic/angular';

declare let cordova: any;
declare let PushIOManager: any;

export class MyPage {

  constructor(
    public platform: Platform
  ) {
  
    this.platform.ready().then(() => {
    
      PushIOManager = cordova.plugins.PushIOManager;

      PushIOManager.setLogLevel(PushIOManager.logLevel.VERBOSE);
    }
}
```

<div id='configure-and-register'/>
#### Configure And Register

- Configure the SDK,

	```javascript
	PushIOManager.configure("pushio_config.json", (success) => {
	      
	}, (error) => {
	      
	});
	```
	
- Once the SDK is configured, register the app with Responsys,

	```javascript
	PushIOManager.registerApp(true, (success) => {
	
	}, (error) => {     
	
	});
	```
	
<div id='user-identification'/>
#### User Identification

- Associate an app installation with a user (usually after login),

	```javascript
	PushIOManager.registerUserId("xyz@yxz.zyx", (success) => {
	      
	}, (error) => {
	    
	});
	```
	
- When the user logs out,

	```javascript
	PushIOManager.unregisterUserId((success) => {
	      
	}, (error) => {
	    
	});
	```
<br/>
	
<div id='engagements-and-conversion'/>
#### Engagements And Conversion

User actions can be attributed to a push notification using,

```javascript
PushIOManager.trackEngagement(PushIOManager.engagementType.PUSHIO_ENGAGEMENT_METRIC_INAPP_PURCHASE,
(success) => {
	      
}, (error) => {
	    
});
```

<br/>

<div id='in-app-messages'/>
#### In-App Messages

In-App Message (IAM) are displayed in a popup window via system-defined triggers like `$ExplicitAppOpen` or custom triggers. IAM that use system-defined triggers are displayed automatically.

IAM can also be displayed on-demand using custom triggers.

- Your marketing team defines a custom trigger in Responsys system and shares the trigger-event name with you.
- Marketer launches the campaign and the IAM is delivered to the device via push or pull mechanism (depending on your Responsys Account settings)
- When you wish to display the IAM popup, use,

	```javascript
	PushIOManager.trackEvent(custom_event_name, properties, (success) => {
	      
	}, (error) => {
	      
	});
	```
<br/>

<div id='message-center'/>
#### Message Center

- Get the Message Center messages list using,

	```javascript
	PushIOManager.fetchMessagesForMessageCenter("Primary", (response) => {
	
	}, (error) => {
	      
	});
	```
	
- If any message has a rich-content (HTML) then call,

	```javascript
	PushIOManager.fetchRichContentForMessage(messageID, (response) => {
	      // `response` is the HTML content
	}, (error) => {
	      
	});
	```
	
	Remember to store these messages, since the SDK cache is purgeable.
	
<br/>

<div id='geofences-and-beacons'/>
#### Geofences And Beacons

If your app is setup to monitor geofence and beacons, you can use the following APIs to record in Responsys when a user enters/exits a geofence/beacon zone.

```javascript
PushIOManager.onGeoRegionEntered(geoRegion, (response) => {}, (error) => {});
PushIOManager.onGeoRegionExited(geoRegion, (response) => {}, (error) => {});
PushIOManager.onBeaconRegionEntered(beaconRegion, (response) => {}, (error) => {});
PushIOManager.onBeaconRegionExited(beaconRegion, (response) => {}, (error) => {});
```

<br/>

<div id='notification-preferences'/>
#### Notification Preferences

Preferences are used to record user-choices for push notifications. The preferences should be [pre-defined in Responsys](https://docs.oracle.com/en/cloud/saas/marketing/responsys-develop-mobile/dev-console/app-design/#notification-preferences) before being used in your app.

- Declare the preference beforehand in the app,

	```javascript
	PushIOManager.declarePreference(key, label, preferenceType, (response) => {
	
	}, (error) => {
	      
	});
	```

- Once a preference is declared successfully, you may save the preference using,

	```javascript
	PushIOManager.setPreference(key, value, (response) => {
	
	}, (error) => {
	      
	});
	```
	
Do not use this as a key/value store as this data is purgeable.

<br/>

<div id='support'/>
### Support

If you have access to My Oracle Support, please raise a request [here](http://support.oracle.com/), otherwise open an issue in this repository. 


<br/>

<div id='license'/>
### License

Copyright (c) 2022 Oracle and/or its affiliates and released under the Universal Permissive License (UPL), Version 1.0.

Oracle and Java are registered trademarks of Oracle and/or its affiliates. Other names may be trademarks of their respective owners.
