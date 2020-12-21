# Server integration guide

## Account Events
When the client performs an event that should be tracked by Bouncer, call this endpoint from your server.

These requests should be made after each of these events:
- user signup
- user login
- payment method added
- transaction performed

### Sign Up

{% tabs %}
{% tab title="Request" %}

The request must include details about the event, including the user ID and IP. The request must additionally be authenticated with your API key.

```
POST https://api.getbouncer.com/insight/v1/events/add
x-bouncer-auth: <your_api_key>

{
    "event": "user_create",
    "occurred_at": Long("<the time in MS since epoch when the event occurred>"),
    "ip_address": String("<IP address of the client performing the action>"),
    "session_id": Optional String("<the ID of the session for this user>"),
    "user_id": String("<id of the user>"),
    "user_name": Optional String("<username performing the event>"),
    "phone_number": Optional String,
    "first_name": Optional String,
    "last_name": Optional String,
    "full_name": Optional String,
    "email": Optional String,
    "shipping_address": Optional String,
    "billing_address": Optional String
}
```

{% endtab %}
{% tab title="Response" %}

```
HTTP 200 OK
{
    "result": "ok"
}
```

{% endtab %}
{% endtabs %}

### Login

{% tabs %}
{% tab title="Request" %}

The request must include details about the event, including the user ID and IP. The request must additionally be authenticated with your API key.

```
POST https://api.getbouncer.com/insight/v1/events/add
x-bouncer-auth: <your_api_key>

{
    "event": "user_login",
    "occurred_at": Long("<the time in MS since epoch when the event occurred>"),
    "ip_address": String("<IP address of the client performing the action>"),
    "session_id": Optional String("<the ID of the session for this user>"),
    "success": Boolean("<whether or not the login attempt succeeded>"),
    "failure_reason": String("<reason the login failed, one of the following: account_unknown, bad_credentials>"),
    "user_id": Optional String("<id of the user>"),
    "user_name": Optional String("<username performing the event>")
}
```

{% endtab %}
{% tab title="Response" %}

```
HTTP 200 OK
{
    "result": "ok"
}
```

{% endtab %}
{% endtabs %}

### User Update

{% tabs %}
{% tab title="Request" %}

The request must include details about the event, including the user ID and IP. The request must additionally be authenticated with your API key.

```
POST https://api.getbouncer.com/insight/v1/events/add
x-bouncer-auth: <your_api_key>

{
    "event": "user_update",
    "occurred_at": Long("<the time in MS since epoch when the event occurred>"),
    "ip_address": String("<IP address of the client performing the action>"),
    "session_id": Optional String("<the ID of the session for this user>"),
    "user_id": String("<id of the user>"),
    "user_name": Optional String("<username performing the event>"),
    "phone_number": Optional String,
    "first_name": Optional String,
    "last_name": Optional String,
    "full_name": Optional String,
    "email": Optional String,
    "shipping_address": Optional String,
    "billing_address": Optional String
}
```

{% endtab %}
{% tab title="Response" %}

```
HTTP 200 OK
{
    "result": "ok"
}
```

{% endtab %}
{% endtabs %}

### Add payment method

{% tabs %}
{% tab title="Request" %}

The request must include details about the event, including the user ID and IP. The request must additionally be authenticated with your API key.

```
POST https://api.getbouncer.com/insight/v1/events/add
x-bouncer-auth: <your_api_key>

{
    "event": "payment_method_add",
    "occurred_at": Long("<the time in MS since epoch when the event occurred>"),
    "ip_address": String("<IP address of the client performing the action>"),
    "session_id": Optional String("<the ID of the session for this user>"),
    "user_id": Optional String("<id of the user>"),
    "payment_method_id": String("<customer-defined payment method ID>"),
    "payment_method_type": String,
    "payment_gateway": String,
    "success": Boolean("<true if the payment method was added successfully>"),
    "card": Optional {
        "card_brand": Optional String,
        "card_funding": Optional String,
        "card_fingerprint": Optional String,
        "card_bin": Optional String,
        "card_last4": Optional String,
        "card_exp_month": Optional String,
        "card_exp_year": Optional String,
        "card_name": Optional String,
        "card_cvc_check": Optional String,
        "card_zip_check": Optional String,
        "card_address": Optional {
            "street1": Optional String,
            "street2": Optional String,
            "city": Optional String,
            "state": Optional String,
            "zip": Optional String,
            "country_iso": Optional String
        }
    },
    "failure_type": String,
    "decline_code": String,
    "failure_reason": String
}
```

{% endtab %}
{% tab title="Response" %}

```
HTTP 200 OK
{
    "result": "ok"
}
```

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
