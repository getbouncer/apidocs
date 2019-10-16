# FraudCheck

FraudCheck Android installation guide

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
    implementation 'com.getbouncer:base:1.0.5102'
    implementation 'com.getbouncer:cardverify:1.0.5102'
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

        CardVerify.onApplicationCreate(this);
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
    CardVerify.start(this, last4, expiry, network, apiKey);
}

protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    if (CardVerify.isVerifyResult(requestCode)) {
        CardVerify.onResult(resultCode, data, new CardVerifyResult() {
            @Override public void userCanceled() {
                // The user pressed on the back button without passing the challenge
            }

            @Override public void verifiedCard(@NonNull CreditCard scannedCard) {
                // The user scanned a card that has the last4 that matches the last4 that you
		// passed in and it passed our integrity checks, let the transaction proceed
		
		// pass the `fraudCheckToken` returned by CardVerify to the server to double check the
		// transaction using a server-to-server call to Bouncer's API server
		String fraudCheckToken = scannedCard.fraudCheckToken
            }

            @Override public void verifiedDifferentCard(@NonNull CreditCard scannedCard) {
                // The user scanned a card and it passed our integrity checks, but the last4 didn't match.
                // The most common option is to use the card details passed back via `scannedCard` to let
                // the user add the new card and pay for the transaction using it.
   
                // IMPORTANT: make sure that the user doesn't change the card number when they add the
                // new card. If they change the card number after the scan, it will invalidate the
                // `fraudCheckToken`
   
                // pass the `fraudCheckToken` returned by CardVerify to the server to double check the
                // transaction using a server-to-server call to Bouncer's API server
		String fraudCheckToken = scannedCard.fraudCheckToken
            }

            @Override public void failedVerification(@NonNull CreditCard scannedCard) {
                // the user scanned a card (details are in the `scannedCard` object) but it failed our
                // integrity check.
                // The most common option is to ask they to try scanning again or to try a different card
                // but this is likely to be a fraudster
            }

            @Override public void fatalError() {
                // There was an unexpected error, this should be rare
                // The most common option is to handle this like the user pressed the "back" button
            }
        });
    }
}
```

## License

CardVerify is proprietary software (c) Bouncer Technologies 2019, all rights reserved.
