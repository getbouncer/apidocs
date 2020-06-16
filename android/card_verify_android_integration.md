# CardVerify

[CardVerify](https://github.com/getbouncer/cardverify-ui-android) Android installation guide

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

Unlike CardScan, the CardVerify library is not publicly available. It can be downloaded from our [BinTray](https://bintray.com/bouncerpaid/) repository. You should have been provided with a username and password when you contracted with Bouncer Technologies to use this library. Use those values in your `build.gradle` file to use CardVerify in your app.

If you do not have the appropriate credentials, please contact [Sam King](mailto:sam@getbouncer.com) and we will provide them.

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

Then add a dependency on the cardverify library in your module's build.gradle file:

```gradle
dependencies {
    implementation 'com.getbouncer:cardverify-ui:2.0.0012'
}
```

If you are still using CardScan, leave that dependency in place.

## Using

Before you can use CardVerify, you must have an API key set up through the [Bouncer API console](https://api.getbouncer.com/console). Please contact [Sam King](mailto:sam@getbouncer.com) or [Steven Liu](mailto:steven@getbouncer.com) to ensure the correct permissions have been added to your API key.

### 1. Library warmup

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
    @Override
    public void onCreate() {
        super.onCreate();
        CardVerifyActivity.warmUp(this, "<YOUR_API_KEY_HERE>");
    }
}
```

### 2. Starting the flow

To start the flow, `CardVerifyActivity` provides a `start` method which takes the required parameters and launches the flow. Pass in the payment card details that you want to verify, including the IIN (first six digits) and last four digits of the payment card.

```java
public void verifyPaymentCard(String cardNumber) {
    CardVerifyActivity.start(
        /* activity or fragment */ this,
        /* api key */ "<YOUR_API_KEY_HERE>",
        /* issuer identification number */ cardNumber.substring(0, 6),
        /* payment card last four */ cardNumber.substring(cardNumber.length() - 4)
    );
}
```

CardVerify will return an encrypted payload containing information about the payment card the user scanned. You can receive this payload using your activity or fragment's `onActivityResult` method.

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    if (CardVerifyActivity.isVerifyResult(requestCode)) {
        CardVerifyActivity.parseVerifyResult(resultCode, data, new CardVerifyActivityResultHandler() {
            @Override
            public void cardScanned(
               @Nullable String instanceId,
               @Nullable String scanId,
               @NonNull CardVerifyActivityResult scanResult,
               int payloadVersion,
               @NonNull String encryptedPayload
            ) {
                /*
                 * The user scanned a card. Details of the card are in the `scanResult` variable. Send the value in the
                 * `encryptedPayload` variable to Bouncer servers to check if the card is genuine. See the server
                 * integration documentation for details.
                 */
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

A licensing agreement is required to use this library.
* Details of licensing (pricing, etc) are available at [https://cardscan.io/pricing](https://cardscan.io/pricing), or you can contact us at [license@getbouncer.com](mailto:license@getbouncer.com).

All contributors must agree to the [CLA](https://github.com/getbouncer/cardscan-demo-android/blob/57102fa3e133febb6b08589185d05b3f06d5657d/Contributor%20License%20Agreement).
