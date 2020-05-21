# CardScan

[CardScan](https://github.com/getbouncer/cardscan-ui-android) Android installation guide

## Contents

* [Requirements](#requirements)
* [Installation](#installation)
* [Using](#using)
* [License](#license)

## Requirements

* Android API level 21 or higher
* AndroidX compatibility
* Kotlin coroutine compatibility

## Installation

These libraries are published in the [jcenter](https://jcenter.bintray.com/com/getbouncer/) repository, so for most gradle configurations you only need to add the dependencies to your app's `build.gradle` file:

```gradle
dependencies {
    implementation 'com.getbouncer:cardscan-ui:2.0.0008'
}
```

## Using

Before you can use CardScan, you must have an API key set up through the [Bouncer API console](https://api.getbouncer.com/console).

### 1. Library warmup

CardScan will download ML models for use in verifying the authenticity of payment cards. To ensure these models are downloaded by the time CardVerify runs, please make sure to call the `warmUp` method on CardVerify as early in the app flow as possible. In most cases, this can be done in the `onApplicationCreate` method of your app. Note that `warmUp` processes on a background thread and will not affect your app's startup time. 

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
    @Override
    public void onCreate() {
        super.onCreate();
        CardScanActivity.warmUp(this, "<YOUR_API_KEY_HERE>");
    }
}
```

### 2. Starting the flow

To start the flow, `CardScanActivity` provides a `start` method which takes the required parameters and launches the flow.

```java
public void scanPaymentCard() {
    CardScanActivity.start(
        /* activity or fragment */ LaunchActivity.this,
        /* apiKey */ "<YOUR_API_KEY_HERE>",
        /* enableEnterCardManually */ true
    );
}
```

CardScan will return details about the payment card the user scanned. You can receive this information using your activity or fragment's `onActivityResult` method.

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    if (CardScanActivity.isScanResult(requestCode)) {
        CardScanActivity.parseScanResult(resultCode, data, new CardScanActivityResultHandler() {
            @Override
            public void cardScanned(@Nullable String scanId, @NotNull CardScanActivityResult scanResult) {
                // The user scanned a card. Details of the card are in the `scanResult` variable.
            }
        
            @Override
            public void enterManually(@Nullable String scanId) {
                // The user tapped the `enter card manually` button
            }
        
            @Override
            public void userCanceled(@Nullable String scanId) {
                // The user pressed the back or close button
            }
        
            @Override
            public void cameraError(@Nullable String scanId) {
                // An error occurred while accessing the camera
            }
        
            @Override
            public void analyzerFailure(@Nullable String scanId) {
                // An error occurred while analyzing the payment card
            }
        
            @Override
            public void canceledUnknown(@Nullable String scanId) {
                // The scan was canceled for an unknown reason
            }
        });
    }
}
```

## License

This library is available under paid and free licenses. See the [LICENSE](https://github.com/getbouncer/cardscan-ui-android/blob/ebb299b0e1b799ec7c12aff1f535d0278d9107c1/LICENSE) file for the full license text.

### Quick summary
In short, this library will remain free forever for non-commercial applications, but use by commercial applications is limited to 90 days, after which time a licensing agreement is required. We're also adding some legal liability protections.

After this period commercial applications need to convert to a licensing agreement to continue to use this library.
* Details of licensing (pricing, etc) are available at [https://cardscan.io/pricing](https://cardscan.io/pricing), or you can contact us at [license@getbouncer.com](mailto:license@getbouncer.com).

### More detailed summary
What’s allowed under the new license:
* Free use for any app for 90 days (for demos, evaluations, hackathons, etc).
* Contributions (contributors must agree to the [Contributor License Agreement](https://github.com/getbouncer/cardscan-demo-android/blob/57102fa3e133febb6b08589185d05b3f06d5657d/Contributor%20License%20Agreement))
* Any modifications as needed to work in your app

What’s not allowed under the new license:
* Commercial applications using the license for longer than 90 days without a license agreement. 
* Using us now in a commercial app today? No worries! Just email [license@getbouncer.com](mailto:license@getbouncer.com) and we’ll get you set up.
* Redistribution under a different license
* Removing attribution
* Modifying logos
* Indemnification: using this free software is ‘at your own risk’, so you can’t sue Bouncer Technologies, Inc. for problems caused by this library

Questions? Concerns? Please email us at [license@getbouncer.com](mailto:license@getbouncer.com) or ask us on [slack](https://getbouncer.slack.com).

A licensing agreement is required to use this library.
* Details of licensing (pricing, etc) are available at [https://cardscan.io/pricing](https://cardscan.io/pricing), or you can contact us at [license@getbouncer.com](mailto:license@getbouncer.com).
