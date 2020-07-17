# Android integration guide

## Requirements

* Android API level 21 or higher
* AndroidX compatibility
* Kotlin coroutine compatibility

Note: Your app does not have to be written in kotlin to integrate this library, but must be able to depend on kotlin functionality.

## Demo

An app demonstrating the basic capabilities of this library is available in [github](https://github.com/getbouncer/cardscan-demo-android).

## Installation

These libraries are published in the [jcenter](https://jcenter.bintray.com/com/getbouncer/) repository, so for most gradle configurations you only need to add the dependencies to your app's `build.gradle` file:

```gradle
dependencies {
    implementation 'com.getbouncer:cardscan-ui:2.0.0014'
}
```

## Using

This library provides a user interface through which payment cards can be scanned.

```kotlin
class LaunchActivity : AppCompatActivity, CardScanActivityResultHandler {

    private const val API_KEY = "";

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_launch);

        // Because this activity displays card numbers, disallow screenshots.
        window.setFlags(
            WindowManager.LayoutParams.FLAG_SECURE,
            WindowManager.LayoutParams.FLAG_SECURE
        )

        findViewById(R.id.scanCardButton).setOnClickListener { _ ->
            CardScanActivity.start(
                activity = LaunchActivity.this,
                apiKey = API_KEY,
                enableEnterCardManually = true,
                enableNameExtraction = true  // name extraction is in beta. See the comment below.
            )
        }

        /*
         * To test name extraction, please first provision an API key, then reach out to support@bouncer.com with
         * details about your use case and estimated volumes.
         *
         * If you are not planning to use name extraction, you can omit the line below.
         */
        CardScanActivity.initializeNameExtraction(this, API_KEY)
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
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

## Customizing

This library is built to be customized to fit your UI.

### Basic modifications

To modify text, colors, or padding of the default UI, see the [customization](https://github.com/getbouncer/scan-ui-android/blob/master/docs/customize.md) documentation.

### Extensive modifications

To modify arrangement or UI functionality, you can create a custom implementation of this library. See examples in the [scan-ui](https://github.com/getbouncer/scan-ui-android) repository.

## Developing

See the [development docs](https://github.com/getbouncer/cardscan-ui-android/blob/master/docs/develop.md) for details on developing for this library.

## Authors

Adam Wushensky, Sam King, and Zain ul Abi Din

## License

This library is available under paid and free licenses. See the [LICENSE](https://github.com/getbouncer/cardscan-ui-android/blob/master/LICENSE) file for the full license text.

### Quick summary

In short, this library will remain free forever for non-commercial applications, but use by commercial applications is limited to 90 days, after which time a licensing agreement is required. We're also adding some legal liability protections.

After this period commercial applications need to convert to a licensing agreement to continue to use this library.

* Details of licensing \(pricing, etc\) are available at [https://cardscan.io/pricing](https://cardscan.io/pricing), or you can contact us at [license@getbouncer.com](mailto:license@getbouncer.com).

### More detailed summary

What’s allowed under the license:

* Free use for any app for 90 days \(for demos, evaluations, hackathons, etc\).
* Contributions \(contributors must agree to the [Contributor License Agreement](https://github.com/getbouncer/cardscan-ui-android/blob/master/Contributor%20License%20Agreement)\)
* Any modifications as needed to work in your app

What’s not allowed under the license:

* Commercial applications using the license for longer than 90 days without a license agreement.
* Using us now in a commercial app today? No worries! Just email [license@getbouncer.com](mailto:license@getbouncer.com) and we’ll get you set up.
* Redistribution under a different license
* Removing attribution
* Modifying logos
* Indemnification: using this free software is ‘at your own risk’, so you can’t sue Bouncer Technologies, Inc. for problems caused by this library

Questions? Concerns? Please email us at [license@getbouncer.com](mailto:license@getbouncer.com) or ask us on [slack](https://getbouncer.slack.com/).

