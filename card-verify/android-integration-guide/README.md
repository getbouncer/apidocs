---
description: Scan a payment card for fraud protection with CardVerify Check for Android.
---

# Android integration guide

## Requirements

* Android API level 21 or higher
* AndroidX compatibility
* Kotlin coroutine compatibility

_Note: Your app does not have to be written in kotlin to integrate this library, but must be able to depend on kotlin functionality._

## SDK Size

We try to keep our SDK as small as possible while maintaining good performance. The size impact including our SDK into your app varies depending on some features of your app:

|  | Base SDK | TFLite Framework | Total |
| :--- | :--- | :--- | :--- |
| App does not yet use TFLite & app _is not_ published as bundle | 3.4MB | 4.0MB | 7.4MB |
| App does not yet use TFLite & app _is_ published as a bundle | 3.4MB | 1.0MB | 4.4MB |
| App already uses TFLite | 3.4MB | 0.0MB | 3.4MB |

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
        url "https://bouncerpaid.bintray.com/cardverify-ui-android"
        credentials {
            username "<FILL_IN_YOUR_USERNAME>"
            password "<FILL_IN_YOUR_PASSWORD>"
        }
    }
}
```

```text
dependencies {
    implementation "com.getbouncer:scan-framework:2.0.0054"
    implementation "com.getbouncer:scan-camera:2.0.0054"
    implementation "com.getbouncer:scan-ui:2.0.0054"
    implementation "com.getbouncer:scan-payment:2.0.0054"
    implementation "com.getbouncer:scan-payment-verify:2.0.0054"
    implementation 'com.getbouncer:cardverify-ui:2.0.0054'
}
```

If you are already using CardScan, leave that dependency in place.

## Using

This library provides a user interface through which payment cards can be verified. API keys can be created through the [Bouncer API console](https://api.getbouncer.com/console). Please contact [support](mailto:support@getbouncer.com) to ensure the correct permissions have been added to your API key.

### 1. Library warm up

CardVerify will download ML models for use in verifying the authenticity of payment cards. To ensure these models are downloaded by the time CardVerify runs, please make sure to call the `warmUp` method on CardVerify as early in the app flow as possible. In most cases, this can be done in the `onApplicationCreate` method of your app. Note that `warmUp` processes on a background thread and will not affect your app's startup time.

#### Adding an application lifecycle listener

If your app doesn't already listen to application lifecycle events, you can extend the `Application` object and connect it using your manifest by setting `android:name` on your application node:

```markup
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
         * CardVerify will download ML models for use in verifying the
         * authenticity of payment cards. To ensure these models are downloaded
         * by the time CardVerify runs, please be sure to call the `warmUp`
         * method on CardVerify as early in the app flow as possible. In most
         * cases, this can be done in the `onApplicationCreate` method of your
         * app. Note that `warmUp` processes on a background thread and will not
         * affect your app's startup time.
         */
        CardVerifyActivity.warmUp(this, API_KEY);
    }
}
```

### 2. Starting the flow

To start the flow, `CardVerifyActivity` provides a `start` method which takes the required parameters and launches the flow. Pass in the payment card details that you want to verify, including the IIN \(first six digits\) and last four digits of the payment card.

{% tabs %}
{% tab title="Kotlin" %}
```kotlin
class LaunchActivity : Activity, CardVerifyActivityResultHandler {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_launch)

        findViewById(R.id.scanCardButton).setOnClickListener { _ ->
            CardVerifyActivity.start(
                activity = LaunchActivity.this,
                apiKey = MyApplication.API_KEY,
                iin = requiredCardNumber.take(6),
                lastFour = requiredCardNumber.takeLast(4)
            )
        }
    }

    override fun onActivityResult(
        requestCode: Int,
        resultCode: Int,
        data: Intent
    ) {
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
         * The user scanned a card. Details of the card are in the `scanResult`
         * variable. Send the value in the `encryptedPayload` variable to
         * Bouncer servers to check if the card is genuine. See the server
         * integration documentation for details.
         */
    }

    override fun missingCard(scanId: String?) {
        // The user does not have possession of this card
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
{% endtab %}

{% tab title="Java" %}
```java
class LaunchActivity
    extends AppCompatActivity
    implements CardVerifyActivityResultHandler {

    private static final String API_KEY = "<your_api_key_here>";

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_launch);

        findViewById(R.id.scanCardButton).setOnClickListener(v ->
            CardVerifyActivity.start(
                activity = LaunchActivity.this,
                apiKey = MyApplication.API_KEY,
                iin = requiredCardNumber.take(6),
                lastFour = requiredCardNumber.takeLast(4)
            )
        );
    }

    @Override
    protected void onActivityResult(
        int requestCode,
        int resultCode,
        @Nullable Intent data
    ) {
        super.onActivityResult(requestCode, resultCode, data);

        if (CardVerifyActivity.isVerifyResult(requestCode)) {
            CardVerifyActivity.parseVerifyResult(resultCode, data, this);
        }
    }

    @Override
    public void cardScanned(
        @Nullable String instanceId,
        @Nullable String scanId,
        @NotNull ScanResult scanResult,
        int payloadVersion,
        @NotNull String encryptedPayload
    ) {
        /*
         * The user scanned a card. Details of the card are in the `scanResult`
         * variable. Send the value in the `encryptedPayload` variable to
         * Bouncer servers to check if the card is genuine. See the server
         * integration documentation for details.
         */
    }

    @Override
    public void missingCard(@Nullable String scanId) {
        // The user does not have possession of this card
    }

    @Override
    public void userCanceled(@Nullable String scanId) {
        // the user canceled the scan
    }

    @Override
    public void cameraError(@Nullable String scanId) {
        // scan was canceled due to a camera error
    }

    @Override
    public void analyzerFailure(@Nullable String scanId) {
        // scan was canceled due to a failure to analyze camera images
    }

    @Override
    public void canceledUnknown(@Nullable String scanId) {
        // scan was canceled for an unknown reason
    }
}
```
{% endtab %}
{% endtabs %}

CardVerify will return an encrypted payload containing information about the payment card the user scanned. As in the example above, you can receive this payload using your activity or fragment's `onActivityResult` method.

## Customizing

CardVerify is built to be customized to fit your UI.

### Basic modifications

To modify text, colors, or padding of the default UI, see the [customization](ui-customization-guide.md) documentation.

### Extensive modifications

To modify arrangement or UI functionality, CardVerify can be used as a library for your custom implementation. See the [example single-activity demo app](https://github.com/getbouncer/cardscan-android/blob/master/demo/src/main/java/com/getbouncer/cardscan/demo/SingleActivityDemo.java).

## Developing

See the [development docs](../../card-scan/android-integration-guide/development-guide.md) for details on developing for CardVerify.

## License

A licensing agreement is required to use this library.

* Details of licensing \(pricing, etc\) are available at

  [https://cardscan.io/pricing](https://cardscan.io/pricing), or you can contact

  us at [license@getbouncer.com](mailto:license@getbouncer.com).

All contributors must agree to the [CLA](https://github.com/getbouncer/cardscan-android/blob/master/Contributor%20License%20Agreement).

