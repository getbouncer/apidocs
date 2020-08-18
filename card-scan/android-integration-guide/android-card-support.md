---
description: >-
  This document describes how to support new types of cards
  that are not available in the base version.
---

# Android card support guide

## Contents

* [Currently Supported Cards](android-card-support.md#currently-supported-cards)
* [Supporting New Cards](android-card-support.md#supporting-new-cards)
* [Formatting](android-card-support.md#formatting)

## Currently Supported Cards

Cards have a variety of properties that can be used to recognize them, this includes the IIN range, the pan length, and the CVC length. These values correspond to the first six numbers on a card, the length of a credit card number, and the length of the security code respectively.

### Existing Support

The CardScan SDK supports the following cards:

| Card Issuer | IIN range | Pan Length | CVC Length |
| :--- | :--- | :--- | :--- |
| American Express | 340000 - 349999 | 15 | 4 |
| American Express | 370000 - 379999 | 15 | 4 |
| Diners Club | 300000 - 379999 | 16 - 19 | 3 |
| Diners Club | 309500 - 309599 | 16 - 19 | 3 |
| Diners Club | 360000 - 369999 | 14 - 19 | 3 |
| Diners Club | 380000 - 399999 | 16 - 19 | 3 |
| Discover | 601100 - 601199 | 16 - 19 | 3 |
| Discover | 622126 - 622925 | 16 - 19 | 3 |
| Discover | 624000 - 626999 | 16 - 19 | 3 |
| Discover | 628200 - 628899 | 16 - 19 | 3 |
| Discover | 640000 - 659999 | 16 - 19 | 3 |
| JCB | 352800 - 358999 | 16 - 19 | 3 |
| UnionPay | 620000 - 629999 | 16 - 19 | 3 |
| UnionPay | 810000 - 819999 | 16 - 19 | 3 |
| MasterCard | 222100 - 272099 | 16 | 3 |
| MasterCard | 510000 - 559999 | 16 | 3 |
| MasterCard | 500000 - 509999 | 16 - 19 | 3 |
| MasterCard | 560000 - 699999 | 16 - 19 | 3 |
| MasterCard | 675900 - 675999 | 16 - 19 | 3 |
| MasterCard | 676770 - 676770 | 16 - 19 | 3 |
| MasterCard | 676774 - 676774 | 16 - 19 | 3 |
| Visa | 400000 - 499999 | 16 - 19 | 3 |

## Supporting New Cards

You may need to support cards which do not come bundled with the CardScan SDK by default, this can be done with the addition of a Custom Issuer.

### Issuer Data

Issuer-related data is stored in a static list found in [Github](https://https://github.com/getbouncer/cardscan-android/blob/master/scan-payment/src/main/java/com/getbouncer/scan/payment/card/PaymentCardUtils.kt)

### Adding New Issuer Data

Issuer data can be expanded via the following method:

```kotlin
fun supportCardIssuer(
    iins: IntRange,
    cardIssuer: CardIssuer,
    panLengths: List<Int>,
    cvcLengths: List<Int>,
    validationFunction: PanValidator = LengthPanValidator + LuhnPanValidator
)
```

#### CardIssuer Objects

Card issuers are represented via the [CardIssuer object](https://https://github.com/getbouncer/cardscan-android/blob/master/scan-payment/src/main/java/com/getbouncer/scan/payment/card/CardIssuer.kt) which is also shown below.

```kotlin
sealed class CardIssuer(open val displayName: String) {
    object AmericanExpress : CardIssuer("American Express")
    data class Custom(override val displayName: String) : CardIssuer(displayName)
    object DinersClub : CardIssuer("Diners Club")
    object Discover : CardIssuer("Discover")
    object JCB : CardIssuer("JCB")
    object MasterCard : CardIssuer("MasterCard")
    object UnionPay : CardIssuer("UnionPay")
    object Unknown : CardIssuer("Unknown")
    object Visa : CardIssuer("Visa")
}
```

If the card issuer you want to add does not exist as part of the predefined ones above, just use:

```
CardIssuer.Custom
```

#### PanValidator Objects

Pan validators are used to check the validity of pans corresponding to certain IIN ranges. Pan validators are like building blocks and can be added to one another to produce composite pan validators which run multiple checks. The default validator runs both Luhns algorithm as well as a simple length check.

If you want to perform more checks, you can extend the [PanValidator interface](https://https://github.com/getbouncer/cardscan-android/blob/master/scan-payment/src/main/java/com/getbouncer/scan/payment/card/PanValidator.kt) and create your own.

#### Custom Card Priority

Any issuer data that you add will be given priority over the pre-packaged issuer data. This means you may override existing defaults if the same IIN range is used for a custom card.


#### Example

```kotlin
supportCardIssuer(860031..860099, CardIssuer.Custom("Uzcard"), listOf(16), listOf(3))
```

The above supports cards with an IIN range of 860031 - 860099 and is assigned a custom card issuer. It accepts pans of length 16 with CVCs of length 3. It uses the default Luhns + Length pan validator.

## Formatting

Pans may be displayed in certain ways and CardIssuers often need a string representation for their name.

### Pan Formatting

Several pans are already supported for formatting and any undefined pans are formatted using default settings based on their length.

```kotlin
fun formatPan(pan: String)
```

#### Custom Pan Formatting

Some cards have very specific ways of arranging the numbers when they are displayed. The CardScan SDK has an option that allows for the formatting of custom pans. Just use the following method:

```kotlin
fun addFormatPan(cardIssuer: CardIssuer, length: Int, vararg blockSizes: Int)
```

#### Example

In order to format a custom card's pan as follows 1234 567 89012 3456, simply call the method as below.

```kotlin
addFormatPan(CardIssuer.Custom("Card Issuer"), 16, 4, 3, 5, 4)
```

### Custom Display Names

Each card issuer has its own display name, including any custom ones you create. In order to get a card issuer's display name, simply call the following method:

```kotlin
fun formatIssuer(issuer: CardIssuer)
```

#### Example

```kotlin
formatIssuer(CardIssuer.MasterCard)
```

returns

```kotlin
"MasterCard"
```
