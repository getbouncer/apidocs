# Server integration guide

## Overview
When you are ready to complete a transaction on the iOS or Android SDK make a server-to-server call from your servers to Bouncer to validate the transaction.

{% tabs %}
{% tab title="Request" %}

The request must include details about the transaction, including the payment instrument used, and the amount to be charged. The request must additionally be authenticated with your API key.

```
POST https://api.getbouncer.com/v1/transaction/score
x-bouncer-auth: <your_api_key>

{
    "user_id": String("<transacting_user_id>"),
    "transaction_amount": Float(<amount_being_charged>),
    "transaction_currency": String("<three_character_currency_code>"),
    "instrument_last_four": String("<last_four_digits_of_payment_instrument>"),
    "instrument_iin": Optional String("<first_six_digits_of_payment_instrument>"),
    "transaction_ip": Optional String("<ip_address_of_client_making_transaction>")
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
