---
description: Scan a payment card with CardScan for Android.
---

# Android integration guide

## Requirements

* Android API level 21 or higher
* AndroidX compatibility
* Kotlin coroutine compatibility

_Note: Your app does not have to be written in kotlin to integrate this library, but must be able to depend on kotlin functionality._

## Demo

The [cardscan repository](https://github.com/getbouncer/cardscan-android) contains a demonstration app for the CardScan product. To build and install this library follow the following steps:

1. Clone the repository from github

   ```bash
    git clone --recursive https://github.com/getbouncer/cardscan-android
   ```

2. Build the library using gradle or [android studio](https://developer.android.com/studio). a. Using android studio, open the directory `cardscan-android`. Install the app on your device or an emulator by clicking the play button in the top right of android studio.

   ![build\_android\_studio](../../../.gitbook/assets/android_build_android_studio.png)

   b. Using gradle, build the demo app by executing the following command:

   ```bash
    ./gradlew cardscan-demo:assembleRelease
   ```

   This will create a release APK in the `cardscan-demo/build/outputs/apk` directory. Copy this file to your device and install it.

## SDK Size

We try to keep our SDK as small as possible while maintaining good performance. The size impact including our SDK into your app varies depending on some features of your app:

We also provide custom implementations of the TensorFlow Lite library that are stripped-down to only support the functions our SDK uses. These custom implementations can drastically reduce the size of the SDK overall.

| TF Flavor | Size \(not bundled\) | Size \(bundled\) | Dependency |
| :--- | :--- | :--- | :--- |
| TensorFlow official release | 4.00MB | 1.00MB | `org.tensorflow:tensorflow-lite:2.4.0` |
| Bouncer TF all architectures | 2.12MB | 0.62MB | `com.getbouncer:tensorflow-lite:2.0.0090` |
| Bouncer TF arm only | 1.06MB | 0.62MB | `com.getbouncer:tensorflow-lite-arm-only:2.0.0090` |

There are two variants of our ML models that you can choose to include in your application; the larger size, faster performing and smaller size, slower performing models. If selecting the slower performing models, the SDK will still attempt to download the faster models over the internet once your app has launched. This helps reduce the size of your app while still guaranteeing performance.

| ML Model | Flavor | Size \(not bundled\) | Size \(bundled\) | dependency |
| :--- | :--- | :--- | :--- | :--- |
| OCR | full size | 1.6 MB | 1.5 MB | `com.getbouncer:scan-payment-ocr:2.0.0090` |
| Card Detection | full size | 1.3 MB | 1.2 MB | `com.getbouncer:scan-payment-card-detect:2.0.0090` |
| OCR | minimal | 1.1 MB | 0.9 MB | `com.getbouncer:scan-payment-ocr-minimal:2.0.0090` |
| Card Detection | minimal | 0.5 MB | 0.3 MB | `com.getbouncer:scan-payment-card-detect-minimal:2.0.0090` |

Given the above table, select the framework you'll be using to calculate the impact that the bouncer SDK will have on the size of your application:

| Using minimal models | Base SDK | TFLite Framework | Total |
| :--- | :--- | :--- | :--- |
| App supports all architectures and not bundled | 2.99 MB | 2.12 MB | 5.11 MB |
| App supports ARM only and not bundled | 2.99 MB | 1.06 MB | 4.05 MB |
| App already uses TFLite and not bundled | 2.99 MB | 0.0 MB | 2.99 MB |
| App released as bundle | 2.04 MB | 0.62 MB | 2.66 MB |
| App already uses TFLite and bundled | 2.04 MB | 0.0 MB | 2.04 MB |

| Using full size models | Base SDK | TFLite Framework | Total |
| :--- | :--- | :--- | :--- |
| App supports all architectures and not bundled | 3.9 MB | 2.1 MB | 6.0 MB |
| App supports ARM only and not bundled | 3.9 MB | 1.1 MB | 5.0 MB |
| App released as bundle | 3.9 MB | 0.6 MB | 4.5 MB |
| App already uses TFLite | 3.9 MB | 0.0 MB | 3.9 MB |

Bouncer provides a method `Scan.isDeviceArchitectureArm()` which will return true if the device is running an ARM architecture. This can be used to determine if the device supports scanning when the `-arm-only` TFLite framework is in use.

## Installation

These libraries are published in the [maven central repository](https://search.maven.org/search?q=g:com.getbouncer), so for most gradle configurations you only need to add the dependencies to your app's `build.gradle` file:

```text
dependencies {
    implementation 'com.getbouncer:cardscan-ui:2.0.0090'

    // you must select one of the following sets of OCR and CardDetect
    // frameworks. See the above chart to see how your selection will affect
    // the size of the SDK.

    // To minimize the size of the SDK, use the following dependencies. These
    // perform slightly slower than the larger normal ML models, but will be
    // upgraded over the internet automatically for your users.
    implementation "com.getbouncer:scan-payment-ocr-minimal:2.0.0090"
    implementation "com.getbouncer:scan-payment-card-detect-minimal:2.0.0090"

    // To ensure the maximum performance of the SDK regardless of network
    // connection, but at the cost of a larger SDK, use the following
    // dependencies.
    implementation "com.getbouncer:scan-payment-ocr:2.0.0090"
    implementation "com.getbouncer:scan-payment-card-detect:2.0.0090"



    // you must select one of the following tensorflow-lite libraries. See the
    // above chart to understand how each will affect the size of your app.

    // If you're already using tensorflow lite elsewhere in your project, make
    // sure you depend on the TFLite framework.
    implementation 'org.tensorflow:tensorflow-lite:2.4.0'

    // If you need to support both ARM and x86 devices (< 1% of all android
    // devices), include this dependency.
    implementation 'com.getbouncer:tensorflow-lite:2.0.0090'

    // If you only plan to support ARM devices, use this library
    implementation 'com.getbouncer:tensorflow-lite-arm-only:2.0.0090'
}
```

### Alternative camera implementations

By default, bouncer uses the Android Camera 1 API. To use Camera2 or CameraX, add one of the following imports:

```groovy
implementation "com.getbouncer:scan-camerax:2.0.0090"

// OR

implementation "com.getbouncer:scan-camera2:2.0.0090"
```

## Using

This library provides a user interface through which payment cards can be scanned. API keys can be created through the [Bouncer API console](https://api.getbouncer.com/console).

{% tabs %}
{% tab title="Kotlin" %}
```kotlin
class LaunchActivity : AppCompatActivity, CardScanActivityResultHandler {

    private const val API_KEY = "<your_api_key_here>"

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_launch)

        findViewById(R.id.scanCardButton).setOnClickListener { _ ->
            CardScanActivity.start(
                activity = LaunchActivity.this,
                apiKey = API_KEY,
                enableEnterCardManually = true,

                // expiry extraction is in beta. See the comment below.
                enableExpiryExtraction = true,

                // name extraction is in beta. See the comment below.
                enableNameExtraction = true
            )
        }

        /*
         * To test name and/or expiry extraction, please first provision an API
         * key, then reach out to support@getbouncer.com with details about your
         * use case and estimated volumes.
         *
         * If you are not planning to use name or expiry extraction, you can
         * omit the line below.
         */
        CardScanActivity.warmUp(
            context = this,
            apiKey = API_KEY,
            initializeNameAndExpiryExtraction = true
        )
    }

    override fun onActivityResult(
        requestCode: Int,
        resultCode: Int,
        data: Intent?
    ) {
        super.onActivityResult(requestCode, resultCode, data)

        if (CardScanActivity.isScanResult(requestCode)) {
            CardScanActivity.parseScanResult(resultCode, data, this)
        }
    }

    override fun cardScanned(scanId: String?, scanResult: ScanResult) {
        // a payment card was scanned successfully
    }

    override fun enterManually(scanId: String?) {
        // the user wants to enter a card manually
    }

    override fun userCanceled(scanId: String?) {
        // the user canceled the scan
    }

    override fun cameraError(scanId: String?) {
        // scan was canceled due to a camera error
    }

    override fun analyzerFailure(scanId: String?) {
        // scan was canceled due to a failure to analyze camera images
    }

    override fun canceledUnknown(scanId: String?) {
        // scan was canceled for an unknown reason
    }
}
```
{% endtab %}

{% tab title="Java" %}
```java
class LaunchActivity extends AppCompatActivity
    implements CardScanActivityResultHandler {

    private static final String API_KEY = "<your_api_key_here>";

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_launch);

        findViewById(R.id.scanCardButton).setOnClickListener(v ->
            CardScanActivity.start(
                activity = LaunchActivity.this,
                apiKey = API_KEY,
                enableEnterCardManually = true,

                // expiry extraction is in beta. See the comment below.
                enableExpiryExtraction = true,

                // name extraction is in beta. See the comment below.
                enableNameExtraction = true
            )
        );

        /*
         * To test name and/or expiry extraction, please first provision an API
         * key, then reach out to support@getbouncer.com with details about your
         * use case and estimated volumes.
         *
         * If you are not planning to use name or expiry extraction, you can
         * omit the line below.
         */
        CardScanActivity.warmUp(
            this,
            API_KEY,
            /* enableNameAndExpiryExtraction */ true
        );
    }

    @Override
    protected void onActivityResult(
        int requestCode,
        int resultCode,
        @Nullable Intent data
    ) {
        super.onActivityResult(requestCode, resultCode, data);

        if (CardScanActivity.isScanResult(requestCode)) {
            CardScanActivity.parseScanResult(resultCode, data, this);
        }
    }

    @Override
    public void cardScanned(
        @Nullable String scanId,
        @NotNull CardScanActivityResult scanResult
    ) {
        // a payment card was scanned successfully
    }

    @Override
    public void enterManually(@Nullable String scanId) {
        // the user wants to enter a card manually
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

### Library warm up

CardScan will attempt to update the ML models used to scan payment cards. To ensure these models are upgraded by the time CardScan runs, please make sure to call the `warmUp` method on CardScan as early in the app flow as possible. In most cases, this can be done in the `onApplicationCreate` method of your app. Note that `warmUp` processes on a background thread and will not affect your app's startup time.

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

{% tabs %}
{% tab title="Kotlin" %}
```kotlin
const val API_KEY = "<your_api_key_here>";

class MyApplication : Application() {

    override fun onCreate() {
        super.onCreate();

        /*
         * CardScan will attempt to update the ML models used to scan payment
         * cards. By placing the call to the `warmUp` method in the
         * `onApplicationCreate` method of your app, you allow the most time
         * possible for models to upgrade. Note that `warmUp` processes on a
         * background thread and will not affect your app's startup time.
         */
        CardScanActivity.warmUp(this, API_KEY)
    }
}
```
{% endtab %}

{% tab title="Java" %}
```java
public class MyApplication extends Application {
    public static final String API_KEY = "<your_api_key_here>";

    @Override
    public void onCreate() {
        super.onCreate();

        /*
         * CardScan will attempt to update the ML models used to scan payment
         * cards. By placing the call to the `warmUp` method in the
         * `onApplicationCreate` method of your app, you allow the most time
         * possible for models to upgrade. Note that `warmUp` processes on a
         * background thread and will not affect your app's startup time.
         */
        CardScanActivity.warmUp(this, API_KEY);
    }
}
```
{% endtab %}
{% endtabs %}

## Name and Expiry Extraction

Name and expiry extraction are in beta, and must be manually enabled. The following steps enable them:

1. In the `warmUp` method, set the optional `initializeNameAndExpiryExtraction` parameter to `true`.
2. In the `start` method, set the optional `enableNameExtraction` and/or `enableExpiryExtraction` parameters to `true`.

The scanner will now attempt to extract the cardholder name and card expiry during scan, and will return the values in the `cardScanned` method.

Note that name extraction requires additional permissions on your API key. Please contact us as [license@getbouncer.com](mailto:license@getbouncer.com) to add that permission.

## Model Downloading

By default, the SDK will download updates to ML models used to extract card information. To disable model downloading, set the following flag to `false` before calling warmup or starting the scan:

{% tabs %}
{% tab title="Kotlin" %}
```kotlin
com.getbouncer.scan.framework.Config.downloadModels = false
```
{% endtab %}

{% tab title="Java" %}
```java
com.getbouncer.scan.framework.Config.setDownloadModels(false);
```
{% endtab %}
{% endtabs %}

## UI Customizing

This library is built to be customized to fit your UI.

### Basic modifications

To modify text, colors, or padding of the default UI, see the [customization documentation](ui-customization-guide.md).

### Extensive modifications

To modify arrangement or UI functionality, you can create a custom implementation of this library. See the [example single-activity demo app](https://github.com/getbouncer/cardscan-android/blob/master/cardscan-demo/src/main/java/com/getbouncer/cardscan/demo/SingleActivityDemo.java).

## Supporting more cards

Though CardScan supports several cards, you may need to add support for cards specific to your business, instructions can be found in the [card support docs](card-support.md).

## Developing

The basics of the CardScan architecture are outlined in the [architecture docs](architecture-overview.md).

See the [development docs](development-guide.md) for details on developing for this library.

## Authors

Adam Wushensky, Sam King, Zain ul Abi Din, and Sven Kuhne

## License

This library is available under paid and free licenses. See the [LICENSE](https://github.com/getbouncer/cardscan-android/blob/master/LICENSE) file for the full license text.

### Quick summary

In short, this library will remain free forever for non-commercial applications, but use by commercial applications is limited to 90 days, after which time a licensing agreement is required. We're also adding some legal liability protections.

After this period commercial applications need to convert to a licensing agreement to continue to use this library.

* Details of licensing \(pricing, etc\) are available at [https://cardscan.io/pricing](https://cardscan.io/pricing), or you can contact us at [license@getbouncer.com](mailto:license@getbouncer.com).

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
* Indemnification: using this free software is ‘at your own risk’, so you can’t sue Bouncer Technologies, Inc. for problems caused by this library

Questions? Concerns? Please email us at [license@getbouncer.com](mailto:license@getbouncer.com) or ask us on [slack](https://getbouncer.slack.com/).

