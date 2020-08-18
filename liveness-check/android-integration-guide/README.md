---
description: >-
  Scan both sides of the card for improved fraud protection with Liveness Check
  for Android.
---

# Android integration guide

Liveness Android installation guide

The Liveness flow on Android checks to make sure the user has a genuine physical debit or credit card in their possession, and optionally that the card's IIN matches what you have on record.

## Requirements

* Android API level 21 or higher
* AndroidX compatibility
* Kotlin coroutine compatibility

## SDK Size

We try to keep our SDK as small as possible while maintaining good performance. The size impact including our SDK into your app varies depending on some features of your app:

|  | Base SDK | TFLite Framework | Total |
| :--- | :--- | :--- | :--- |
| App does not yet use TFLite & app _is not_ published as bundle | 5.5MB | 4.0MB | 9.5MB |
| App does not yet use TFLite & app _is_ published as a bundle | 5.5MB | 1.0MB | 6.5MB |
| App already uses TFLite | 5.5MB | 0.0MB | 5.5MB |

If your app is not packaged as a bundle, you can reduce the size of the TFLite framework by restricting the binaries included in your APK. Add the following to your `build.gradle` file to include only the `arm` binaries:

```text
android {
    defaultConfig {
        ndk {
            abiFilters 'armeabi-v7a', 'arm64-v8a'
        }
    }
}
```

## Integration

Unlike CardScan, this library is not publicly available. It can be downloaded from our [BinTray](https://bintray.com/bouncerpaid/) repository. You should have been provided with a username and password when you contracted with Bouncer Technologies to use this library. Use those values in your `build.gradle` file to use CardVerify in your app.

If you do not have the appropriate credentials, please contact [Support](mailto:support@getbouncer.com) and we will provide them.

```text
// Add the Bouncer repo to both the buildscript and allprojects elements
repositories {
    ...
    maven {
        url "https://bouncerpaid.bintray.com/liveness-ui-android"
        credentials {
            username "<FILL_IN_YOUR_USERNAME>"
            password "<FILL_IN_YOUR_PASSWORD>"
        }
    }
}
```

Then add a dependency on the liveness library in your module's build.gradle file:

```text
dependencies {
    implementation 'com.getbouncer:liveness-ui:2.0.0018'
}
```

If you are still using CardScan, leave that dependency in place.

## Using

Before you can use Liveness, you must have an API key set up through the [Bouncer API console](https://api.getbouncer.com/console). Please contact [Sam King](mailto:sam@getbouncer.com) or [Steven Liu](mailto:steven@getbouncer.com) to ensure the correct permissions have been added to your API key.

### 1. Library warmup

Liveness will download ML models for use in verifying the authenticity of payment cards. To ensure these models are downloaded by the time Liveness runs, please make sure to call the `warmUp` method on Liveness as early in the app flow as possible. In most cases, this can be done in the `onApplicationCreate` method of your app. Note that `warmUp` processes on a background thread and will not affect your app's startup time.

#### Adding an application lifecycle listener

If your app doesn't already listen to application lifecycle events, you can extend the `Application` object and connect it using your manifest by setting `android:name` on your application node:

```markup
<application
    android:icon="..."
    android:label="..."
    android:roundIcon="..."
    android:name=".MyApplication">

    ...

application>
```

Create a class with the same name:

```java
public class MyApplication extends Application {
    public static final String API_KEY = "<your_api_key_here>";

    @Override
    public void onCreate() {
        super.onCreate();

        /*
         * Liveness will download ML models for use in verifying the authenticity of payment cards. To ensure these
         * models are downloaded by the time Liveness runs, please be sure to call the `warmUp` method on Liveness
         * as early in the app flow as possible. In most cases, this can be done in the `onApplicationCreate` method of
         * your app. Note that `warmUp` processes on a background thread and will not affect your app's startup time.
         */
        LivenessActivity.warmUp(this, API_KEY);
    }
}
```

### 2. Starting the flow

To start the flow, `LivenessActivity` provides a `start` method which takes the required parameters and launches the flow. Pass in the payment card details that you want to verify, including the IIN \(first six digits\) and last four digits of the payment card. You can also optionally specify which side of the card the user should scan. If left `null`, the user will be able to scan either side.

```kotlin
class LaunchActivity : Activity, LivenessActivityResultHandler {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_launch);

        findViewById(R.id.scanCardButton).setOnClickListener { _ ->
            LivenessActivity.start(
                activity = this,
                apiKey = MyApplication.API_KEY,
                cardSide = CardSides.NUMBER_SIDE,
                iin = cardNumber.substring(0, 6),
                lastFour = cardNumber.substring(cardNumber.length() - 4)
            )
        }
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent) {
        super.onActivityResult(requestCode, resultCode, data)

        if (LivenessActivity.isLivenessResult(requestCode)) {
            LivenessActivity.parseLivenessResult(resultCode, data, this)
        }
    }

    override fun cardScanned(
        instanceId: String?,
        scanId: String?,
        payloadVersion: Int,
        encryptedPayload: String,
        cardPan: String?,
        @CardSide cardSide: Int,
        cardImage: Bitmap?
    ) {
        /*
         * The user scanned a card. Details of the card are in the `scanResult` variable. Send the value in the
         * `encryptedPayload` variable to Bouncer servers to check if the card is genuine. See the server integration
         * documentation for details. An image of the scanned card is available in the `cardImage` variable.
         */
    }

    override fun userCanceled(scanId: String?) {
        // The user canceled the scan.
    }

    override fun cameraError(scanId: String?) {
        // The scan failed because of a camera error.
    }

    override fun analyzerFailure(scanId: String?) {
        // The scan failed to analyze images from the camera.
    }

    override fun canceledUnknown(scanId: String?) {
        // The scan was canceled due to unknown reasons.
    }
}
```

Liveness will return an encrypted payload containing information about the payment card the user scanned. As in the example above, you can receive this payload using your activity or fragment's `onActivityResult` method.

## Customizing

Liveness is built to be customized to fit your UI.

### Basic modifications

To modify text, colors, or padding of the default UI, see the [customization](android-ui-customization-guide.md) documentation.

### Extensive modifications

To modify arrangement or UI functionality, Liveness can be used as a library for your custom implementation. See the [example single-activity demo app](https://github.com/getbouncer/cardscan-android/blob/master/demo/src/main/java/com/getbouncer/cardscan/demo/SingleActivityDemo.java).

## Developing

See the [development docs](../../card-scan/android-integration-guide/android-development-guide.md) for details on developing for Liveness.

## License

A licensing agreement is required to use this library.

* Details of licensing \(pricing, etc\) are available at [https://cardscan.io/pricing](https://cardscan.io/pricing), or you can contact us at [license@getbouncer.com](mailto:license@getbouncer.com).

All contributors must agree to the [CLA](https://github.com/getbouncer/cardscan-android/blob/master/Contributor%20License%20Agreement).

