---
description: Android zero-fraud integration guide.
---

# Android integration guide

## Installation

This library is not publicly available. You should have been provided with a username and password when you contracted with Bouncer Technologies to use this library. Use those values in your `build.gradle` file to use Instrumentation in your app.

If you do not have the appropriate credentials, please contact [Support](mailto:support@getbouncer.com) and we will provide them.

```text
// Add the Bouncer repo to both the buildscript and allprojects elements
repositories {
    ...
    maven {
        url "https://bouncerpaid.bintray.com/instrumentation-android"
        credentials {
            username "<FILL_IN_YOUR_USERNAME>"
            password "<FILL_IN_YOUR_PASSWORD>"
        }
    }
}
```

```text
dependencies {
    implementation "com.getbouncer:scan-framework:2.0.0057"
    implementation 'com.getbouncer:insights:2.0.0057'
}
```

## Using Zero Fraud

To use Zero Fraud, invoke the `CardVerifyActivity.start`  method and add implement the `CardVerifyActivityResultHandler` to be notified when a user has successfully scanned a card or has asked to enter details manually. 

### Using your own card entry form

If you plan to build or already have your own card entry form, you can add bouncer instrumentation to the form.

```kotlin
const val RESULT_EVENTS = "instrumentation_result"
const val RESULT_NUMBER = "card_number"

class AddCardActivity : AppCompatActivity(), CoroutineScope {

    private val viewBinding by lazy { ActivityCardAddBinding.inflate(layoutInflater) }
    private val bouncerInstrumentation by lazy { BouncerInstrumentedPaymentCardForm(this) }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(viewBinding.root)

        viewBinding.addCardButton.setOnClickListener {
            // Submit instrumentation to Bouncer servers and get a token for future reference
            bouncerInstrumentation.onFormSubmit(this@AddCardActivity, "my_user_id") { bouncerInstrumentationToken ->
                setResult(
                    Activity.RESULT_OK,
                    Intent()
                        .putExtra(RESULT_EVENTS, bouncerInstrumentationToken)
                        .putExtra(RESULT_NUMBER, viewBinding.cardAddWidget.card?.number)
                )
                finish()
            }

            displayProgress()
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

    private fun displayProgress() {
        // display a loading spinner
    }

    override val coroutineContext: CoroutineContext = Dispatchers.Default
}
```

## Generating a Card Authentication Payload

For each transaction, to authenticate the payment method that you plan to use, you must generate a `bouncerInstrumentationToken` from the Bouncer SDK, pass this to your server, and then make a server-to-server call to Bouncer to get the authentication result.

**Note:** If the authentication result shows a failure to authenticate the card, you should put the user through a forced scan flow using [Card Verify](../card-verify/get-started.md) or pass the authentication result into your own decision engine to make a more refined decision.

This token is generated from the `onFormSubmit` method of the instrumentation.
