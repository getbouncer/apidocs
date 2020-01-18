# CardVerify

CardVerify Android installation guide

## Contents

* [Requirements](#requirements)
* [Installation](#installation)
* [Using FraudCheck](#using-fraudcheck)
* [License](#license)

## Requirements

* Android API level 19 or higher

## Installation

We publish our library in a private bintray repository, so first you need to specify our repo in your top-level build.gradle file. Email sam@getbouncer.com to get credentials.

```gradle
// Add our repo to both the buildscript and allprojects elements
repositories {
    ...
    maven {
        url "https://bouncerpaid.bintray.com/CardVerify"
        credentials {
            username "<FILL_IN_YOUR_USERNAME>"
            password "<FILL_IN_YOUR_PASSWORD>"
        }
    }
}
````

Then add a depenendency to the cardverify library in your module's
build.gradle file:

```gradle
dependencies {
    implementation 'com.getbouncer:cardscan-base:1.0.5126'
    implementation 'com.getbouncer:cardverify:1.0.5126'
}
```

If you used to use `cardscan` you should replace it with `cardverify`.

## Using CardVerify

First, make sure that you invoke the CardVerify library's `onApplicationCreate` 
handler when the application launches. This routine will initialize the
CardVeriy library and it runs in a background thread, so it won't affect your
app's startup time.

If your app doesn't already listen to application lifecycle events,
you can extend the Application object and connect it using your
manifest by setting `android:name` on your application node:

```xml
<application
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:name=".MyApplication">
    <activity android:name=".PlacingOrder"></activity>
    <activity android:name=".EnterCard" />
    <activity android:name=".LaunchActivity">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>
</application>
```

And in your application class:

```java
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        // Application initialization

        CardVerify.onApplicationCreate(this, YOUR_API_KEY);
    }
}
```

To use CardVerify, you start the flow and get the results via the
`onActivityResult` method. Pass in the card details that you want
CardVerify to verify, including the `last4` of the credit card number,
the `expiry`, and the card network (using CardVerify's
`CreditCard.Network` data type):

```java
public void verifyCard() {
    CardVerify.start(this, last4, expiry, network);
}

protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    if (CardVerify.isVerifyResult(requestCode)) {
        CardVerify.onResult(resultCode, data, new CardVerifyResult() {
            @Override
            public void userCanceled() {
                Log.d(TAG, "The user pressed the back button");
            }

            @Override
            public void userScannedSameCard(@NonNull CreditCard scannedCard, @NonNull String encryptedPayload) {
                // The user scanned the card and the scanned card details are in the `scannedCard` variable.
		// The user scanned the same card that you passed into the challenge, use the encryptedPayload
		// in a server-to-server call to check if the card is genuine.
            }

            @Override
            public void userScannedDifferentCard(@NonNull CreditCard scannedCard, @NonNull String encryptedPayload) {
                // The user scanned the card and the scanned card details are in the `scannedCard` variable.
		// The user scanned a different card that you passed into the challenge, use the encryptedPayload
		// in a server-to-server call to check if the card is genuine.
            }

            @Override
            public void error(@NonNull CreditCard scannedCard) {
                // An error occurred when generating the encrypted payload but the scan succeeded. You can see the scanned
		// card details in the `scannedCard` variable but we are unable to run the verification checks on this card.
		//
		// This should be rare.
            }

            @Override
            public void fatalError() {
		// An unexpected error occurred and we were unable to scan the card.
		//
		// This should be rare.
            }
        });
    }
}
```

## License

CardVerify is proprietary software (c) Bouncer Technologies 2019, all rights reserved.
