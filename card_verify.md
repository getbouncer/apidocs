# CardVerify APIs

## Bouncer SDK to Server Validation
When the client library verifies a card, it sends to the Bouncer Servers an encrypted payload containing details of the scan for further validation. The server responds with whether the scan failed, high level reason codes for the failure, and a token that can be used by the customer's servers to validate that the Bouncer SDK to server call was indeed genuine, and for more detailed failure reasons regarding the scan.

To give customers control over this call, the Bouncer SDK supports proxying this request through the customer's servers.

Here is a sample API request that the Bouncer SDK would make to the server. 

```bash
curl -X POST https://api.getbouncer.com/v1/card/verify \
    -H "X-Bouncer-Auth: secret" \
    -d '{"payload": "ENCRYPTED_PAYLOAD", "timestamp_ms": timestamp}'
```

The API returns a response indicating whether the card is successfully verified, high level failure reasons if it isn't, and a token that should be used by the customer's servers to verify the authenticity of this response.
```json
{
  "verified": true,
  "token": "VALIDATION_TOKEN",
  "failure_reasons": []
}
```

The failure codes here are intentionally high level, as they are passed back to the client, which may be compromised.

| failure_code       | failure description                                                                          |
|------------------|--------------------------------------------------------------------------------------- |
| verification_failure    | The card image is tampered with, or otherwise inconsistent with expected cards of this type.   |
| generic       | The request failed for other reasons, including suspicious interactions, device rate limits, compromised device, etc.                          |


## Token Validation
After the Bouncer SDK verifies a card, it returns a token that can be used to fetch detailed results from the request. You can send this token to the `/v1/token/verify` endpoint to check for signs of tampering, such as MITM-ing the Bouncer SDK and Server request.

To use this api you need to have an `API_secret` from Bouncer to authenticate your API request and a valid `token` from the Bouncer SDK after the CardVerify flow:

```bash
curl -X POST https://api.getbouncer.com/v1/token/verify \
    -H "Authorization: Bearer API_secret" \
    -d '{"token": "TOKEN_FROM_CLIENT"}'
```

The API returns a response indicating where the scan was valid, whether the card was successfully verified, and in the case of failure, why the card was suspicious it was suspicious.

```json
// for valid payloads
{
    "token_valid": true,
    "card_verified": true,
    "failure_reasons": [],
    "verification_attempt_at_ms: 
}
```

For suspicious payloads, the response object will include a populated list of `failure_reasons`, containing all of the reasons this scan failed:

| failure_code       | failure description                                                                          |
|------------------|--------------------------------------------------------------------------------------- |
| card_mismatch    | The card image is inconsistent with expected cards of this type.                         |
| scan_limit       | The user has scanned too many cards this month on this device                          |
| bot_detected     | Detected bot environment on rooted phone, script, or manipulated hardware                  |
| suspicious_interactions  | The way the user interacts with the challenge is inauthentic or associated with fraud |
| image_tampering  | The card image shows signs of tampering                                                |
| invalid_token    | The token is invalid, suggesting that the request was tampered
