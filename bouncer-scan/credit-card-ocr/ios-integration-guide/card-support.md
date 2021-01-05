---
description: >-
  This document describes how to support new types of cards that are not
  available in the base version.
---

# Card support guide

## Currently Supported Cards

Cards have a variety of properties that can be used to recognize them, this includes the IIN range, the pan length, and the CVC length. These values correspond to the first six numbers on a card, the length of a credit card number, and the length of the security code respectively.

### Existing Support

The CardVerify SDK supports the following cards:

| Card Issuer | IIN range |
| :--- | :--- |
| American Express | 340000 - 349999 |
| American Express | 370000 - 379999 |
| Diners Club | 300000 - 305999 |
| Diners Club | 309000 - 309999 |
| Diners Club | 360000 - 369999 |
| Diners Club | 380000 - 399999 |
| Discover | 601100 - 601199 |
| Discover | 640000 - 659999 |
| JCB | 350000 - 359999 |
| UnionPay | 620000 - 629999 |
| MasterCard | 222100 - 222999 |
| MasterCard | 223000 - 229999 |
| MasterCard | 230000 - 272099 |
| MasterCard | 500000 - 559999 |
| MasterCard | 670000 - 679999 |
| Visa | 400000 - 499999 |

## Supporting New Cards

You may need to support cards which do not come bundled with the CardVerify SDK by default, this can be done with the addition of Regional Issuers.

### Adding Regional Issuers

Issuer data can be expanded via the following method:

```kotlin
CreditCardUtils.prefixesRegional += ["123456"]
```

Where additionally supported prefixes are added.

