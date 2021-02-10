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

We also provide custom implementations of the TensorFlow Lite library that are stripped-down to only support the functions our SDK uses. These custom implementations can drastically reduce the size of the SDK overall.

| TF Flavor | Size \(not bundled\) | Size \(bundled\) | Dependency |
| :--- | :--- | :--- | :--- |
| TensorFlow official release | 4.0MB | 1.0MB | `com.tensorflow:tensorflow-lite:2.4.0` |
| Bouncer TF all architectures | 3.3MB | 0.8MB | `com.getbouncer:tensorflow-lite:2.0.0072` |
| Bouncer TF arm only | 1.9MB | 0.8MB | `com.getbouncer:tensorflow-lite-arm-only:2.0.0072` |

Given the above table, select the framework you'll be using to calculate the impact that the bouncer SDK will have on the size of your SDK:

|  | Base SDK | TFLite Framework | Total |
| :--- | :--- | :--- | :--- |
| App supports all architectures and not bundled | 3.9 MB | 3.3 MB | 7.2 MB |
| App supports ARM only and not bundled | 3.9 MB | 1.9 MB | 5.8 MB |
| App released as bundle | 3.9 MB | 0.8 MB | 4.7 MB |
| App already uses TFLite | 3.9 MB | 0.0 MB | 3.9 MB |

Bouncer provides a method `Scan.isDeviceArchitectureArm()` which will return true if the device is running an ARM architecture. This can be used to determine if the device supports scanning when the `-arm-only` TFLite framework is in use.

## Integration

Unlike CardScan, this library is not publicly available. You should have been provided with a username and password when you contracted with Bouncer Technologies to use this library. Use those values in your `build.gradle` file to use CardVerify in your app.

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
    implementation "com.getbouncer:scan-framework:2.0.0072"
    implementation "com.getbouncer:scan-camera:2.0.0072"
    implementation "com.getbouncer:scan-ui:2.0.0072"
    implementation "com.getbouncer:scan-payment:2.0.0072"
    implementation "com.getbouncer:scan-payment-card-detect:2.0.0072"
    implementation 'com.getbouncer:cardverify-ui:2.0.0072'

    // you must select one of the following tensorflow-lite libraries. See the
    // above chart to understand how each will affect the size of your app.

    // If you're already using tensorflow lite elsewhere in your project, make
    // sure you depend on the TFLite framework.
    implementation 'com.tensorflow:tensorflow-lite:2.4.0'

    // If you need to support both ARM and x86 devices (< 1% of all android
    // devices), include this dependency.
    implementation 'com.getbouncer:tensorflow-lite:2.0.0072'

    // If you only plan to support ARM devices, use this library
    implementation 'com.getbouncer:tensorflow-lite-arm-only:2.0.0072'
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

    override fun userMissingCard(scanId: String?) {
        // The user does not have possession of this card
    }

    override fun enterManually(scanId: String?) {
        // The user wants to manually enter a card
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
    public void userMissingCard(@Nullable String scanId) {
        // The user does not have possession of this card
    }

    @Override
    public void enterManually(@Nullable String scanId) {
        // The user wants to manually enter a card
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

## Name and Expiry Extraction

Name and expiry extraction are in beta, and must be manually enabled. The following steps enable them:

1. In the `warmUp` method, set the optional `initializeNameAndExpiryExtraction` parameter to `true`.
2. In the `start` method, set the optional `enableNameExtraction` and/or `enableExpiryExtraction` parameters to `true`.

The scanner will now attempt to extract the cardholder name and card expiry during scan, and will return the values in the `cardScanned` method.

## Customizing

CardVerify is built to be customized to fit your UI.

### Basic modifications

To modify text, colors, or padding of the default UI, see the [customization](ui-customization-guide.md) documentation.

### Extensive modifications

To modify arrangement or UI functionality, CardVerify can be used as a library for your custom implementation. See the [example single-activity demo app](https://github.com/getbouncer/cardscan-android/blob/master/cardscan-demo/src/main/java/com/getbouncer/cardscan/demo/SingleActivityDemo.java).

## Developing

See the [development docs](../../credit-card-ocr/android-integration-guide/development-guide.md) for details on developing for CardVerify.

## License

A licensing agreement is required to use this library.

* Details of licensing \(pricing, etc\) are available at

  [https://cardscan.io/pricing](https://cardscan.io/pricing), or you can contact

  us at [license@getbouncer.com](mailto:license@getbouncer.com).

All contributors must agree to the [CLA](https://github.com/getbouncer/cardscan-android/blob/master/Contributor%20License%20Agreement).

