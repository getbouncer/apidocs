---
description: Scan a payment card with CardScan for Android.
---

# Android integration guide

## Requirements

* TensorFlowJS script tag
* Bouncer SDK script tag

## Demo

The [cardscan repository](https://github.com/getbouncer/cardscan-web) contains a demonstration app for the CardScan product. To build and install this library follow the following steps:

1. Clone the repository from github

   ```bash
   git clone https://github.com/getbouncer/cardscan-web && cd cardscan-web
   ```
   
1. Install dependencies using NPM

   ```bash
   npm install
   ```

1. Build the library using NPM and webpack

   ```bash
   npm run build
   ```
   
   This will create a `dist` directory with the compiled javascript.

1. Start a local server to host the scanner and demo app

   ```bash
   python -m SimpleHTTPServer 8000
   ```

1. Navigate to http://localhost:8000 to view a sample app.

## SDK Size

We try to keep our SDK as small as possible while maintaining good performance.

| Component | Size |
| :--- | :--- |
| Base SDK | 1.58 MB |
| TensorFlow JS | 0.25 MB |

## Installation

In your HTML, add the following script tags

```html
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@2.8.3/dist/tf.min.js"></script>
<script src="https://cardscan-web.s3.us-east-2.amazonaws.com/bouncer_cardscan.bundle.js"></script>
```

Importing these scripts will add the Bouncer scan model DOM elements to the end of your HTML and hide them. It will also expose a global JavaScript variable `bouncerCardScan`.

## Using

To launch the Bouncer CardScan modal, call the `BouncerCardScan.scanCard` method and provide callback methods for the success and canceled cases.

### Example

The following javascript will launch the Bouncer CardScan modal and collect the results.

```javascript
// pre-download the ML models onto the client in the background for faster scan load times.
bouncerCardScan.warmUp();

const onCardScanComplete = function(
    number,
    cardholderName,
    expiryMonth,
    expiryYear,
    verificationResults
) {
    // The user scanned a card
}

const onCardScanCanceled = function() {
    // the user closed the scan modal without scanning a card
}

function launchCardScan() {
    bouncerCardScan.scanCard("<your_api_key_here>", onCardScanComplete, onCardScanCanceled);
}
```

To determine the authenticity of the scanned card, pass the `verificationResults` back to your server. Details about the network call are available in the [Rest APIs documentation](../rest-apis/v1-card-verify.md).

## Localization

This library supports full string localization. Strings are stored in the BouncerConfiguration object, accessible via `bouncerCardScan.getConfig()`. The following strings are available for localization:

| Variable | Default Value |
| :--- | :--- |
| instructionsString | `Scan Your Card` |
| securityNotificationString | `Your card info is secure` |
| networkErrorString | `Network Error` |

For example, to change the text for the instructions, set the value of `instructionsString` before starting the scan.

```javascript
// localize the instructions to Spanish
bouncerCardScan.getConfig().instructionsString = "Escanea tu tarjeta";

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
