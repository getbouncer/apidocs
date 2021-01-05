---
description: >-
  Card Verify is an SDK that you can add to your app to scan credit and debit
  cards and verify that they are genuine. Use Card Verify on all suspicious
  transactions.
---

# Verifying High Risk Cards

## **The high-risk transaction UX**

![](../../.gitbook/assets/image%20%281%29.png)

When your app detects a high-risk transaction, rather than blocking this transaction you can verify that their card is genuine. From a UX perspective, Bouncer provides two activities to help with this: a security explanation activity and an activity for scanning and verifying cards. Depending on the outcome of the scanning process, your app will either accept the transaction and let it proceed, or block it, optionally providing the user with other options.

