---
description: The Bouncer Insight Android SDK integration guide.
---

# Android integration guide

## Installation

```text
dependencies {
    implementation "com.getbouncer:cardverify-ui:2.1.0001"
    implementation "com.getbouncer:insights:2.1.0001"
    implementation "com.getbouncer:scan-camera:2.1.0001"
    implementation "com.getbouncer:scan-framework:2.1.0001"
    implementation "com.getbouncer:scan-payment-base:2.1.0001"
    implementation "com.getbouncer:scan-ui:2.1.0001"

    // you must select one of the following sets of OCR and CardDetect
    // frameworks. See the above chart to see how your selection will affect
    // the size of the SDK.

    // To minimize the size of the SDK, use the following dependencies. These
    // perform slightly slower than the larger normal ML models, but will be
    // upgraded over the internet automatically for your users.
    implementation "com.getbouncer:scan-payment-ocr-minimal:2.1.0001"
    implementation "com.getbouncer:scan-payment-card-detect-minimal:2.1.0001"

    // To ensure the maximum performance of the SDK regardless of network
    // connection, but at the cost of a larger SDK, use the following
    // dependencies.
    implementation "com.getbouncer:scan-payment-ocr:2.1.0001"
    implementation "com.getbouncer:scan-payment-card-detect:2.1.0001"



    // you must select one of the following tensorflow-lite libraries. See the
    // above chart to understand how each will affect the size of your app.

    // If you're already using tensorflow lite elsewhere in your project, make
    // sure you depend on the TFLite framework.
    implementation 'org.tensorflow:tensorflow-lite:2.4.0'

    // If you need to support both ARM and x86 devices (< 1% of all android
    // devices), include this dependency.
    implementation 'com.getbouncer:tensorflow-lite:2.1.0001'

    // If you only plan to support ARM devices, use this library
    implementation 'com.getbouncer:tensorflow-lite-arm-only:2.1.0001'
}
```

## Using Bouncer Insight

Bouncer Insight for Android consists of the following parts:

1. The bouncer card scanner for adding payment methods to user accounts
2. An instrumented payment method add form
3. Instrumented user events \(signup, login\)

### Scanning to add a card

#### 1. Library warm up

CardVerify will download ML models for use in verifying the authenticity of payment cards. To ensure these models are downloaded by the time CardVerify runs, please make sure to call the `warmUp` method on CardVerify as early in the app flow as possible. In most cases, this can be done in the `onApplicationCreate` method of your app. Note that `warmUp` processes on a background thread and will not affect your app's startup time.

**Adding an application lifecycle listener**

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

#### 2. Starting the flow

To start the flow, `CardVerifyActivity` provides a `start` method which takes the required parameters and launches the flow. Be sure to include a unique ID for the user that you will use to look up the user later.

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
                userId = MyUser.id  // your unique ID for the user
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
        scanResult: ScanResult
    ) {
        // The user scanned a card. Details of the card are in the `scanResult`
        // variable.
    }

    override fun enterManually(scanId: String?) {
        // The user wants to manually enter a card, launch the manual card entry
        // activity
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
                userId = MyUser.id  // your unique ID for the user
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
        @NotNull ScanResult scanResult
    ) {
        // The user scanned a card. Details of the card are in the `scanResult`
        // variable.
    }

    @Override
    public void enterManually(@Nullable String scanId) {
        // The user wants to manually enter a card, launch the manual card entry
        // activity
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

To use Zero Fraud, invoke the `CardVerifyActivity.start` method and add implement the `CardVerifyActivityResultHandler` to be notified when a user has successfully scanned a card or has asked to enter details manually.

### Instrumenting the payment card form

Soon, Bouncer will provide a payment card entry form. Until then, Bouncer Insights is designed to use your own existing payment card entry form.

#### Using your own card entry form

If you plan to build or already have your own card entry form, you can add bouncer instrumentation to the form.

```kotlin
const val RESULT_CARD = "card_params"

class AddCardActivity : AppCompatActivity(), CoroutineScope {

    private val viewBinding by lazy { ActivityCardAddBinding.inflate(layoutInflater) }
    private val bouncerInstrumentation by lazy { BouncerInstrumentedPaymentCardForm(this) }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(viewBinding.root)

        viewBinding.addCardButton.setOnClickListener {
            // Submit instrumentation to Bouncer servers
            bouncerInstrumentation.onFormSubmit(this@AddCardActivity, "my_user_id")
            setResult(Activity.RESULT_OK, Intent().putExtra(RESULT_CARD, viewBinding.cardAddWidget.cardParams))
            finish()
        }

        try {
            // Instrument the composite stripe CardMultilineWidget by describing the contents of the widget in order
            bouncerInstrumentation.instrumentComposite(
                view = viewBinding.cardAddWidget,
                subviewOrder = listOf(
                    PaymentCardFormFieldType.CardNumber,
                    PaymentCardFormFieldType.Expiry,
                    PaymentCardFormFieldType.Cvc,
                    PaymentCardFormFieldType.PostalCode,
                ),
            )
        } catch (e: IncorrectViewCount) {
            // If the wrong number of subviews is specified, determine how many are required
            Log.e("Demo", "Invalid subview order for cardAddWidget, expected ${e.expectedViewCount}")
        }

        // Instrument an additional card holder name text field
        bouncerInstrumentation.instrument(viewBinding.cardHolderName, PaymentCardFormFieldType.CardHolderName)
    }

    override val coroutineContext: CoroutineContext = Dispatchers.Default
}
```

If you're not using the `CardVerifyActivity.warmUp()` at application start, you'll have to manually set your API key before instrumenting your payment form. You can do this by setting:

```kotlin
com.getbouncer.scan.framework.Config.apiKey = "<your_api_key_here>"
```

### Instrumenting User Events

For better fraud accuracy, add Bouncer instrumentation to the following events for your users:

1. User Creation
2. User Login

If you're not using the `CardVerifyActivity.warmUp()` at application start, you'll have to manually set your API key before instrumenting your user events. You can do this by setting:

```kotlin
com.getbouncer.scan.framework.Config.apiKey = "<your_api_key_here>"
```

#### Tracking user creation

In your user signup flow, create an `EventTracker` by calling `BouncerUserInstrumentation.trackUserCreate`. Optionally, you can provide a unique device identifier if you have one.

```kotlin
class SignUpActivity : AppCompatActivity() {

    private val bouncerTracker = BouncerUserInstrumentation.trackUserCreate()

    // ...

    private fun createUser(userName: String, password: String): Boolean {
        // create a user
        val user = MyApi.createUser(userName, password)
        if (user != null) {
            bouncerTracker.recordSuccess(
                context = this,
                userId = user.id,
                loginIdentifier = userName,
            )
        }

        return user != null
    }
}
```

#### Tracking user login

In your user login flow, create an `EventTracker` by calling `BouncerUserInstrumentation.trackUserLogin`. Optionally, you can provide the user ID and unique device identifier if you have them.

```kotlin
class UserLoginActivity : AppCompatActivity() {

    private val bouncerTracker = BouncerUserInstrumentation.trackUserLogin()

    // ...

    private fun userLogIn(userName: String, password: String): Boolean {
        // log the user in
        val user = MyApi.getUserFromUserName(userName)
        val authToken = user?.logIn(password)

        if (authToken != null) {
            bouncerTracker.recordSuccess(this, userName, user.id, metaData = null)
        } else {
            bouncerTracker.recordFailure(this, userName, user?.id, metaData = null)
            showLoginFailed()
        }

        return authToken != null
    }
}
```

## Getting a fraud risk score from Bouncer

Before allowing a transaction to proceed, your servers should query Bouncer servers with the userId, payment method identifier \(e.g. stripe token\), transaction amount, and transaction currency. Bouncer servers will respond with a risk score. See the [Server Integration Guide](server-integration-guide.md) for details.

