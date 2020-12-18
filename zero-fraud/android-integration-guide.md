---
description: Android zero-fraud integration guide.
---

# Android integration guide

## Installation
This library is not publicly available. You should have been provided with a username and password when you contracted with Bouncer Technologies to use this library. Use those values in your `build.gradle` file to use Instrumentation in your app.

If you do not have the appropriate credentials, please contact [Support](mailto:support@getbouncer.com) and we will provide them.

```text
// Add the Bouncer repositories
repositories {
    ...
    maven {
        url "https://bouncerpaid.bintray.com/insights-android"
        credentials {
            username "<FILL_IN_YOUR_USERNAME>"
            password "<FILL_IN_YOUR_PASSWORD>"
        }
    }

    maven {
        url "https://bouncerpaid.bintray.com/cardverify-ui-zerofraud-android"
        credentials {
            username "<FILL_IN_YOUR_USERNAME>"
            password "<FILL_IN_YOUR_PASSWORD>"
        }
    }
}
```

```text
dependencies {
    implementation "com.getbouncer:cardverify-ui-zerofraud:2.0.0058"
    implementation "com.getbouncer:insights:2.0.0058"
    implementation "com.getbouncer:scan-camera:2.0.0058"
    implementation "com.getbouncer:scan-framework:2.0.0058"
    implementation "com.getbouncer:scan-payment:2.0.0058"
    implementation "com.getbouncer:scan-ui:2.0.058"
}
```

## Using Zero Fraud
Zero Fraud consists of the following parts:
1. The bouncer card scanner for adding payment methods to user accounts
1. An instrumented payment method add form
1. Instrumented account events (signup, login, payment add, transaction)

### Scanning to add a card
#### 1. Library warm up
CardVerify will download ML models for use in verifying the authenticity of payment cards. To ensure these models are downloaded by the time CardVerify runs, please make sure to call the `warmUp` method on CardVerify as early in the app flow as possible. In most cases, this can be done in the `onApplicationCreate` method of your app. Note that `warmUp` processes on a background thread and will not affect your app's startup time.

##### Adding an application lifecycle listener
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

To use Zero Fraud, invoke the `CardVerifyActivity.start`  method and add implement the `CardVerifyActivityResultHandler` to be notified when a user has successfully scanned a card or has asked to enter details manually. 

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

### Instrumenting Account Events
For better fraud accuracy, add Bouncer instrumentation to the following events for your users:
1. Account Creation
1. Account Log In
1. Add payment method
1. Use payment method

#### Tracking account creation
In your new account flow, create an `EventTracker` by calling `BouncerAccountInstrumentation.trackNewAccount`. Optionally, you can provide a unique device identifier if you have one.

```kotlin
class UserSignUpActivity : AppCompatActivity() {

    private val bouncerTracker = BouncerAccountInstrumentation.trackNewAccount()

    // ...

    private fun createUser(userName: String, password: String) {
        // create a user
        val user = MyApi.createUser(userName, password)
        bouncerTracker.recordSuccess(
            context = this,
            userId = user.id,
        )
    }
}
```

#### Tracking account login
In your login flow, create an `EventTracker` by calling `BouncerAccountInstrumentation.trackAccountLogIn`. Optionally, you can provide the user ID and unique device identifier if you have them.

```kotlin
class UserLoginActivity : AppCompatActivity() {

    private val bouncerTracker = BouncerAccountInstrumentation.trackAccountLogIn()

    // ...

    private fun userLogIn(userName: String, password: String) {
        val user = MyApi.getUserFromUserName(userName)

        // create a user
        try {
            val authToken = user.logIn(password)
            
            if (authToken != null) {
                bouncerTracker.recordSuccess(this, user.id)
                storeAuthToken(authToken)
            } else {
                bouncerTracker.recordFailure(this, user.id)
                showLoginFailed()
            }
        } catch (t: Throwable) {
            bouncerTracker.recordError(this, user.id, error = t)
            showLoginError()
        }
    }
}
```

#### Tracking adding a payment method
In your add payment method form, create an `EventTracker` by calling `BouncerAccountInstrumentation.trackAddPaymentMethod`. Optionally, you can provide the user ID and a unique device identifier if you have them.

```kotlin
class AddCardActivity : AppCompatActivity() {

    private val bouncerTracker = BouncerAccountInstrumentation.trackAddPaymentMethod()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        viewBinding.addCardButton.setOnClickListener {
            // attempt to add the payment method to the user's account

            try {
                val paymentToken = MyApi.addPaymentMethod(userId, viewBinding.cardAddWidget.cardParams)

                if (paymentToken != null) {
                    bouncerTracker.recordSuccess(this, userId)
                    storePaymentToken(paymentToken)
                } else {
                    bouncerTracker.recordFailure(this, userId)
                    showPaymentMethodAddFailed()
                }
            } catch (t: Throwable) {
                bouncerTracker.recordError(this, userId, error = t)
                showPaymentMethodAddError()
            }
        }
    }
}
```

#### Tracking using a payment method
In your checkout flow, create an `EventTracker` by calling `BouncerAccountInstrumentation.trackTransaction`. Optionally, you can provide the user ID, a unique device identifier, the amount of the transaction, and the currency of the transaction. Currency should be a 3-letter representation (e.g. `"USD"` for the U.S. Dollar).

```kotlin
class CheckOutActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val bouncerTracker = BouncerAccountInstrumentation.trackTransaction(
            transactionAmount = getTransactionAmount(),
            transactionCurrency = "AUD"  // e.g. Australian dollars
        )

        viewBinding.addCardButton.setOnClickListener {
            // attempt to add the payment method to the user's account

            try {
                val transactionSuccess = MyApi.chargePayment(userId, paymentToken)

                if (transactionSuccess) {
                    bouncerTracker.recordSuccess(this, userId)
                    showTransactionSuccess()
                } else {
                    bouncerTracker.recordFailure(this, userId)
                    showTransactionFailed()
                }
            } catch (t: Throwable) {
                bouncerTracker.recordError(this, userId, error = t)
                showTransactionError()
            }
        }
    }
}
```

## Getting a fraud risk score from Bouncer
Before allowing a transaction to proceed, your servers should query Bouncer servers with the userId, payment method identifier (e.g. stripe token), transaction amount, and transaction currency. Bouncer servers will respond with a risk score. See the [Server Integration Guide](server-integration-guide/README.md) for details.
