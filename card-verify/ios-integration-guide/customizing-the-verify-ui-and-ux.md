# Customizing the Verify UI and UX

Although the interfaces exposed by ScanBaseViewContoller aren't designed for general purpose subclassing, you can subclass ScanBaseViewController to control the UI/UX. Please note that although the ScanBaseViewController is public we consider the interface between it and scanning ViewControllers to be internal, so they may change in the future.

## The basics: subclassing

See the source code for `VerifyCardViewController` or the `LivenessViewController` for examples of how to subclass ScanBaseViewController. Note: the ML pipeline depends on UI elements and a full screen preview to pull card images out of the video stream. There are a bunch of assumptions that we make which may limit your flexibility a bit.

The most important part of a custom subclass is to make sure that you invoke `setupOnViewDidLoad` and make sure that you set the `regionOfInterestLabel` to a UIView that's frame matches the viewport where you expect users to scan cards.

## Localization

In addition to the localizable strings in [CardScan](../../card-scan/ios-integration-guide/ios-customization-guide.md#localization), the following fields are public, static, and mutable, and can be reassigned to meet your localization needs:

| Name | Description | Default Value |
| :--- | :--- | :--- |
| **VerifyScanCardViewController.missingCardString** | Button displayed at the bottom of the scan window for users to indicate that they do not have the required card. | I don't have this card |
| **VerifyScanCardViewController.wrongCardString** | Instructions displayed when a card is scanned that does not match the required card. | Card doesn't match |
| **VerifyCardSimpleViewController.processingText** | Text shown when processing the result of a scan | Processing, please wait |

## Invoking the verify pipeline

First, you must set the `scanEventsDelegate` to a new `CardVerifyFraudData` object for each scan. To do this, you can override the `viewWillAppear` method and set the variable:

```swift
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    self.scanEventsDelegate = CardVerifyFraudData()
}
```

Second, you override the `onScannedCard` method and invoke the verify pipeline to extract the encrypted payload:

```swift
override public func onScannedCard(number: String, expiryYear: String?, expiryMonth: String?, scannedImage: UIImage?) {
    runVerifyPipeline(cardToVerify: self.card, number: number, expiryYear: expiryYear, expiryMonth: expiryMonth, debugForceError: nil) { scannedCard in
        // the encrypted payload is stored in `scannedCard`
        if scannedCard.isNewCard {
            self.delegate?.userDidScanDifferentCard(self, card: scannedCard)
        } else {
            self.delegate?.userDidScanSameCard(self, card: scannedCard)
        }
    }
}
```

