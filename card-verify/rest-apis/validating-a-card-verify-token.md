# Validating a Card Verify Token

## **POST `/v1/token/validate`**

The `/v1/token/validate` endpoint accepts a CardVerifyToken generated from `/v1/card/verify` endpoint and returns whether the Card Verify attempt corresponding to that token is valid or not. On invalid requests, the response will contain a list of failure\_reasons around why Bouncer believes this Card Verify scan to be suspicious.

### HTTP Request

**Authentication Required**

{% tabs %}
{% tab title="Request" %}
### **Body Parameters**

`token: string` CardVerifyToken from the `/v1/card/verify` endpoint

### **Sample Request**

```bash
curl -X POST "https://api.getbouncer.com/v1/token/validate"
  -H "Content-Type: application/json"
  -H "Authorization: Bearer API_KEY"
  -d '{
    "token"             : "CARD_VERIFY_TOKEN"
  }'
```
{% endtab %}

{% tab title="Response" %}
## **Response Body**

Returns a `TokenValidate` response JSON object

#### `TokenValidate`

`token_valid: boolean` Whether this is a legitimate token belonging to a scan. If this is false, then this token is likely forged

`card_verified: boolean` Whether Bouncer believes this scan is a legitimate card on a legitimate device. False values represents Bouncer believing this is a high risk scan. If `card_verified` is true, `failure_reasons` will be a non-empty list.

`card_verify_attempt_at: string` [ISO8601](https://www.w3.org/TR/NOTE-datetime) format of the timestamp when the scan was recorded by Bouncer servers

`failure_reasons: list[string])` Contains a list of every reason Bouncer believes this scan was illegitimate. Only contains values if card\_verified or token\_valid is false. Full list of possible failure reasons:

* `bin_mismatch` Present if there is a card design mismatch between what we expect from the BIN of the card challenged and the actual card that we scanned. On a Card Verify scan, we extract various design features on the card, like the presence and location of the Visa / MC logo, the chip, bank logo, etc to determine whether these design features match what we expect from the BIN of the card challenged.
* `screen_detected` Present if the card scanned does not appear to be a physical card. We check for whether the card appears to be a recaptured image on a screen \(i.e. the user is scanning a picture of a card on another phone, or a Google Doc\) or a card image that is printed on paper.
* `card_number_mismatch` Present if the BIN \(if provided in `/v1/card/verify`\) and the last4 of the card that was challenged do not match the output of the corresponding scan.
* `tampered_request` Present if the request payload has been tampered with or is corrupt. For example, a man-in-the-middle attack that attempts to modify the request payload will trigger this failure reason.

### Response Examples

Successful Scan

```javascript
{
  "token_valid": true,
  "card_verified": true,
  "card_verify_attempt_at": "2019-10-23T00:48:07+0000",
  "failure_reasons": []
}
```

Invalid Token

```javascript
{
  "token_valid": false,
  "card_verified": false,
  "card_verify_attempt_at": "2019-10-23T00:48:07+0000",
  "failure_reasons": ["TOKEN_INVALID"]
}
```

Card not verified

```javascript
{
  "token_valid": true,
  "card_verified": false,
  "card_verify_attempt_at": "2019-10-23T00:48:07+0000",
  "failure_reasons": [
    "bin_mismatch",
    "card_number_mismatch",
    "screen_detected",
    "tampered_request"
  ]
}
```
{% endtab %}
{% endtabs %}

