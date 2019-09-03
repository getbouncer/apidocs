# FraudCheck API

After the client library scans a card, it returns an encrypted fraud payload. You can send this encrypted fraud payload to the `/card/fraud_check` endpoint to check for signs of abuse.

To use this api you need to have an `API_secret` from Bouncer to authenticate your API request and a valid `fraud_payload`:

```bash
curl -X POST https://api.getbouncer.com/v1/card/fraud_check \
    -H "Authorization: Bearer API_secret" \
    -d '{"fraud_check_token": "TOKEN_FROM_CLIENT", "last4": "4242"}'
```

And the API returns a `fraud_check` object indicating either (1) that the scan was valid or (2) the scan was suspicious and why it was suspicious.

```json
// for valid payloads
{
    "fraud_check": {
        "result": "success",
        "error_code": null,
        "error_message": null
    }
}
```

For suspicious payloads, the `fraud_check` object includes both an `error_code` and an `error_message` to provide information about the failure:

| error_code       | error_message                                                                          |
|------------------|--------------------------------------------------------------------------------------- |
| card_mismatch    | The card image is inconsistent with expected cards of this type.                         |
| scan_limit       | The user has scanned too many cards this month on this device                          |
| bot_detected     | Detected bot environment on rooted phone, script, or manipulated hardware                  |
| suspicious_interactions  | The way the user interacts with the challenge is inauthentic or associated with fraud |
| image_tampering  | The card image shows signs of tampering                                                |
