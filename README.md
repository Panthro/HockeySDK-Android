[![Build Status](https://travis-ci.org/bitstadium/HockeySDK-Android.svg?branch=develop)](https://travis-ci.org/bitstadium/HockeySDK-Android)
[![Slack Status](https://slack.hockeyapp.net/badge.svg)](https://slack.hockeyapp.net)

## Preseason - Version 4.1.0-beta.2

## Introduction

HockeySDK-Android implements support for using HockeyApp in your Android application.

The following features are currently supported:

1. **Crash Reports:** If your app crashes, a crash log is written to the device's storage. If the user starts the app again, they will be asked asked to submit the crash report to HockeyApp. This works for both beta and live apps, i.e. those submitted to Google Play or other app stores. Crash logs contain viable information for you to help resolve the issue. Furthermore, you as a developer can add additional information to the report as well.

2. **User Metrics:** Understand user behavior to improve your app. Track usage through daily and monthly active users, monitor crash impacted users, as well as customer engagement through session count. User Metrics requires a minimum API level of 14 (Android 4.x Ice Cream Sandwich).

3. **Update alpha/beta apps:** The app will check with HockeyApp if a new version for your alpha/beta build is available. If yes, it will show a dialog to users and let them see the release notes, the version history and start the installation process right away. You can even force the installation of certain updates.

4. **Feedback:** Besides crash reports, collecting feedback from your users from within your app is a great option to help with improving your app. You act on and answer feedback directly from the HockeyApp backend.

5. **Feedback:** Besides crash reports, collecting feedback from your users from within your app is a great option to help with improving your app. You act and answer feedback directly from the HockeyApp backend.

6. **Authenticate:** To help you stay in control of closed tester groups you can identify and authenticate users against your registered testers with the HockeyApp backend. The authentication feature supports several ways of authentication.

This document contains the following sections:

1. [Requirements](#requirements)
2. [Setup](#setup)
  1. [Obtain an app identifier](#app-identifier)
  2. [Get the SDK](#get-sdk)
  3. [Integrate HockeySDK](#integrate-sdk)
  4. [Add Crash Reporting](#crashreporting)
  5. [Add User Metrics](#user-metrics)
  6. [Add update distribution](#updatedistribution)
  7. [Add in-app feedback](#feedback)
  8. [Add authentication](#authentication)
3. [Changelog](#changelog)
4. [Advanced setup](#advancedsetup) 
  1. [Manual library dependency](#manualdependency)
  2. [Crash Reporting](#crashreporting-advanced)
  3. [Update distribution](#updatedistribution-advanced)
  4. [In-App feedback](#feedback-advanced)
  5. [Authentication] (#authentication-advanced)
  6. [Strings & localization](#strings-advanced)
  7. [Permissions](#permissions-advanced)
  8. [Control output to LogCat](#logcat-output)
  9. [ProGuard](#proguard)
5. [Documentation](#documentation)
6. [Troubleshooting](#troubleshooting)
7. [Contributing](#contributing)
8. [Contributor license](#contributorlicense)
9. [Contact](#contact)

<a id="requirements"></a> 
## 1. Requirements

1. We assume that you already have an Android project in Android Studio or another Android IDE.
2. The SDK runs on devices with Android 2.3 or later, but you need to build your app with Android SDK 3.0 (Level 11) or later for the integration with HockeySDK.

<a id="setup"></a>
## 2. Setup

We recommend integration of our compiled library into your project using Android Studio and Gradle.
For other ways to setup the SDK, see [advanced setup](#advancedsetup).
A sample integration can be found in [this GitHub repository](https://github.com/bitstadium/HockeySDK-AndroidDemo).

**Note:** For initial setup it is assumed that you want to use all of HockeyApp's features such as Crash Reporting, update distribution, and feedback. This means your app also needs all the basic permissions. If you only want to use a subset of features and thus only need to ask for a subset of permissions, please see the [permissions section](#permissions-advanced) of the advanced setup section.

<a id="app-identifier"></a>
### 2.1 Obtain an app identifier

Please see the "[How to create a new app](http://support.hockeyapp.net/kb/about-general-faq/how-to-create-a-new-app)" tutorial. This will provide you with an HockeyApp-specific App Identifier to be used to initialize the SDK.

<a id="get-sdk"></a>
### 2.2 Get the SDK

Add the SDK to your app module's dependencies in Android Studio by adding the following line to your `dependencies { ... }` configuration:

```groovy
compile 'net.hockeyapp.android:HockeySDK:4.1.0-beta.2'
```

<a id="integrate-sdk"></a>
### 2.3 Integrate HockeySDK

1. Open your module's `build.gradle` file.
2. Add the following manifest placeholder to your configuration (typically the `defaultConfig`):
  
  ```groovy
  manifestPlaceholders = [HOCKEYAPP_APP_ID: "$APP_ID"]
  ```

3. The param `$APP_ID` must be replaced by your HockeyApp App Identifier. The app identifier can be found on the app's page in the "Overview" section of the HockeyApp backend.
4. Save your `build.gradle` file and make sure to trigger a Gradle sync.
5. Open your AndroidManifest.xml file and add a `meta-data`-tag for the HockeySDK.
	
  ```xml
  <application> 
  	//your activity declarations and other stuff
  	<meta-data android:name="net.hockeyapp.android.appIdentifier" android:value="${HOCKEYAPP_APP_ID}" />
  </application>  
  ```

6. Save your AndroidManifest.xml file.

Now that you've integrated the SDK with your project it's time to make use of its features.

<a id="crashreporting"></a>
### 2.4 Add Crash Reporting
This will add Crash Reporting capabilities to your app. Advanced ways to configure Crash Reporting are covered in [advanced setup](#advancedsetup).

1. Open your main activity.
2. Add the following lines:

```java
import net.hockeyapp.android.CrashManager;

public class YourActivity extends Activity {
  @Override
  public void onResume() {
    super.onResume();
    // ... your own onResume implementation
    checkForCrashes();
  }
    
  private void checkForCrashes() {
    CrashManager.register(this);
  }

}
```

When the activity is resumed, the crash manager is triggered and checks if a new crash was created before. If there a crash log is present, it presents a dialog to ask the user whether they want to send the crash log to HockeyApp. On app launch the crash manager registers a new exception handler to recognize app crashes.

<a id="user-metrics"></a>
### 2.5 Add User Metrics

HockeyApp automatically provides you with nice, intelligible, and informative metrics about how your app is used and by whom.

* **Sessions:** A new session is tracked by the SDK whenever the containing app is restarted (this refers to a 'cold start', i.e. when the app has not already been in memory prior to being launched) or whenever it becomes active again after having been in the background for 20 seconds or more.
* **Users:** The SDK anonymously tracks the users of your app by creating a random UUID that is then securely stored. The UUID is securely stored in the preferences of the client app.
* **Custom Events:** If you are part of [Preseason](hockeyapp.net/preseason), you can now track Custom Events in your app, understand user actions and see the aggregates on the HockeyApp portal.

To integrate User Metrics with your app, perform the following steps:

1. Open your app's main activity and add the import statement and one line of code to the activity's `onCreate`-callback. Add the `trackEvent()`-call wherever you want to track a Custom Event.

```java
//add this import
#import net.hockeyapp.android.metrics.MetricsManager;

//add this to your main activity's onCreate()-callback
MetricsManager.register(this, getApplication());

//add this wherever you want to track a custom event
MetricsManager.trackEvent("YOUR_EVENT_NAME");

```

Make sure to replace `"YOUR_EVENT_NAME"` with a name for your custom event, e.g. `"Login Button Pressed"`. 

**Limits**

* Accepted characters for tracking events are: [a-zA-Z0-9_. -]. If you use other than the accepted characters, your events will not show up in the HockeyApp web portal.
* There is currently a limit of 300 unique event names per app per week.
* There is NO limit on the number of times an event can happen.

<a id="updatedistribution"></a>
### 2.6 Add update distribution
This will add the in-app update mechanism to your app. For more configuration options of the update manager module see the section about [advanced setup](#advancedsetup).

1. Open the activity where you want to inform the user about updates. We'll assume you want to do this on startup of your main activity for our example.
2. Add the following lines and make sure to always balance `register(...)` calls to SDK managers with `unregister()` calls in the corresponding lifecycle callbacks:

```java
import net.hockeyapp.android.UpdateManager;

public class YourActivity extends Activity {
  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // Your own code to create the view
    // ...
    
    checkForUpdates();
  }

  private void checkForUpdates() {
    // Remove this for store builds!
    UpdateManager.register(this);
  }
  
  private void unregisterManagers() {
    UpdateManager.unregister();
  }

    
  @Override
  public void onPause() {
    super.onPause();
    unregisterManagers();
  }
  
  @Override
  public void onDestroy() {
    super.onDestroy();
    unregisterManagers();
  }

}
```

When the activity is created, the update manager checks for new updates in the background. If it finds a new update, an alert dialog is shown and if the user presses **Show**, they will be taken to the update activity. The reason to only do this once upon creation is that the update check causes network traffic and therefore potential costs for your users. In addition, update checks are cached by the HockeySDK to minimize traffic.


<a id="feedback"></a>
### 2.7 Add in-app feedback
This will add the ability for your users to provide feedback from right inside your app. Detailed configuration options are in [advanced setup](#advancedsetup).

1. You'll typically only want to show the feedback interface upon user interaction, for this example we assume you have a button `feedback_button` in your view for this.
2. Add the following lines to your respective activity, handling the touch events and showing the feedback interface:

```java
import net.hockeyapp.android.FeedbackManager;

public class YourActivity extends Activitiy {

  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // Your own code to create the view
    // ...

    FeedbackManager.register(this);

    Button feedbackButton = (Button) findViewById(R.id.feedback_button);
    feedbackButton.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            FeedbackManager.showFeedbackActivity(MainActivity.this);
        }
    });
  }
  
}
```

When the user taps on the feedback button it will launch the HockeySDK's feedback interface, where the user can create a new feedback discussion, add screenshots or other files for reference, and act on their previous feedback conversations.

<a id="authentication"></a>
### 2.8 Add authentication
You can force authentication of your users through the `LoginManager` class. This will show a login screen to users if they are not fully authenticated to protect your app.

1. Retrieve your app secret from the HockeyApp backend. You can find this on the app details page in the backend right next to the "App ID" value. Click "Show" to access it. 
2. Open the activity you want to protect, if you want to protect all of your app this will be your main activity.
3. Add the following lines to this activity:

```java
import net.hockeyapp.android.LoginManager;

public class YourActivity extends Activity {
  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // Your own code to create the view
    // ...

    LoginManager.register(this, APP_SECRET, LoginManager.LOGIN_MODE_EMAIL_PASSWORD);
    LoginManager.verifyLogin(this, getIntent());
  }
}
```

Make sure to replace `APP_SECRET` with the value retrieved in step 1. This will launch the login activity when a user launches your app the first time. To read more about the different, have a look in the chapter in our [advanced setup section ](#authentication-advanced).

<a id="changelog"></a>
## 3. Changelog
You can access the full changelog in our [releases-section](https://github.com/bitstadium/HockeySDK-Android/releases). The following paragraphs contain information what you might need to change when upgrading to the different new versions.

## 3.1 Upgrading from 3.6.x to 3.7.0 or newer

1. We didn't introduce any breaking changes, except that we have raised the minimum API level to 9.
2. Also consider switching to our new register-calls and adding your app id to your configuration as described above.
3. The `Strings` class for overriding SDK strings has been removed in favor of resource merging. See our section on [strings & localizations](#strings-advanced) for more details.
4. If you integrate the SDK using Gradle, you can remove the previously required activities from your manifest file:

```xml
 <!-- HockeySDK Activities – no longer required as of 3.7.0 and up! -->
 <activity android:name="net.hockeyapp.android.UpdateActivity" />
 <activity android:name="net.hockeyapp.android.FeedbackActivity" />
 <activity android:name="net.hockeyapp.android.PaintActivity" />
```

5. Permissions get automatically merged into your manifest. If your app does not use update distribution you might consider removing the permission `WRITE_EXTERNAL_STORAGE` - see the [advanced permissions section](#permissions-advanced) for details.

<a id="advancedsetup"></a> 
## 4. Advanced setup

<a id="manualdependency"></a> 
### 4.1 Manual library dependency
If you don't want to use Gradle or Maven dependency management you can also download and add the library manually. The easiest way to do this is using Android Studio.

1. Download the latest release from [here](http://hockeyapp.net/releases/#android).
2. Unzip the release distribution.
3. Copy the file libs/HockeySDK-$version.aar to the `libs` folder of your Android project. (`$version` is the version of the downloaded SDK, if the version is < 3.7.0 it will be a .jar file instead)
4. Configure your development tools to use the .aar/.jar file.
5. In Android Studio, create a new module via `File > New > New Module`
6. Select **Import .JAR/.AAR Package** and click **Next**.
7. In the next menu select the .aar/.jar file you just copied to the libs folder. You can rename the module to whatever you want, but we in general recommend leaving it as is. If you don't rename the module, it will match the name of the .aar/.jar file, in this case **HockeySDK-4.1.0-beta.2**. This way you'll quickly know which version of the SDK you are using in the future.
8. Make sure Android Studio added the necessary code to integrate the HockeySDK:

Head over to your app's `build.gradle` to verify the dependency was added correctly. It should look like this:

```groovy
dependencies {
	//your other dependencies
	//...
	
    compile project(':HockeySDK-4.1.0-beta.2')
}
```
Next, make sure your `settings.gradle` contains the new module:

```groovy
include ':app', ':HockeySDK-4.1.0-beta.2'
```

Finally, check the `build.gradle` of the newly added module:
```groovy
configurations.maybeCreate("default")
artifacts.add("default", file('HockeySDK-4.1.0-beta.2.aar'))
```

Once you have verified that everything necessary has been added, proceed with [SDK integration](#integrate-sdk).

<a id="crashreporting-advanced"></a>
### 4.2 Crash Reporting
The following options show only some of the many possibilities to interact and fine-tune the Crash Reporting feature. For more please check the full documentation of the classes `net.hockeyapp.android.CrashManager` and `net.hockeyapp.android.CrashManagerListener` in our [documentation](#documentation).

To configure a custom `CrashManagerListener` use the following `register()` method when configuring the manager:

```java
  CrashManager.register(context, APP_ID, new MyCustomCrashManagerListener());
```

#### 4.2.1 Autosend Crash Reports
Crashes are usually sent the next time the app starts. If your custom crash manager listener returns `true` for `shouldAutoUploadCrashes()`, crashes will be sent without any user interaction, otherwise a dialog will appear allowing the user to decide whether they want to send the report or not.

```java
public class MyCustomCrashManagerListener extends CrashManagerListener {
  @Override
  public boolean shouldAutoUploadCrashes() {
    return true;
  }
}
```

#### 4.2.2 Attach additional meta data
Starting with HockeyApp 3.6.0, you can add additional meta data (e.g. user-provided information) to a Crash Report. 
To achieve this call `CrashManager.handleUserInput()` and provide an instance of `net.hockeyapp.android.objects.CrashMetaData`.


<a id="updatedistribution-advanced"></a>
### 4.3 Update distribution
You can customize the behavior of the in-app update process in several ways. The main class to look at is `net.hockeyapp.android.UpdateManagerListener` in our [documentation](#documentation).

To configure a custom `UpdateManagerListener` use the following `register()` method when configuring the manager:

```java
  UpdateManager.register(context, APP_ID, new MyCustomUpdateManagerListener());
```

### 4.3.1 Providing your own user interface for the update process
The `UpdateManager` will select a suitable activity or fragment depending on the availability of the feature. You can also supply your own by overriding the respective methods `getUpdateActivityClass()` and `getUpdateFragmentClass()` in your UpdateManagerListener subclass.


<a id="feedback-advanced"></a> 
### 4.4 In-app feedback
As stated in the setup guide you'll typically want to show the feedback interface from an `onClick`, `onMenuItemSelected`, or `onOptionsItemSelected` listener method.

### 4.4.1 Capturing a screenshot for feedback
You can configure a notification to show to the user. When they select the notification the SDK will create a screenshot from the app in its current state and create a new feedback draft from it.

1. Open the activity from which you want to enable the screenshot.
2. Add this to your `onCreate()` method or an `onClick()` listener:

```java
  FeedbackManager.setActivityForScreenshot(YourActivity.this);
```

<a id="authentication-advanced"></a>
### 4.5 Authentication

HockeySDK-Android currently supports 4 different authentication modes. The following code snippets usually make most sense in your main activity's `onCreate()`-callback.


### 4.5.1 Anonymous

This is equal to the default HockeySDK behavior if you don't use `LoginManager`.
So if you want to switch between authentication modes for whatever reason,

```java 
LoginManager.register(this, APP_SECRET, LoginManager.LOGIN_MODE_ANONYMOUS);
LoginManager.verifyLogin(this, getIntent());
```

will switch the authentication feature off.

### 4.5.2 Authentication via email or email and password

If you want users to authenticate when they first open the app, the following two modes require them to enter their HockeyApp login data.

While `LoginManager.LOGIN_MODE_EMAIL_ONLY` requires users to authenticate with their email address, `LoginManager.LOGIN_MODE_EMAIL_PASSWORD` also requires them to enter their password.

```java 
LoginManager.register(this, APP_SECRET, LoginManager.LOGIN_MODE_EMAIL_PASSWORD);
LoginManager.verifyLogin(this, getIntent());
```

**NOTE**
Both authentication modes don't restrict updates once the user has authenticated themself, so as long as the user has a HockeyApp account, they will continue to receive updates even if they are no longer accociated with your app.
Restricting a version to user's won't have any effect if you have chosen this authentication mode.

### 4.5.3 Verify the user's authentication status and restrict updates

If you want to restrict updates for individual versions or want the HockeySDK to check the your tester is still associated with your app, use `LoginManager.LOGIN_MODE_VALIDATE`.


```java 
LoginManager.register(this, APP_SECRET, LoginManager.LOGIN_MODE_VALIDATE);
LoginManager.verifyLogin(this, getIntent());
```

If you chose this validation mode, the user will no longer be able to use the app if they have been removed from the app.

`LOGIN_MODE_VALIDATE` also affects in-app updates. If you have chosen this authentication mode, you can restrict versions to individual testers.

It also requires to user to have an internet connection to avoid users who want to circumvent your restriction by going into airplane mode.

<a id="strings-advanced"></a>
### 4.6 Strings & localization
HockeySDK for Android comes with English, French, German and Spanish localizations of all user interface strings. If you want to add further localizations or override certain strings to suit your app's user interface, you can simply override them and [resource merging](http://tools.android.com/tech-docs/new-build-system/resource-merging) takes care of the rest.

Our base strings resource file is located in [`hockeysdk/src/main/res/values/strings.xml`](https://github.com/bitstadium/HockeySDK-Android/blob/master/hockeysdk/src/main/res/values/strings.xml). If your app overrides any of these strings in its `strings.xml` file, the overridden strings will be used in your app.

In case you want to add a localization, please also consider [creating a pull request](#contributing).

<a id="permissions-advanced"></a>
### 4.7 Permissions

HockeySDK requires some permissions to be granted for its operation. These are:

* `android.permission.ACCESS_NETWORK_STATE`: Required to verify if network connectivity is available, as a preliminary measure before sending crash reports, checking for updates, and transmitting feedback.
* `android.permission.INTERNET`: Required to actually transmit data to HockeyApp's servers for crash reports, update distribution and feedback.
* `android.permission.WRITE_EXTERNAL_STORAGE`: Required for downloading app updates to a location that is reachable by the Android package installer which takes care of update installation.

HockeyApp registers these permissions with your app's `AndroidManifest.xml` through [manifest merging](http://tools.android.com/tech-docs/new-build-system/user-guide/manifest-merger). By default, all three permissions get added to your app's manifest file.

### 4.7.1 Removing external storage permission
If your app does not require access to external storage – for example if it doesn't use HockeyApp's update distribution – you might want to remove the `WRITE_EXTERNAL_STORAGE`-permission since it might not be needed by your app. To perform this, use a [remove instruction](http://tools.android.com/tech-docs/new-build-system/user-guide/manifest-merger#TOC-tools:node-markers) for manifest merging:


1. Open your `AndroidManifest.xml` file.
2. Add the tools-namespace to the root element if not already present:

  ```xml
  <manifest xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:tools="http://schemas.android.com/tools" …>
  ```
3. Add the remove instruction for the `WRITE_EXTERNAL_STORAGE`-permission after the other permissions:

  ```xml
  <uses-permission
          android:name="android.permission.WRITE_EXTERNAL_STORAGE"
          tools:node="remove" />
  ```
4. Build your app.

The crucial part in this is the `tools:node="remove"`-part which will make sure the complete node will get removed from the resulting manifest file.

**Note:** If you later decide to use update distribution or any of your apps' dependencies requires write access to external storage, you will have to revert this change.

<a id="logcat-output"></a>

### 4.8 Control output to LogCat

You can control the amount of log messages from HockeySDK that show up in LogCat. By default, we keep the noise as low as possible, only errors will show up. To enable additional logging, i.e. while debugging, add the following line of code:

```java
HockeyLog.setLogLevel(Log.DEBUG);
```

The different log levels match Android's own log levels.

```java
HockeyLog.setLogLevel(Log.VERBOSE); // show all log statements
HockeyLog.setLogLevel(Log.DEBUG); // show most log statements – useful for debugging
HockeyLog.setLogLevel(Log.INFO); // show informative or higher log messages
HockeyLog.setLogLevel(Log.WARN); // show warnings and errors
HockeyLog.setLogLevel(Log.ERROR); // show only errors – the default log level
```
<a id="proguard"></a>

### 4.9 ProGuard

Starting with our 3.7.0 release, the SDK ships with the [required ProGuard configuration](https://github.com/bitstadium/HockeySDK-Android/blob/develop/hockeysdk/proguard-rules.pro) out of the box, so typically you won't have to do anything.

However, if you provide a custom user interface fragment for the update distribution module, e.g. by overriding [`UpdateManagerListener.getUpdateFragmentClass()`](https://github.com/bitstadium/HockeySDK-Android/blob/ac386d5e2a02d4c3e1c9dac3cdf7ea68b2c1165e/hockeysdk/src/main/java/net/hockeyapp/android/UpdateManagerListener.java#L56), you will have to add an exception to your ProGuard configuration since this class is instantiated via reflection. Follow these steps:

1. Open your app module's ProGuard configuration (`proguard-rules.pro` in your app's module in Android Studio)
2. Add the following lines at the end of your existing configuration, using the full class name of your custom update fragment:

```
-keepclassmembers class your.custom.UpdateFragment { 
  *;
}
```

<a id="documentation"></a>
## 5. Documentation

Our documentation can be found on [HockeyApp](http://hockeyapp.net/help/sdk/android/4.1.0-beta.2/index.html).

<a id="troubleshooting"></a>
## 6.Troubleshooting

1. Check if the APP_ID matches the App ID in HockeyApp.

2. Check if the `applicationId` in your `build.gradle` file matches the Bundle Identifier of the app in HockeyApp. HockeyApp accepts crashes only if both the App ID and the bundle identifier match their corresponding values in your app. Please note that the package value in your `AndroidManifest.xml` file might differ from the bundle identifier.

3. If your app crashes and you start it again, does the dialog show up which asks the user to send the crash report? If not, please crash your app again, then connect the debugger and set a break point in `CrashManager.java`'s [register](https://github.com/bitstadium/HockeySDK-Android/blob/master/src/main/java/net/hockeyapp/android/CrashManager.java#L100)-method to see why the dialog is not shown.

4. If you continue to encouter issues, please [contact us](http://support.hockeyapp.net/discussion/new).

<a id="contributing"></a>
## 7. Contributing

We're looking forward to your contributions via pull requests.

**Coding style**

* Please follow our [coding styleguide](https://github.com/bitstadium/android-guidelines).
* Every PR should build and lint without errors.

**Development environment**

* Mac/Linux/Windows machine running the latest version of [Android Studio and the Android SDK](https://developer.android.com/sdk/index.html)

<a id="contributorlicense"></a>
## 8. Contributor license

You must sign a [Contributor License Agreement](https://cla.microsoft.com/) before submitting your pull request. To complete the Contributor License Agreement (CLA), you will need to submit a request via the [form](https://cla.microsoft.com/) and then electronically sign the CLA when you receive the email containing the link to the document. You need to sign the CLA only once to cover submission to any Microsoft OSS project. 

<a id="contact"></a>
## 9. Contact

If you have further questions or are running into trouble that cannot be resolved by any of the steps here, feel free to open a GitHub issue here or contact us at [support@hockeyapp.net](mailto:support@hockeyapp.net) or in our [public Slack channel](https://slack.hockeyapp.net).
