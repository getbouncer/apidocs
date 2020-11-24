# Server integration guide

## Overview

When you are ready to complete a transaction on the iOS or Android SDK, the SDK will return a payload that should be verified via a server-to-server call to `/v1/card/authenticate`.

The`/v1/card/authenticate`endpoint takes in the payload and details of the  payment instrument, and returns a `CardVerifyToken` that can then be validated either immediately or at a later time using `/v1/token/validate`. This separates the responsibility of adding the card vs validating and acting on the fraud response, and gives the implementor flexibility around when to enforce it.

