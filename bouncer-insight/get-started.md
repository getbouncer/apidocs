---
description: >-
  Bouncer Insight is a risk score for your users and transactions. It works by
  instrumenting your app to detect suspicious behavior, and then provides a risk
  score for your transactions.
---

# Get started

You can use Bouncer Insight's risk score for your [web app](javascript-beacon.md), [iOS app](ios-integration-guide.md), and [Android app](android-integration-guide.md). These client libraries instrument key parts of your app to detect suspicious behavior. Then, [your server can query Bouncer's servers](server-integration-guide/) to get a risk score for each transaction.

For iOS and Android, Bouncer Insight does include user-visible changes around the flows in your app for adding a payment card to an account. For these payments flows, you should either [default to scanning](get-started.md#the-default-to-scan-ux) a card or provide an [optional scan](get-started.md#the-optional-scan-ux) button that users can use. When they scan their card, we'll run our fraud checks and use this data when the user initiates a transaction later.

## The "default to scan" UX

![](../.gitbook/assets/image%20%282%29.png)

For the default to scan flow, the app presents the user with the VerifyCardAdd flow first. If they scan their card, the flow takes them to the standard manual entry form with the number and expiry \(optionally\) filled in. The VerifyCardAdd flow has a “enter card details manually” button that people can press to go directly to the card entry form, where they can fill in all of the card details.

## The "optional scan" UX

![](../.gitbook/assets/image%20%284%29.png)

For the optional scan flow, the app presents the user with a standard card entry form, but includes a “Scan card” button that allows users to scan cards. If successful, the scanning process fills in the card number and expiry \(optionally\) in the standard card entry form.

## Verifying high-risk transactions

At checkout, make a server-to-server API call to Bouncer's servers and we'll give you a fraud score for that transaction and the ability to [forced-verify cards](../bouncer-scan/verifying-high-risk-cards/) if you detect anything suspicious.

