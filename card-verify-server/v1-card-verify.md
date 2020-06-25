# Verifying a Card Verify Payload

## **POST /v1/card/verify**

### HTTP Request

Authentication Required

#### **Fields**

`payload: string` CardVerify Payload from the client side CardVerify scan

`bin: string (optional)`BIN of the challenged card

`last4: string` Last 4 of the challenged card  
  
`exp_month: string (optional)` Expiration month of the challenged card. Used to check whether the user scanned card matches the challenged card.  
  
`exp_year: string (optional)`Expiration year of the challenged card. Used to check whether the user scanned card matches the challenged card.

{% api-method method="post" host="https://api.getbouncer.com" path="/v1/card/verify" %}
{% api-method-summary %}

{% endapi-method-summary %}

{% api-method-description %}

{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-body-parameters %}
{% api-method-parameter name="" type="string" required=false %}

{% endapi-method-parameter %}

{% api-method-parameter name="" type="string" required=false %}

{% endapi-method-parameter %}

{% api-method-parameter name="last4" type="string" required=false %}

{% endapi-method-parameter %}

{% api-method-parameter name="bin" type="string" required=false %}

{% endapi-method-parameter %}

{% api-method-parameter name="payload" type="string" required=true %}
Card verify payload from the client side scan
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```

```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

```text
curl -X POST "https://api.getbouncer.com/v1/card/verify"
  -H "Content-Type: application/json"
  -H "Authorization: Bearer API_KEY"
  -d '{
    "payload"             : "ENCRYPTED_PAYLOAD",
    "last4"               : "0123",
    "bin"                 : "424242",
    "exp_month"           : "11",
    "exp_year"            : "2022"
  }'

```

