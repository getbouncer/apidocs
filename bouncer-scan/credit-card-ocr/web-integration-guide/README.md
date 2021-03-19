---
description: Scan a payment card with CardScan for Android.
---

# Web integration guide

## Requirements

* TensorFlowJS script tag
* Bouncer SDK script tag

## Demo

A demo checkout page is available at [https://cardscan-web.s3.us-east-2.amazonaws.com/index.html](https://cardscan-web.s3.us-east-2.amazonaws.com/demo.html)

## SDK Size

We try to keep our SDK as small as possible while maintaining good performance.

| Component | Size |
| :--- | :--- |
| Base SDK | 0.93 MB |
| TensorFlow JS | 0.26 MB |

## Installation

In your HTML, add the following script tags

```markup
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@3.0.0/dist/tf.min.js"></script>
<script src="https://cardscan-web.s3.us-east-2.amazonaws.com/bouncer_cardscan.bundle.js"></script>
```

Importing these scripts will add the Bouncer scan model DOM elements to the end of your HTML and hide them. It will also expose a global JavaScript variable `bouncerCardScan`.

## Using

First, add the Bouncer modal to your page's DOM by calling the `setUpScan` method once your page's DOM has completed loading:

```javascript
window.addEventListener('DOMContentLoaded', bouncerCardScan.setUpScan);
```

Once you're ready to scan, call the `BouncerCardScan.scanCard` method and provide callback methods for the success and canceled cases. API keys can be created through the [Bouncer API console](https://api.getbouncer.com/console).

### Example

First, check to make sure that CardScan is supported on the user's browser with the following call:

```javascript
const isBouncerScanSupported = bouncerCardScan.isSupported();
```

The following javascript will launch the Bouncer CardScan modal and collect the results.

```javascript
const onCardScanComplete = function(
    number,
    cardholderName,
    expiryMonth,
    expiryYear,
    verificationResult
) {
    // The user scanned a card
}

const onCardScanCanceled = function() {
    // the user closed the scan modal without scanning a card
}

const onScanError = function(error) {
    // An error occurred during the scan. return true to stop the scan, false to attempt to continue scanning.
}

function launchCardScan() {
    bouncerCardScan.scanCard("<your_api_key_here>", onCardScanComplete, onCardScanCanceled, onScanError);
}
```

The `verificationResult` parameter can be used to determine the legitimacy of the card scanned. See [Verifying High Risk Cards](../verifying-high-risk-cards/) for more information.

## Localization

This library supports full string localization. Strings are stored in the BouncerConfiguration object, accessible via `bouncerCardScan.getConfig()`. The following strings are available for localization:

| Variable | Default Value |
| :--- | :--- |
| instructionsLoadingString | `Loading...` |
| instructionsScanString | `Scan Your Card` |
| instructionsReadingString | `Reading card...` |
| instructionsCapturingImages | `Almost done...` |
| instructionsProcessingString | `Processing...` |
| securityNotificationString | `Your card info is secure` |
| networkErrorString | `Network Error` |

For example, to change the text for the instructions, set the value of `instructionsScanString` before starting the scan.

```javascript
// localize the instructions to Spanish
bouncerCardScan.getConfig().instructionsScanString = "Escanea tu tarjeta";

// At some point later, launch the CardScan modal
bouncerCardScan.scanCard(...)
```

## Authors

Adam Wushensky, Sven Kuhne, and Steven Liu

## License

This library is available under paid and free licenses. See the [LICENSE](https://github.com/getbouncer/cardscan-web/blob/master/LICENSE) file for the full license text.

### Quick summary

In short, this library will remain free forever for non-commercial applications, but use by commercial applications is limited to 90 days, after which time a licensing agreement is required. We're also adding some legal liability protections.

After this period commercial applications need to convert to a licensing agreement to continue to use this library.

* Details of licensing \(pricing, etc\) are available at [https://cardscan.io/pricing](https://getbouncer.com/pricing), or you can contact us at [license@getbouncer.com](mailto:license@getbouncer.com).

### More detailed summary

What’s allowed under the license:

* Free use for any app for 90 days \(for demos, evaluations, hackathons, etc\).
* Contributions \(contributors must agree to the [Contributor License Agreement](https://github.com/getbouncer/cardscan-web/blob/master/Contributor%20License%20Agreement)\)
* Any modifications as needed to work in your app

What’s not allowed under the license:

* Commercial applications using the license for longer than 90 days without a license agreement.
* Using us now in a commercial app today? No worries! Just email [license@getbouncer.com](mailto:license@getbouncer.com) and we’ll get you set up.
* Redistribution under a different license
* Removing attribution
* Modifying logos
* Indemnification: using this free software is ‘at your own risk’, so you can’t sue Bouncer Technologies, Inc. for problems caused by this library

Questions? Concerns? Please email us at [license@getbouncer.com](mailto:license@getbouncer.com) or ask us on [slack](https://getbouncer.slack.com/).

