
# Publishing a Flutter App on the Google Play Store and Apple App Store


This guide outlines the steps to build, sign, and release a Flutter app on the Google Play Store and Apple App Store. Follow these instructions carefully to ensure your app is published correctly.

## 1. Build and  Release and Android App

### 1.1 Add a Launcher Icon 

When a new Flutter app is created, it has a default launcher icon. To customize this icon, you might want to check out the **flutter_launcher_icons** package.

Alternatively, you can do it manually using the following steps:

* Review the **Material Design product** icons guidelines for icon design.

* In the [project]/android/app/src/main/res/ directory, place your icon files in folders named using configuration qualifiers. The default mipmap- folders demonstrate the correct naming convention.

* In AndroidManifest.xml, update the application tag's android:icon attribute to reference icons from the previous step (for example, <application android:icon="@mipmap/ic_launcher" ...).

* To verify that the icon has been replaced, run your app and inspect the app icon in the Launcher.

You see how the Gmail App Icon looks the same as if your Flutter app must have an icon. How can you generate it? 🤔 If you have your app logo and wanna make that logo an app icon so just do these steps:
First, go to this website that allows you to check how your logo will look to android and iOS devices.

https://icon.kitchen/i/H4sIAAAAAAAAA6tWKkvMKU0tVrKqVkpJLMoOyUjNTVWySkvMKU6t1VHKzU8pzQHJRisl5qUU5WemKOkoZeYXA8ny1CSl2FoApT8%2BHkAAAAA%3D

<!-- Upload picture here -->

If you have your app logo then you can select the image and upload your app logo and see what it looks like.

<!-- Upload picture here -->

### 1.2 Sign the App 

To publish on the Play Store, you need to sign your app with a digital certificate.
Android uses two signing keys: upload and app signing.

* Developers upload an .aab or .apk file signed with an upload key to the Play Store.

* The end-users download the .apk file signed with an app signing key.

To create your app signing key, use Play App Signing as described in the [official playstore Documentation](https://support.google.com/googleplay/android-developer/answer/7384423?hl=en)

To sign your app, use the following instructions.

### 1.3 Create an upload KeyStore 

If you have an existing keystore, skip to the next step. If not, create one using one of the following methods:

* Follow the [Android Studio key generation steps](https://developer.android.com/studio/publish/app-signing#generate-key)

* Run the following command at the command line:

On macOs or Linux, use the followin Command:

```
keytool -genkey -v -keystore ~/upload-keystore.jks -keyalg RSA \
        -keysize 2048 -validity 10000 -alias upload
```

 On Windows, use the following command in PowerShell:


```
keytool -genkey -v -keystore $env:USERPROFILE\upload-keystore.jks `
        -storetype JKS -keyalg RSA -keysize 2048 -validity 10000 `
        -alias upload

 ```

 You need to follow some steps, by adding your origination/your details
This command stores the upload-keystore.jks file in your home directory. If you want to store it elsewhere, change the argument you pass to the -keystore parameter. However, keep the keystore file private; don't check it into public source control!

```
Note
The keytool command might not be in your path--it's part of Java, which is installed as part of Android Studio. For the concrete path, run flutter doctor -v and locate the path printed after 'Java binary at:'. Then use that fully qualified path replacing java (at the end) with keytool. If your path includes space-separated names, such as Program Files, use platform-appropriate notation for the names. For example, on Mac/Linux use Program\ Files, and on Windows use "Program Files".
The -storetype JKS tag is only required for Java 9 or newer. As of the Java 9 release, the keystore type defaults to PKS12.

```

 ### 1.4 Reference the KeyStore from the app 

 Create a file named [project]/android/key.properties that contains a reference to your keystore. Don't include the angle brackets (< >). They indicate that the text serves as a placeholder for your values.

```
storePassword=<password-from-previous-step>
keyPassword=<password-from-previous-step>
keyAlias=upload
storeFile=<keystore-file-location>

 ```

 The storeFile might be located at /Users/<user name>/upload-keystore.jks on macOS or C:\\Users\\<user name>\\upload-keystore.jks on Windows.

 Warning:
Keep the key.properties file private; don't check it into public source control.

### 1.5 Configure Signing in gradle 

When building your app in release mode, configure gradle to use your upload key. To configure gradle, edit the <project>/android/app/build.gradle file.

* Define and load the keystore properties file before the android property block.

* Set the keystoreProperties object to load the key.properties file.

```
[project]/android/app/build.gradle

 ```

 ```
 def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
   ...
}

 ```

 Add the signing configuration before the buildTypes property block inside the android property block.

 ```
 [project]/android/app/build.gradle

 ```

 ```
 android {
    // ...

    signingConfigs {
        release {
            keyAlias = keystoreProperties['keyAlias']
            keyPassword = keystoreProperties['keyPassword']
            storeFile = keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword = keystoreProperties['storePassword']
        }
    }
    buildTypes {
        release {
            // TODO: Add your own signing config for the release build.
            // Signing with the debug keys for now,
            // so `flutter run --release` works.
         -  signingConfig = signingConfigs.debug
         +   signingConfig = signingConfigs.release
        }
    }
...
}

 ```

 Note
You might need to run flutter clean after changing the gradle file. This prevents cached builds from affecting the signing process.

### 1.6 Review the app manifest:
Review the default App Manifest file.

```
 <manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <application
        android:label="[project]" // your app name
        ...
    </application>
    ...
    <uses-permission android:name="android.permission.INTERNET"/>
</manifest>

 ```

 ### 1.7 Review or change the Gradle build configuration

 To verify the Android build configuration, review the android block in the default Gradle build script. The default Gradle build script is found at [project]/android/app/build.gradle. You can change the values of any of these properties.

 ```
 [project]/android/app/build.gradle

 ```

 ```
android {
    namespace = "com.example.[project]"
    // Any value starting with "flutter." gets its value from
    // the Flutter Gradle plugin.
    // To change from these defaults, make your changes in this file.
    compileSdk = flutter.compileSdkVersion
    ndkVersion = flutter.ndkVersion

    ...

    defaultConfig {
        // TODO: Specify your own unique Application ID (https://developer.android.com/studio/build/application-id.html).
        applicationId = "com.example.[project]" // update your package name

        // You can update the following values to match your application needs.
        minSdk = flutter.minSdkVersion
        targetSdk = flutter.targetSdkVersion
        // These two properties use values defined elsewhere in this file.
        // You can set these values in the property declaration
        // or use a variable.
        versionCode = flutterVersionCode.toInteger()
        versionName = flutterVersionName
    }

    buildTypes {
        ...
    }
}

 ```

 ### 1.8 Build the app for Release 

You have two possible release formats when publishing to the Play Store.

* App Bundle Preffered 
* APK 

### 1.8.1 Build an app bundle 

This section describes how to build a release app bundle. If you completed the signing steps, the app bundle will be signed. At this point, you might consider obfuscating your Dart code to make it more difficult to reverse engineer. Obfuscating your code involves adding a couple flags to your build command, and maintaining additional files to de-obfuscate stack traces.

From the command line:

* Enter cd [project]
* Run flutter build appbundle
  (Running flutter build defaults to a release build.)

The release bundle for your app is created at [project]/build/app/outputs/bundle/release/app.aab

By default, the app bundle contains your Dart code and the Flutter runtime compiled for armeabi-v7a (ARM 32-bit), arm64-v8a (ARM 64-bit), and x86-64 (x86 64-bit).

### 1.8.2 Build an APK 

Although app bundles are preferred over APKs, there are stores that don't yet support app bundles. In this case, build a release APK for each target ABI (Application Binary Interface).

If you complete the signing steps, the APK will be signed. At this point, you might consider obfuscating your Dart code to make it more difficult to reverse engineer. Obfuscating your code involves adding a couple of flags to your build command.

From the command line:

* Enter cd [project].

* Run flutter build apk --split-per-abi. (The flutter 
build command defaults to --release.)

* [project]/build/app/outputs/apk/release/app-armeabi-v7a-release.apk

* [project]/build/app/outputs/apk/release/app-arm64-v8a-release.apk

* [project]/build/app/outputs/apk/release/app-x86_64-release.apk

Removing the --split-per-abi flag results in a fat APK that contains your code compiled for all the target ABIs. Such APKs are larger in size than their split counterparts, causing the user to download native binaries that are not applicable to their device's architecture.

### 1.9 Update the app's version number 

The default version number of the app is 1.0.0. To update it, navigate to the pubspec.yaml file and update the following line:

version: 1.0.0+1

The version number is three numbers separated by dots, such as 1.0.0 in the example above, followed by an optional build number such as 1 in the example above, separated by a +.

Both the version and the build number can be overridden in Flutter's build by specifying --build-name and --build-number, respectively.

In Android, build-name is used as versionName while build-number used as versionCode. For more information, check out Version your app in the Android documentation.

When you rebuild the app for Android, any updates in the version number from the pubspec file will update the versionName and versionCode in the local.properties file.

### 1.10 Publish to PlayStore 

Now, your app bundle is reading so the final step is to publish it to the Play Store. For that, you must have a developer account for google play console. First, go and create a developer account from here:


https://play.google.com/console/signup?source=post_page-----69cb0cd5a30b

### After that, you need to pay the one-time fees that are:

<!-- Add a picture here -->

After successful registration, you will have your developer account where you can upload your apps.

Go to your console:  https://play.google.com/console

Now let’s start publishing your app. First create an app as shown:

<!-- Add a picture here -->

After that provide the information according to your app. I am giving you just an example of putting data.

* Name your app whatever you want. 

* Select the default language of your app. 

* Select whether it is just an app or any gaming app.

<!-- Add a picture here -->

__Make sure if you are making your app free this time so you will never change it in the future so decide it accordingly.__

And, obviously you need to check the declarations.

<!-- Add a picture here -->

By this your app is created successfully. Now you need to start the configuration by providing your app details.

* Just go to the store settings tab.

* Provide if it is app or game.

* Select the right category of your app.

* Then select the correct tags that your based on. You can select up to 5 tags.

<!-- Add a picture here -->

* Then provide the contact details where all app related notification will receive. It includes the app launches details or etc.

<!-- Add a picture here -->

* After that provide the necessary app details

<!-- Add a picture here -->

Now scroll down and you need to provide your app icon and make sure the dimention and size as listed here:

<!-- Add a picture here -->

Now you need to provide the feature graphics of your app which will show as cover photo of your store listing.

<!-- Add a picture here -->

* Now you need to provide the screen shots of phone screen where 2 is the minimum and 8 is the the maximum.
* You can also provide the video link to showcase your app.

<!-- Add a picture here -->

* You are also needed to provide the screen shots of 7-inch tablets and 10-inch tablets. You can provide the same screen shots to these if you don’t have specifically for it.

* Now there are list of tracks where you can upload your app. There are follows:

__Let’s say we are uploading for public rating so we choose the production track__


*Go to production tab and then create new release.


<!-- Add a picture here -->

* Now you will have to choose the signing key.

<!-- Add a picture here -->

* Then choose google-generated key

<!-- Add a picture here -->

* Now you need to upload the app bundle which we have created in start. can find your bundle on your project location.

* In my case the path is:

* D:\FlutterProjects\[project_name]\build\app\outputs\bundle

<!-- Add a picture here -->

After that you can enter the message for this release like if you have new updates and features so can show as message here.

<!-- Add a picture here -->

* Now you would be facing some errors like those shown 

<!-- Add a picture here -->

* Let resolve it to finally rollout this app.

* Click back on production tab and go to select the contries and region for where your app will be available. You can select or de-select any if you don’t want your app there.

<!-- Add a picture here -->

* Now you need to select the countries. If you want all so simply select all from there and save it.

<!-- Add a picture here -->

* After that go down on action tab and find app content tab where you need to provide the further declaration for this app.

<!-- Add a picture here -->

* Here you need to provide the URL hosted on any where that shows your app privacy policy

<!-- Add a picture here -->

* Now save it to clear this step and get back from it with the arrow above.

<!-- Add a picture here -->

* Now save it to clear this step and get back from it with the arrow above.

<!-- Add a picture here -->

* After that choose according to your app whether contains ads or not.

<!-- Add a picture here -->

* Now save it to clear this step and get back from it with the arrow above

* Now you need to tell whether your app can be access completely or some part needs special access.

<!-- Add a picture here -->

* Do according to your app if all functionality is accessible without any restriction or not.

<!-- Add a picture here -->

 Now save it to clear this step and get back from it with the arrow above.

* Now its time to tell the content rating and start the questionnaire

<!-- Add a picture here -->

* You need to give the email address and category as:

<!-- Add a picture here -->

* Then do answer these questionnaire according to your app and go next:

<!-- Add a picture here -->

* After that there would be the summary of what you have selected just save it after review.

<!-- Add a picture here -->

 Now save it to clear this step and get back from it with the arrow above.

<!-- Add a picture here -->

* Select it according to your app and then save it.

<!-- Add a picture here -->

* Then you need to tell whether your app has any personal data or not. Do save and save.

<!-- Add a picture here -->

* If you want your app to be tested by teachers so you can apply for it then hit save.

<!-- Add a picture here -->

* Then finally save it clear this step and get back from it with the arrow above.

<!-- Add a picture here -->

* Choose according to your app's purpose.

<!-- Add a picture here -->

 Now save it to clear this step and get back from it with the arrow above.

* After that, you need to tell if your app is a COVID-19 contact tracing status app or other.

<!-- Add a picture here -->

### Choose according to your app.

<!-- Add a picture here -->

 Now save it to clear this step and get back from it with the arrow above.

* After that, you need to tell about your app’s privacy and security practices.

<!-- Add a picture here -->

* Provide these details and hit next.

<!-- Add a picture here -->

* Then finally save it clear this step and get back from it with the arrow above.

<!-- Add a picture here -->

### Choose according to your app.

Now save it to clear this step and get back from it with the arrow above

* After that, you need to tell whether your app is government property or not.

<!-- Add a picture here -->

### Choose according to your app.

<!-- Add a picture here -->

Now save it to clear this step and get back from it with the arrow above.

* After that, you need to tell the financial features of your app

<!-- Add a picture here -->

### Choose according to your app 

<!-- Add a picture here -->

Now save it to clear this step and get back from it with the arrow above. 

After completing all these declarations get back to the production tab and go to the edit release.

<!-- Add a picture here -->

Then check every detail and click on next.

<!-- Add a picture here -->

After that save it and go to overview.

<!-- Add a picture here -->

Now, finally, send these changes to Google for review.

<!-- Add a picture here -->

Now, if you come to all app sections you will see the app sent for review 😍
Finally, you did it!🥴 your app is sent for review and it will take around 48 hours to give you a response. Make sure that your app uses the user's current location or these types of features so Google will ask to provide valid details for accessing it.


## 2. Build and release an IOS app 

This guide provides a step-by-step walkthrough of releasing a Flutter app to the [App Store](https://developer.apple.com/app-store/submissions/) and [TestFlight](https://developer.apple.com/testflight/)

### 2.1 Preliminaries 

Xcode is required to build and release your app. You must use a device running macOS to follow this guide.

Before beginning the process of releasing your app, ensure that it meets Apple's [App Review Guidelines](https://developer.apple.com/app-store/review/)

To publish your app to the App Store, you must first enroll in the [App Developer Program](https://developer.apple.com/programs/) You can read more about the various membership options in Apple's [Choosing a MemberShip](https://developer.apple.com/support/compare-memberships/) guide.

### 2.2 Register and app on App Store Connect 

Manage your app's life cycle on [App Store Connect](https://developer.apple.com/support/app-store-connect/) (formerly iTunes Connect). You define your app name and description, add screenshots, set pricing, and manage releases to the App Store and TestFlight.

Registering your app involves two steps: registering a unique Bundle ID, and creating an application record on App Store Connect.

For a detailed overview of App Store Connect, see the 
[App Store Connect](https://developer.apple.com/support/app-store-connect/)

### 2.3 Register Bundle ID

Every iOS application is associated with a Bundle ID, a unique identifier registered with Apple. To register a Bundle ID for your app, follow these steps:

* Open the [App IDs] page of your developer account.

* Click + to create a new Bundle ID

* Enter an app name , select Explicit App ID, and enter an ID.

* On the next page, confirm the details and click Register to register your Bundle ID.

### 2.4 Create an application record on App Store Connect 

Register your app on App Store Connect:

* Open [App Store Connect](https://appstoreconnect.apple.com/) in your browser.

* On the App Store Connect landing page, click My Apps

* Click + in the top-left corner of the My Apps page, then select New App.

* Fill in your app details in the form that appears. In the Platforms section, ensure that iOS is checked. Since Flutter does not currently support tvOS, leave that checkbox unchecked. Click Create.

* Navigate to the application details for your app and select App Information from the sidebar.

* In the General Information section, select the Bundle ID you registered in the preceding step.

For a detailed overview, see [Add an app to your account](https://help.apple.com/app-store-connect/#/dev2cd126805)

### 2.5 Review Xcode Project Settings 

This step covers reviewing the most important settings in the Xcode workspace. For detailed procedures and descriptions, see [Prepare for app distribution](https://help.apple.com/xcode/mac/current/#/dev91fe7130a)

Navigate to your target's settings in Xcode:

* Open the default Xcode workspace in your project by running open ios/Runner.xcworkspace in a terminal window from your Flutter project directory.

* To view your app's settings, select the Runner target in the Xcode navigator.

Verify the most important settings.

In the Identity section of the General tab:

__Display Name__

The display name of your app.

__Bundle Identifier__

The App ID you registered on App Store Connect.
In the Signing & Capabilities tab:

__Automatically manage signing__

Whether Xcode should automatically manage app signing and provisioning. This is set true by default, which should be sufficient for most apps. For more complex scenarios, see the [Code Signing Guide](https://developer.apple.com/library/content/documentation/Security/Conceptual/CodeSigningGuide/Introduction/Introduction.html) 

__Team__

Select the team associated with your registered Apple Developer account. If required, select Add Account..., then update this setting.
In the Deployment section of the Build Settings tab:

__iOS Deployment Target__

The minimum iOS version that your app supports. Flutter supports iOS 12 and later. If your app or plugins include Objective-C or Swift code that makes use of APIs newer than iOS 12, update this setting to the highest required version.

The General tab of your project settings should resemble the following:

<!-- Add a picture here -->

For a detailed overview of app signing, see [Create, export, and delete signing certificates.](https://help.apple.com/xcode/mac/current/#/dev154b28f09)

### 2.6 Updating the apps deployment Version

If you change __Deployment Target__ in your Xcode project, open __ios/Flutter/AppframeworkInfo.plist__ in your Flutter app and update the MinimumOSVersion value to match.

### 2.7 Add an app icon 

When a new Flutter app is created, a placeholder icon set is created. This step covers replacing these placeholder icons with your app's icons:







