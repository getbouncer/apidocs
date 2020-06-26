# Server Integration

## Overview

On a completed Card Verify scan using the iOS or Android SDK, the SDK will return a payload that should be verified via a server-to-server call to`/v1/card/verify`. 

The`/v1/card/verify`endpoint takes in the payload and details of the challenged payment instrument, and returns a `CardVerifyToken` that can then be validated either immediately or at a later time using `/v1/token/validate`. This separates the responsibility of scanning the card vs validating and acting on the fraud response, and gives the implementor flexibility around when to issue the Card Verify challenge and when to enforce it. 

For example, the implementor may decide to challenge the user early on in the user lifecycle during card add, and only decide to validate the token when the user attempts to make a risky transaction. 

Alternatively, the implementor may decide to issue the Card Verify challenge at the time of a risky transaction, obtain the payload, and call `/v1/card/verify` and `/v1/token/validate` in succession to immediately get a decision on the challenged card.

`/v1/token/validate`details with the payload and details about the challenged card. 

