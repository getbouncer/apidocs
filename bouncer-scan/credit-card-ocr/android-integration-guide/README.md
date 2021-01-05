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

|  | Base SDK | TFLite Framework | Total |
| :--- | :--- | :--- | :--- |
| App does not yet use TFLite & app _is not_ published as bundle | 1.9MB | 4.0MB | 5.9MB |
| App does not yet use TFLite & app _is_ published as a bundle | 1.9MB | 1.0MB | 2.9MB |
| App already uses TFLite | 1.9MB | 0.0MB | 1.9MB |

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

## Installation

These libraries are published in the [jcenter](https://jcenter.bintray.com/com/getbouncer/) repository, so for most gradle configurations you only need to add the dependencies to your app's `build.gradle` file:

```text
dependencies {
    implementation 'com.getbouncer:cardscan-ui:2.0.0062'
}
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

## Customizing

This library is built to be customized to fit your UI.

### Basic modifications

To modify text, colors, or padding of the default UI, see the [customization documentation](ui-customization-guide.md).

### Extensive modifications

To modify arrangement or UI functionality, you can create a custom implementation of this library. See the [example single-activity demo app](https://github.com/getbouncer/cardscan-android/blob/master/demo/src/main/java/com/getbouncer/cardscan/demo/SingleActivityDemo.java).

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
