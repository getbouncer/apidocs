# React-native integration guide

CardVerify React Native installation guide

Visit our website at [https://www.getbouncer.com](https://www.getbouncer.com) for examples. Native libraries for android and iOS are also available through our private repositories. Contact [license@getbouncer.com](mailto:license@getbouncer.com) for access to these libraries.

CardVerify is closed source, and requires a license agreement. See the [license](./#license) section for details.

## Installation

### 1. Install the [react-native-cardverify](https://www.npmjs.com/package/react-native-cardverify) package from NPM

Add a dependency to `react-native-cardverify` in your `package.json` file. The latest version of `react-native-cardverify` can be added to your dependencies automatically by installing the SDK in the same directory as your `package.json` file.

```text
$ npm install react-native-cardverify
```

### 2. Install the iOS dependencies into your `ios` directory

Add the `CardVerify` and `react-native-cardverify` pods to the `~/ios/Podfile` file in your app

```ruby
target 'Your App' do
  ...
  pod 'CardVerify', :http => 'https://api.getbouncer.com/v1/downloads/sdk/card_verify/<API_KEY>/cardverify-ios-2.0.15.tgz'
  pod 'react-native-cardverify', :path => '../node_modules/react-native-cardverify/react-native-cardverify.podspec'
end
```

CardVerify uses xcframeworks for its native package. Add the following to your `~/ios/Podfile` file in your app to enable this:

```ruby
dynamic_frameworks = ['CardVerify']

pre_install do |installer|
  installer.pod_targets.each do |pod|
    if dynamic_frameworks.include?(pod.name)
      puts "Overriding the build_as_static_framework? method for #{pod.name}"
      def pod.build_as_static_framework?;
        false
      end
    end
  end
end
```

_More installation options for iOS can be found in the **Installation** section of the_ [_iOS integration guide_](../ios-integration-guide/#installation)_._

### 3. Install the android native library

_Note: You will need a username and password to set up these repositories. Please contact_ [_license@getbouncer.com_](mailto:license@getbouncer.com) _to request credentials._

Add the CardVerify repository to the android section of your react-native project. In your `android/build.gradle` file, append the following to repositories:

```groovy
repositories {
    // ...
    mavenCentral()
}
```

Add the cardverify dependencies to the android section of your react-native project. In your `android/app/build.gradle` file, append the following to dependencies:

```groovy
dependencies {
    implementation "com.getbouncer:cardverify-ui:2.0.0084"
    implementation "com.getbouncer:scan-payment-card-detect:2.0.0084"
    implementation "com.getbouncer:scan-payment-ocr:2.0.0084"
    implementation "com.getbouncer:tensorflow-lite:2.0.0084"
}
```

Please add the following to your `android/app/build.gradle` file in the `android` section:
```groovy
aaptOptions {
    noCompress "tflite"
}
```

#### Alternative camera implementations

By default, bouncer uses the Android Camera 1 API. To use Camera2 or CameraX, add one of the following imports:
```groovy
implementation "com.getbouncer:scan-camerax:2.0.0084"

// OR

implementation "com.getbouncer:scan-camera2:2.0.0084"
```

### 4. Link native dependencies

For react-native version 0.59 and below, follow the [linking native dependencies](link-native-dependencies.md) guide.

## Configuration

### 1. Create an API key

Go to the [Bouncer API console](https://api.getbouncer.com/console) and create an API key.

### 2. Configure Android

Open `android/app/src/main/java/[...]/MainApplication.java` and add the following lines to the end of the `onCreate()` method.

```java
import com.getbouncer.RNCardVerifyModule;

...

public class MainApplication
    extends Application
    implements ReactApplication {

    ...

    public void onCreate() {

        ...

        // set your generated API key
        RNCardVerifyModule.apiKey = "<YOUR_API_KEY_HERE>";

        // set to true for experimental expiry extraction
        RNCardVerifyModule.enableExpiryExtraction = false;

        // set to true for experimental name extraction
        RNCardVerifyModule.enableNameExtraction = false;
    }
}
```

### 3. Configure iOS Permissions

CardVerify uses the device camera to scan cards, so you'll need to add a description of camera usage to your `Info.plist` file:

![XCode iOS camera permission](../../../.gitbook/assets/ios_configure_camera_permission.png)

The string you add here will be what CardVerify displays to your users when CardVerify first prompts them for permission to use the camera.

Alternatively, you can add this permission directly to your Info.plist file:

```markup
<key>NSCameraUsageDescription</key>
<string>We need access to your camera to scan your card</string>
```

### 4. Configure iOS

{% tabs %}
{% tab title="Swift" %}
In your `AppDelegate.swift` file, Add an import for `CardVerify`, and set your API key. 

```swift
import UIKit
import CardVerify

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
         Bouncer.configure(apiKey: "<your_api_key_here>") 
        // do any other necessary launch configuration
        return true
    }
}
```
{% endtab %}

{% tab title="Objective C" %}
If your app is configured using objective-c, in your `AppDelegate.m` file, Add an import for `CardVerify`, and set your API key.

```objectivec
#import "AppDelegate.h"
...
@import CardVerify;

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  ...
  [Bouncer configureWithApiKey:@"<your_api_key_here>"];

  return YES;
}
```
{% endtab %}
{% endtabs %}

## Using

react-native-cardverify exposes two static methods on the `CardVerify` object, `isSupportedAsync()` and `scan(bin, last4)`.

To determine if the device supports CardVerify, use the `isSupportedAsync()` method. For example,

```javascript
import CardVerify from 'react-native-cardverify';

CardVerify.isSupportedAsync()
  .then(supported => {
    if (supported) {
      // YES, show scan button
    } else {
      // NO
    }
  })
```

To scan a card, use the `scan(bin, last4)` method.

* `bin` is an optional string of the first 6 digits of the card to be scanned.

  If non-null, the user will be required to scan a card starting with these

  digits.

* `last4` is an optional string of the last 4 digits of the card to be scanned.

  If non-null, the user will be required to scan a card ending with these digits.

For example,

```javascript
import CardVerify from 'react-native-cardverify';

CardVerify.scan(bin, last4)
  .then(({action, scanId, payload, canceledReason}) => {
    if (action === 'scanned') {
      const {
        number,
        expiryMonth,
        expiryYear,
        issuer,
        legalName,
        payloadVersion,
        verificationPayload
      } = payload;
      // Display information, use the verificationPayload to check card validity
    } else if (action === 'canceled') {
      if (canceledReason === 'user_missing_card') {
        // user tapped the "I don't have this card" button
      }
      if (canceledReason === 'user_canceled') {
        // user closed the scan
      }
      if (canceledReason === 'camera_error') {
        // an error occurred with the camera during scan
      }
      if (canceledReason === 'fatal_error') {
        // an error occurred while processing scan results
      }
      if (canceledReason === 'unknown') {
        // the scan was canceled for an unknown reason
      }
    } else if (action === 'skipped') {
      // User skipped
    } else if (action === 'unknown') {
      // Unknown reason for scan canceled
    }
  })
```

## Example app

An example application is included in the [react-native-cardverify github repository](https://github.com/getbouncer/react-native-cardverify).

Inside the `example` directory, you can find an example React Native project that you can run.

_Note: you will have to have a valid API key and repository access to run the demo app._

To run the example app, do the following:

* Add your repository username and password to `android/build.gradle`.
* Add your API key to the cocoapods configuration file `ios/Podfile`.
* Add your API key to `android/app/src/main/java/com/getbouncer/example/MainApplication.java` and `ios/example/AppDelegate.m`.
* Point the android app to the SDK: create a file `example/android/local.properties` with a line

  ```text
  sdk.dir=<full_path_to_android_sdk>
  ```

* `$ npm install`
* `$ cd example`
* `$ npm install`
* `$ cd ios && pod install && cd ..`
* To run Android app: `react-native run-android`
* To run iOS app: `react-native run-ios`

## Troubleshooting

See the [troubleshooting](troubleshooting.md) documentation.

## Authors

Adam Wushensky, Sam King, Zain ul Abi Din, Jaime Park, and Stefano Suryanto

## License

This library is available under paid and free licenses. See the [LICENSE](https://github.com/getbouncer/cardscan-android/blob/master/LICENSE) file for the full license text.

### Quick summary

In short, this library will remain free forever for non-commercial applications, but use by commercial applications is limited to 90 days, after which time a licensing agreement is required. We're also adding some legal liability protections.

After this period commercial applications need to convert to a licensing agreement to continue to use this library.

Details of licensing \(pricing, etc\) are available at [https://getbouncer.com/pricing](https://getbouncer.com/pricing), or you can contact us at [license@getbouncer.com](mailto:license@getbouncer.com).

### More detailed summary

What’s allowed under the license:

* Free use for any app for 90 days \(for demos, evaluations, hackathons, etc\).
* Contributions \(contributors must agree to the [Contributor License Agreement](https://github.com/getbouncer/cardscan-android/blob/master/Contributor%20License%20Agreement)\)
* Any modifications as needed to work in your app

What’s not allowed under the license:

* Commercial applications using the license for longer than 90 days without a license agreement.
* Using us now in a commercial app today? No worries! Just email [license@getbouncer.com](mailto:license@getbouncer.com) and we’ll get you set up.
* Redistribution under a different license
* Removing attribution
* Modifying logos
* Indemnification: using this free software is ‘at your own risk’, so you can't sue Bouncer Technologies, Inc. for problems caused by this library

Questions? Concerns? Please email us at [license@getbouncer.com](mailto:license@getbouncer.com) or ask us on [slack](https://getbouncer.slack.com/).

