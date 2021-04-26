# Verifying a Card Verify Payload

## **POST `/v1/card/verify`**

### HTTP Request

**Authentication Required**

On a completed Card Verify scan using the iOS or Android SDKs, the SDK will return a payload. The implementor should then make a server-to-server call to this `/v1/card/verify` endpoint with the payload and details about the challenged card. This endpoint will then return a `CardVerifyToken`, which can be used to retrieve validation results at a later point. This allows the implementor to issue a Card Verify scan at an earlier point in the user journey \(e.g. payment card add\), store the `CardVerifyToken`, and validate that token at a later point \(e.g. on a risky transaction\).

{% tabs %}
{% tab title="Request" %}
### Body Parameters

| parameter | type | required | description |
| :--- | :--- | :--- | :--- |
| `payload` | `string` | yes | CardVerify Payload from the client side CardVerify scan |
| `payload_version` | `int` | no | The version of the payload from the client CardVerify scan. If omitted, this defaults to `1` |
| `bin` | `string` | no | BIN of the challenged card |
| `last4` | `string` | yes | Last 4 of the challenged card |
| `exp_month` | `string` | no | Expiration month of the challenged card. Used to check whether the user scanned card matches the challenged card. |
| `exp_year` | `string` | no | Expiration year of the challenged card. Used to check whether the user scanned card matches the challenged card. |

### Sample Request

```bash
curl -X POST "https://api.getbouncer.com/v1/card/verify"
  -H "Content-Type: application/json"
  -H "Authorization: Bearer API_KEY"
  -d '{
    "payload"             : "ENCRYPTED_PAYLOAD",
    "payload_version"     : 1,
    "last4"               : "0123",
    "bin"                 : "424242",
    "exp_month"           : "11",
    "exp_year"            : "2022"
  }'
```
{% endtab %}

{% tab title="Response" %}
## Response Body

Returns a `CardVerifyToken` object

### `CardVerifyToken`

`token: string` A unique ID representing a CardVerify scan. Can be exchanged for a `TokenResponse` object with a [`/v1/token/validate`](validating-a-card-verify-token.md) call.

```bash
{
  "token": "AAAAAA-BBBBBB-CCCCCC-DDDDDD"
}
```
{% endtab %}
{% endtabs %}

## **POST `/v1/card/verify_web`**

### HTTP Request

**Authentication Required**

On a completed scan using the web SDK, the SDK will return an array of strings in the `verificationResults` callback parameter. The implementor should then make a server-to-server call to this `/v1/card/verify_web` endpoint with the payload and details about the challenged card. This endpoint will then return a `CardVerifyToken`, which can be used to retrieve validation results at a later point. This allows the implementor to issue a verification scan at an earlier point in the user journey \(e.g. payment card add\), store the `CardVerifyToken`, and validate that token at a later point \(e.g. on a risky transaction\).

{% tabs %}
{% tab title="Request" %}
### Body Parameters

| parameter | type | required | description |
| :--- | :--- | :--- | :--- |
| `verification_results` | `string` | yes | The `verificationResult` from the SDK callback |
| `card_challenged` | `CardDetails` | no | Details about the card that the user was expected to scan |
| `ocr_result` | `CardDetails` | yes | Details about the card that was scanned |

Card details have the following parameters:

| parameter | type | required | description |
| :--- | :--- | :--- | :--- |
| `bin` | `string` | no | BIN \(first 6 digits\) of the card |
| `last4` | `string` | yes | Last 4 digits of the card |
| `exp_month` | `string` | no | Expiration month of the card. Used to check whether the user scanned card matches the challenged card. |
| `exp_year` | `string` | no | Expiration year of the card. Used to check whether the user scanned card matches the challenged card. |
| `card_name` | `string` | no | Name of the card holder. Used to check whether the user scanned card matches the challenged card. |

### Sample Request

```bash
curl 'https://api.getbouncer.com/v1/card/verify_web' \
  -H 'x-bouncer-auth: qOJ_fF-WLDMbG05iBq5wvwiTNTmM2qIn' \
  -H 'content-type: application/json' \
  --data-raw '{
    "ocr_result":{
      "bin":"424242",
      "last4":"0123",
      "exp_month": "11",
      "exp_year": "2024",
      "card_name": "John Doe"
    },
    "card_challenged": {
      "bin": "424242",
      "last4": "0123",
      "exp_month": "11",
      "exp_year": "2024",
      "card_name": "John Doe"
    },
    "verification_results":"<base64_encoded_verification_results>"
  }'
```
{% endtab %}

{% tab title="Response" %}
## Response Body

Returns a `CardVerifyToken` object

### `CardVerifyToken`

`token: string` A unique ID representing a CardVerify scan. Can be exchanged for a `TokenResponse` object with a [`/v1/token/validate`](https://docs.google.com/document/d/1zPc-20khzrr0VZ5gcohaso7JYx9MChHk1i5sahyOfpo/edit#) call.

```bash
{
  "token": "AAAAAA-BBBBBB-CCCCCC-DDDDDD"
}
```
{% endtab %}
{% endtabs %}

