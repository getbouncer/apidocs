# LivenessCheck

LivenessCheck Android installation guide

The LivenessCheck flow on Android checks to make sure that the user has a
genuine physical debit or credit card in their possession, and optionally that
the card's BIN matches what you have on record.

## Requirements

Make sure that you install and initialize the CardVerify library. See the
[instructions for installing CardVerify](card_verify_android_integration.md).

## Using LivenessCheck

First, make sure that you invoke the CardVerify library's `onApplicationCreate`
handler when the application launches. This routine will initialize the
CardVeriy library and it runs in a background thread, so it won't affect your
app's startup time.

If your app doesn't already listen to application lifecycle events, you can
extend the Application object and connect it using your manifest by setting
`android:name` on your application node:

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

To use LivenessCheck, you start the flow via the `start` methohd, and get the
results via the `onActivityResult` method. Pass in the card details that you
want LivenessCheck to verify, including the `last4` of the credit card number
and the `BIN`:

**IMPORTANT:** You are responsible for deleting any images that we store and
return via the activity result. The deletion logic can be subtle due to
Activity recreation.

**IMPORTANT:** We download some of our liveness check models in the background.
You need to check if they have been downloaded and are ready to be used before
starting the LivenessCheckActivity.


```java
public void verifyCard() {
    // starts a liveness check and asks the user to scan
    // the front of the card. There is also a
    // `startCheckCardBack` method that you can use that
    // asks the user to scan the back of theh card
    if (LivenessCheck.canRunLivenessCheck(this)) {
        // alternatively you can ask LivenessCheck to store one of the images
        // that it scans using this method:
        // LivenessCheck.startCheckCardFrontAndReturnImage(this, last4, bin)
        LivenessCheck.startCheckCardFront(this, last4, bin);
    } else {
        Log.d("Payment", "Can't run liveness check");
    }
}

protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    if (LivenessCheck.isLivenessResult(requestCode)) {
       LivenessCheck.getPayload(resultCode, data, new LivenessCheckResult() {
            @Override
            public void userCanceled() {
                Log.d("Liveness", "userCanceled");
            }

            @Override
            public void success(
                @NonNull String encryptedPayload,
                @Nullable String imagePath,
                @Nullable String ocrNumber
            ) {
                // Make sure to handle the image file that we store if you
                // invoke the liveness check and ask it to return the image
                if (!TextUtils.isEmpty(imagePath)) {
                    Bitmap bitmap = BitmapFactory.decodeFile(imagePath);
                    ((ImageView) findViewById(R.id.livenessImage))
                        .setImageBitmap(bitmap);
                    (new File(imagePath)).delete();
                }

                // Optionally, do something with the scanned card information
                if (!TextUtils.isEmpty(ocrNumber)) {
                    ((TextView) findViewById(R.id.paymentCardNumber))
                        .setText(ocrNumber);
                }

                // Use the encrypted payload in a server-to-server call with
                // the Bouncer servers to check if the card that the library
                // scanned is genuine
            }

            @Override
            public void error(String imagePath) {
                if (!TextUtils.isEmpty(imagePath)) {
                    (new File(imagePath)).delete();
                }
                Log.d("Liveness", "error");
            }
        });
    }
}
```

## License

CardVerify is proprietary software (c) Bouncer Technologies 2019, all rights
reserved.
