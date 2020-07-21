# Android integration guide

## Requirements
* Android API level 21 or higher
* AndroidX compatibility
* Kotlin coroutine compatibility

*Note: Your app does not have to be written in kotlin to integrate this library, but must be able to depend on kotlin functionality.*

## Integration
Unlike CardScan, this library is not publicly available. It can be downloaded from our [BinTray](https://bintray.com/bouncerpaid/) repository. You should have been provided with a username and password when you contracted with Bouncer Technologies to use this library. Use those values in your `build.gradle` file to use CardVerify in your app.

If you do not have the appropriate credentials, please contact [Support](mailto:support@getbouncer.com) and we will provide them.

```gradle
// Add the Bouncer repo to both the buildscript and allprojects elements
repositories {
    ...
    maven {
        url "https://bouncerpaid.bintray.com/cardverify-ui-android"
        credentials {
            username "<FILL_IN_YOUR_USERNAME>"
            password "<FILL_IN_YOUR_PASSWORD>"
        }
    }
}
```

```gradle
dependencies {
    implementation 'com.getbouncer:cardverify-ui:2.0.0016'
}
```

If you are already using CardScan, leave that dependency in place.

## Using
This library provides a user interface through which payment cards can be verified. API keys can be created through the [Bouncer API console](https://api.getbouncer.com/console). Please contact [support](mailto:support@getbouncer.com) to ensure the correct permissions have been added to your API key.

### 1. Library warm up
CardVerify will download ML models for use in verifying the authenticity of payment cards. To ensure these models are downloaded by the time CardVerify runs, please make sure to call the `warmUp` method on CardVerify as early in the app flow as possible. In most cases, this can be done in the `onApplicationCreate` method of your app. Note that `warmUp` processes on a background thread and will not affect your app's startup time.

#### Adding an application lifecycle listener
If your app doesn't already listen to application lifecycle events, you can extend the `Application` object and connect it using your manifest by setting `android:name` on your application node:

```xml
<application
    android:icon="..."
    android:label="..."
    android:roundIcon="..."
    android:name=".MyApplication">

    ...

</application>
```

Create a class with the same name:
```java
public class MyApplication extends Application {
    public static final String API_KEY = "<your_api_key_here>";

    @Override
    public void onCreate() {
        super.onCreate();

        /*
         * CardVerify will download ML models for use in verifying the authenticity of payment cards. To ensure these
         * models are downloaded by the time CardVerify runs, please be sure to call the `warmUp` method on CardVerify
         * as early in the app flow as possible. In most cases, this can be done in the `onApplicationCreate` method of
         * your app. Note that `warmUp` processes on a background thread and will not affect your app's startup time.
         */
        CardVerifyActivity.warmUp(this, API_KEY);
    }
}
```

### 2. Starting the flow
To start the flow, `CardVerifyActivity` provides a `start` method which takes the required parameters and launches the flow. Pass in the payment card details that you want to verify, including the IIN \(first six digits\) and last four digits of the payment card.

```kotlin
class LaunchActivity : Activity, CardVerifyActivityResultHandler {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_launch);

        findViewById(R.id.scanCardButton).setOnClickListener { _ ->
            CardVerifyActivity.start(
                activity = LaunchActivity.this,
                apiKey = MyApplication.API_KEY,
                iin = requiredCardNumber.take(6),
                lastFour = requiredCardNumber.takeLast(4)
            )
        }
    }
    
    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent) {
        super.onActivityResult(requestCode, resultCode, data)

        if (CardVerifyActivity.isVerifyResult(requestCode)) {
            CardVerifyActivity.parseVerifyResult(resultCode, data, this)
        }
    }
    
    override fun cardScanned(
        instanceId: String?,
        scanId: String?,
        scanResult: ScanResult,
        payloadVersion: Int,
        encryptedPayload: String
    ) {
        /*
         * The user scanned a card. Details of the card are in the `scanResult` variable. Send the value in the
         * `encryptedPayload` variable to Bouncer servers to check if the card is genuine. See the server integration
         * documentation for details.
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

CardVerify will return an encrypted payload containing information about the payment card the user scanned. As in the example above, you can receive this payload using your activity or fragment's `onActivityResult` method.

## Customizing
CardVerify is built to be customized to fit your UI.

### Basic modifications
To modify text, colors, or padding of the default UI, see the [customization](/cardscan/android-integration-guide/android-customization-guide.md) documentation.

### Extensive modifications
To modify arrangement or UI functionality, CardVerify can be used as a library for your custom implementation. See the [example single-activity demo app](https://github.com/getbouncer/cardscan-android/blob/master/demo/src/main/java/com/getbouncer/cardscan/demo/SingleActivityDemo.java).

## Developing
See the [development docs](card-scan/android-integration-guide/android-development-guide) for details on developing for CardVerify.

## License
A licensing agreement is required to use this library.
* Details of licensing \(pricing, etc\) are available at [https://cardscan.io/pricing](https://cardscan.io/pricing), or you can contact us at [license@getbouncer.com](mailto:license@getbouncer.com).

All contributors must agree to the [CLA](https://github.com/getbouncer/cardscan-android/blob/master/Contributor%20License%20Agreement).
