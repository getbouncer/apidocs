# Server integration guide

## Account Events
When the client performs an event that should be tracked by Bouncer, call this endpoint from your server.

These requests should be made after each of these events:
- user signup
- user login
- payment method added
- transaction performed

{% tabs %}
{% tab title="Request" %}

The request must include details about the event, including the user ID and IP. The request must additionally be authenticated with your API key.

```
POST https://api.getbouncer.com/insight/v1/events/add
x-bouncer-auth: <your_api_key>

{
    "event_name": String("<One of the following: new_account, account_login, add_payment_method, transaction>"),
    "event_result": String("<One of the following: success, failure, error>"),
    "user_id": String("<id of the user>"),
    "client_ip": String("<IP address of the client performing the action>"),
    "metadata": Object of String:Any
}
```

The `metadata` field should contain information about the event. Each event expects different metadata:

### Signup
no additional metadata (field can be null)

### Login
no additional metadata (field can be null)

### Add payment method
Metadata should contain:

```json
{
  "instrument_last_four": "<payment method last 4 digits>",
  "instrument_iin": "<payment method first 6 digits>",
  "instrument_token": "<stripe token for the payment method>",
  "customer_id": "<stripe customer id>"
}
```

### Performing a transaction with a payment method
Metadata should contain:

```json
{
  "instrument_last_four": "<payment method last 4 digits>",
  "instrument_iin": "<payment method first 6 digits>",
  "instrument_token": "<stripe token for the payment method>",
  "customer_id": "<stripe customer id>"
}
```

{% endtab %}
{% tab title="Response" %}

The response will include a score from Bouncer and a recommendation to challenge or allow the transaction.

```
HTTP 200 OK
{
    "result": "ok"
}
```

`challenge_recommended` will be a boolean indicating whether bouncer recommends challenging the transaction. `fraud_risk_score` will be a float between 0 and 1 (inclusive) that indicates how fraudulent Bouncer believes this transaction to be.

{% endtab %}
{% endtabs %}

## Tokenizing a Payment Card
When the client adds a new payment method and that payment method is tokenized (e.g. through Stripe), call this endpoint from your server. 

{% tabs %}
{% tab title="Request" %}

The request must include details about the payment instrument, including the payment method token. The request must additionally be authenticated with your API key.

```
POST https://api.getbouncer.com/insight/v1/tokenize
x-bouncer-auth: <your_api_key>

{
    "user_id": String("<tokenizing_user_id>"),
    "customer_id": String("<customer_stripe_id>"),
    "instrument_token": String("<payment_instrument_stripe_token>"),
    "instrument_last_four": String("<last_four_digits_of_payment_instrument>"),
    "instrument_iin": Optional String("<first_six_digits_of_payment_instrument>"),
    "client_ip": Optional String("<ip_address_of_client_adding_payment_instrument>")
}
```

{% endtab %}
{% tab title="Response" %}

The response will include a score from bouncer and a recommendation to challenge or allow the transaction.

```
HTTP 200 OK
{
    "result": "ok"
}
```

`challenge_recommended` will be a boolean indicating whether bouncer recommends challenging the transaction. `fraud_risk_score` will be a float between 0 and 1 (inclusive) that indicates how fraudulent Bouncer believes this transaction to be.

{% endtab %}
{% endtabs %}

## Getting a fraud risk score
When you are ready to complete a transaction on the iOS or Android SDK make a server-to-server call from your servers to Bouncer to validate the transaction.

{% tabs %}
{% tab title="Request" %}

The request must include details about the transaction, including the payment instrument used, and the amount to be charged. The request must additionally be authenticated with your API key.

```
POST https://api.getbouncer.com/insight/v1/score
x-bouncer-auth: <your_api_key>

{
    "user_id": String("<transacting_user_id>"),
    "transaction_amount": Float(<amount_being_charged>),
    "transaction_currency": String("<three_character_currency_code>"),
    "customer_id": String("<customer_stripe_id>"),
    "instrument_token": String("<payment_instrument_stripe_token>"),
    "instrument_last_four": String("<last_four_digits_of_payment_instrument>"),
    "instrument_iin": Optional String("<first_six_digits_of_payment_instrument>"),
    "client_ip": Optional String("<ip_address_of_client_making_transaction>"),
    "product_sku": Optional Array of Strings("<SKUs of the product purchased>")
}
```

{% endtab %}
{% tab title="Response" %}

The response will include a score from bouncer and a recommendation to challenge or allow the transaction.

```
HTTP 200 OK
{
    "challenge_recommended": true,
    "fraud_risk_score": 0.4
}
```

`challenge_recommended` will be a boolean indicating whether bouncer recommends challenging the transaction. `fraud_risk_score` will be a float between 0 and 1 (inclusive) that indicates how fraudulent Bouncer believes this transaction to be.

{% endtab %}
{% endtabs %}
