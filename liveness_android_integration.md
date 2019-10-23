# LivenessCheck

LivenessCheck Android installation guide

The LivenessCheck flow on Android checks to make sure that the user
has a genuine physical debit or credit card on them, and optionally
that that card's BIN matches what you have on record.

## Requirements

Make sure that you install and initialize the CardVerify library. You
can find [instructions for installing CardVerify here](card_verify_android_integration.md).

## Using LivenessCheck

First, make sure that you invoke the CardVerify library's
`onApplicationCreate` handler when the application launches. This
routine will initialize the CardVeriy library and it runs in a
background thread, so it won't affect your app's startup time.

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

To use LivenessCheck, you start the flow via the `start` methohd, and
get the results via the `onActivityResult` method. Pass in the card
details that you want LivenessCheck to verify, including the `last4` of
the credit card number and the `BIN`:

**IMPORTANT:** When you process the results in the `onActiveResult`
method the SDK will make a non-blocking API call and will only invoke
the `response` and `error` methods after the API call returns. _Please
make sure that you account for the potential network round trip in
your user experience._

**IMPORTANT:** You are responsible for deleting any images that we store
and return via the activity result. The deletion logic can be subtle
due to Activity recreation.

**IMPORTANT:** We download some of our liveness check models in the
background. You need to check if they have been downloaded and are
ready to be used before starting the LivenessCheckActivity.


```java
public void verifyCard() {
    // starts a liveness check and asks the user to scan
    // the front of the card. There is also a
    // `startCheckCardBack` method that you can use that
    // asks the user to scan the back of theh card
    if (LivenessCheck.canRunLivenessCheck(this)) {
        LivenessCheck.startCheckCardFront(this, last4, bin);
    } else {
        Log.d("Payment", "Can't run liveness check");
    }
}

protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    if (LivenessCheck.isLivenessResult(requestCode)) {
        LivenessCheck.postApiAndGetResult(this, resultCode, data, new LivenessCheckResult() {
            @Override
            public void userCanceled() {
	        // the user pressed the "cancel" button or
		// the Android back button
                Log.d("Liveness", "userCanceled");
            }

            @Override
            public void response(@NonNull String token, @NonNull String imagePath) {
                // The user finished the flow and the API call to Bouncer's
                // server returned. Store the `token` and send it to your
                // server for a server-to-server call to see if the scan
                // passed the Bouncer security checks.
                Bitmap bitmap = BitmapFactory.decodeFile(imagePath);
                ((ImageView) findViewById(R.id.livenessImage)).setImageBitmap(bitmap);
                (new File(imagePath)).delete();
                // send the token to your server to check the result
            }

            @Override
            public void error(String imagePath) {
                // There was a generic error when trying to call the Bouncer
                // server's API.
                if (!TextUtils.isEmpty(imagePath)) {
                    (new File(imagePath)).delete();
                }
            }

            @Override
            public void fatalError()
                // There was an exception in the core library that Bouncer
                // can't overcome. This should be rare / never happen.
                Log.d("Liveness", "fatalError");
            }
        });
    }
}
```

## License

CardVerify is proprietary software (c) Bouncer Technologies 2019, all
rights reserved.
