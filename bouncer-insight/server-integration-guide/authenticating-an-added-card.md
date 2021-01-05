# Authenticating an added card

## **POST `/v1/card/authenticate`**

### HTTP Request

**Authentication Required**

On a completed Card Verify scan using the iOS or Android SDKs, the SDK will return a payload. The implementor should then make a server-to-server call to this `/v1/card/authenticate` endpoint with the payload and details about the challenged card. This endpoint will then return a `CardVerifyToken`, which can then be validated at a later point. This allows the implementor to issue a Card Verify scan at an earlier point in the user journey \(e.g. payment card add\), store the `CardVerifyToken`, and validate that token at a later point \(e.g. on a risky transaction\).

{% tabs %}
{% tab title="Request" %}
### Body Parameters

| parameter | type | required | description |
| :--- | :--- | :--- | :--- |
| `payload` | `string` | yes | CardVerify Payload from the client side CardVerify scan |
| `payload_version` | `int` | no | The version of the payload from the client CardVerify scan. If omitted, this defaults to `1` |
| `bin` | `string` | no | BIN of the payment card for this transaction |
| `last4` | `string` | yes | Last 4 of the payment card for this transaction |
| `exp_month` | `string` | no | Expiration month of the challenged card. Used to check whether the user scanned card matches the challenged card. |
| `exp_year` | `string` | no | Expiration year of the challenged card. Used to check whether the user scanned card matches the challenged card. |

### Sample Request

```bash
curl -X POST "https://api.getbouncer.com/v1/card/authenticate"
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
{% endtabs %}

