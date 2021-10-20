---
description: Scan a payment card for fraud protection with Bouncer Scan for iOS.
---

# iOS integration guide

## Requirements

* Xcode 11 or higher
* iOS 11 or higher for integration
* iOS 11.2 or higher for using the scanning view controllers
*   iOS 13 or higher for our name and expiration models. The number model will

    work on older versions of iOS and it will always return nil for the name and

    expiration fields.

## iPad Support

CardScan defaults to a `formSheet` for the iPad, which handles all screen orientations and autorotation correctly. However, if you'd like to use it in full screen mode instead, make sure to select the `Requires full screen` option in your `Info.plist` file via XCode, or else non-portrait orientations won't work.

## Installation

CardVerify is published to the bouncer private repositories. Please request access to this repository by emailing [license@getbouncer.com](mailto:license@getbouncer.com) with a request for access.

CardVerify is available through [CocoaPods](https://cocoapods.org). To install it, simply add the following line to your Podfile:

```bash
pod 'CardVerify', :http => 'https://api.getbouncer.com/v1/downloads/sdk/card_verify/<your_api_key_here>/cardverify-ios-2.1.5.tgz'
```

Next, install the new pod. From a terminal, run:

```bash
pod install
```

Or, if you have previously integrated Bouncer, update your Podfile and update CardVerify:

```
pod update CardVerify
```

When using Cocoapods, you use the `.xcworkspace` instead of the `.xcodeproj`. Again from the terminal, run:

```bash
open YourProject.xcworkspace
```

#### Note: The Podfile can specify the iOS platform target to be lower than 11.2. However, as stated in the requirements CardVerify will only run on iOS 11.2 or higher.

## Set up permissions

CardScan uses the camera, so you'll need to add an description of camera usage to your Info.plist file:

![XCode iOS camera permission](../../../.gitbook/assets/ios\_configure\_camera\_permission.png)

The string you add here will be what CardScan displays to your users when CardScan first prompts them for permission to use the camera.

![iOS camera prompt](../../../.gitbook/assets/ios\_camera\_prompt.png)

Alternatively, you can add this permission directly to your Info.plist file:

```markup
<key>NSCameraUsageDescriptionkey>
<string>We need access to your camera to scan your cardstring>
```

## Configure the CardVerify module

CardVerify can be configured and run through Swift or Objective-C. CardVerify requires an API key, which can be generated for your app through the [Bouncer API console](https://api.getbouncer.com/console). _Note that for CardVerify, you will need permissions added to your API key to perform validation. Please contact_ [_support@getbouncer.com_](mailto:support@getbouncer.com) _once you've created your API key to have permissions added._

The CardScan SDK will send anonymous stats to Bouncer's servers. [This code snippet](https://github.com/getbouncer/cardscan-ios/blob/da77e36c49f1de4b678e7ecaab56cc1466602716/CardScan/Classes/ScanStats.swift#L50) shows what we send.

Apps using CardVerify must configure the library at launch.

{% tabs %}
{% tab title="Swift" %}
Configure the library when your application launches by adding CardScan to your `AppDelegate.swift` file. If you are planning to use a navigation controller or support rotation, also be sure to add `supportedOrientationMaskOrDefault`.

```swift
import UIKit
import CardVerify

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?
    ) -> Bool {
        Bouncer.configure(apiKey: "<your_api_key_here>")
        // do any other necessary launch configuration
        return true
    }

    func application(
        _ application: UIApplication,
        supportedInterfaceOrientationsFor window: UIWindow?
    ) -> UIInterfaceOrientationMask {
        // if you are planning to embed scanViewController into a navigation
        // controller, put this line to handle rotations
        return ScanBaseViewController.supportedOrientationMaskOrDefault()
    }
}
```
{% endtab %}
{% endtabs %}

## Using Bouncer Scan for high-risk transactions

There are two main uses for the CardVerify module: verify a card for a high-risk transaction and block if the scan is invalid using the `VerifyCardViewController` and proactively running fraud models when users add a card using the `VerifyCardAddViewController`.

### Using VerifyCardViewController for high-risk transactions

When you use the `VerifyCardViewController` the view controller will scan the card that your user is using for the high-risk transaction and block the transaction if the scan is invalid or detects telltale signs of fraud. If the user tries to scan a different card, the `VerifyCardViewController` will show an error state by outlining in red the card rectangle in the UI but will continue scanning to detect the correct card. Because this flow is for high-risk transactions, we provide a `VerifyCardExplanationViewController` that you can use to provide context for your users. The overall flow will look like this in most cases:

![](../../../.gitbook/assets/image.png)

To use this flow:

{% tabs %}
{% tab title="Swift" %}
```swift
import UIKit
import CardVerify

class ViewController: UIViewController, VerifyCardExplanationResult, VerifyCardResult {
    @IBAction func buttonPressed() {
        let vc = VerifyCardExplanationViewController()
        vc.lastFourForDisplay = self.lastFourOnFile
        vc.cardNetworkForDisplay = self.networkOfCardOnFile
        vc.expiryOrNameForDisplay = nameOnFile ?? expiryOnFile ?? ""
        vc.delegate = self

        self.present(vc, animated: true, completion: nil)
    }

    // MARK: -Explanation protocol implementation
    func userDidPressScanCardExplaination(_ viewController: VerifyCardExplanationViewController) {
        // Start the Verification process
        guard let lastFour = lastFourOnFile, let bin = binOnFile else { return }
        let vc = VerifyCardViewController(userId: "1234", lastFour: lastFour, bin: bin, cardNetwork: nil)
        vc.verifyCardDelegate = self

        viewController.present(vc, animated: true, completion: nil)
    }

    func userDidPressPayAnotherWayExplanation(_ viewController: VerifyCardExplanationViewController) {
        dismiss(animated: true)
        // let the user select a different card for this transaction
    }

    func userDidPressCloseExplanation(_ viewController: VerifyCardExplanationViewController) {
        dismiss(animated: true)
    }

    // MARK: -VerifyCardResult protocol
    func userCanceledVerifyCard(viewController: VerifyCardViewController) {
        dismiss(animated: true)
    }

    func fraudModelResultsVerifyCard(viewController: VerifyCardViewController, creditCard: CreditCard, encryptedPayload: String?, extraData: [String : Any]) {
        // at this point the scan is done, send the encrypted payload
        // to your servers which will get the result from Bouncer's
        // servers
    }
}
```
{% endtab %}
{% endtabs %}

### Using VerifyCardAddViewController to verify cards when users add them

Our `VerifyCardViewController` flow verifies a specific card for a high-risk transaction and our `VerifyCardAddViewController` flow proactively verifies any cards that users add to your platform. To support this flow, you should use the `VerifyCardAddViewController` instead of the `VerifyCardViewController`, and implement the optional `fraudModelResultsVerifyCardAdd` protocol method in the `VerifyCardAddResult` protocol:

```swift
import CardVerify
import UIKit

class VerifyAddFlowViewController: UIViewController, VerifyCardAddResult {
    @IBAction func buttonPressed() {
        // Start the Verification process
        let vc = VerifyCardAddViewController(userId: "1234")
        vc.cardAddDelegate = self
        vc.enableManualEntry = true

        viewController.present(vc, animated: true, completion: nil)
    }

    // MARK: -VerifyCardAddResult protocol implementation
    func userDidCancelCardAdd(_ viewController: UIViewController) {
        dismiss(animated: true)
    }

    func fraudModelResultsVerifyCardAdd(viewController: UIViewController, creditCard: CreditCard, encryptedPayload: String?, extraData: [String: Any]) {
        // at this point the scan is done, send the encrypted payload
        // to your servers which will get the result from Bouncer's
        // servers

        // depending on the results from Bouncer + if the card matched
        // what you were expecting handle each case
    }

    func userDidScanCardAdd(_ viewController: UIViewController, creditCard: CreditCard) {
        // The flow is complete, but fraud results aren't back yet
    }

    func userDidPressManualCardAdd(_ viewController: UIViewController) {
        // The user pressed "manual entry", go directly to the card entry form
    }
}
```

### Allowing users to scan any card during high risk transactions

You can also use `VerifyCardAddViewController` to verify high risk transactions without limiting them to the specific card that you have on file for the transaction. To support this flow, follow the instructions in the [Using VerifyCardAddViewController](./#using-verifycardaddviewcontroller-to-verify-cards-when-users-add-them) section, but disable the button that allows users to enter card details manually by setting the `enableManualEntry` property on the `VerifyCardAddViewController` to `false`

{% tabs %}
{% tab title="Swift" %}
```swift
import UIKit
import CardVerify

class ViewController: UIViewController, VerifyCardExplanationResult, VerifyCardAddResult {
    @IBAction func buttonPressed() {
        let vc = VerifyCardExplanationViewController()
        vc.lastFourForDisplay = self.lastFourOnFile
        vc.cardNetworkForDisplay = self.networkOfCardOnFile
        vc.expiryOrNameForDisplay = nameOnFile ?? expiryOnFile ?? ""
        vc.delegate = self

        self.present(vc, animated: true, completion: nil)
    }

    // MARK: -Explanation protocol implementation
    func userDidPressScanCardExplaination(_ viewController: VerifyCardExplanationViewController) {
        // Start the Verification process
        let vc = VerifyCardAddViewController(userId: "1234")
        vc.cardAddDelegate = self
        vc.enableManualEntry = false

        viewController.present(vc, animated: true, completion: nil)
    }

    func userDidPressPayAnotherWayExplanation(_ viewController: VerifyCardExplanationViewController) {
        dismiss(animated: true)
        // let the user select a different card for this transaction
    }

    func userDidPressCloseExplanation(_ viewController: VerifyCardExplanationViewController) {
        dismiss(animated: true)
    }

    // MARK: -VerifyCardAddResult protocol
    func fraudModelResultsVerifyCardAdd(viewController: UIViewController, creditCard: CreditCard, encryptedPayload: String?, extraData: [String: Any]) {
        // at this point the scan is done, send the encrypted payload
        // to your servers which will get the result from Bouncer's
        // servers

        // depending on the results from Bouncer + if the card matched
        // what you were expecting handle each case
    }

    func userDidScanCardAdd(_ viewController: UIViewController, creditCard: CreditCard) {
        // don't do anything here, wait for the fraud result to come back
    }

    func userDidPressManualCardAdd(_ viewController: UIViewController) {
        preconditionFailure("Manual button not shown")
    }

    func userDidCancelCardAdd(_ viewController: UIViewController) {
        dismiss(animated: true)
    }
}
```
{% endtab %}
{% endtabs %}

### Local Verification

This is a less stringent verification flow than network verification, as the verification logic runs locally on the device. In cases where it is not viable to send a payload to Bouncer servers, this can be used to verify the authenticity of a card, but will be less accurate than network verification.

Once you conform to the `verifyCardDelegate` and `cardAddDelegate` protocol when utilizing `VerifyCardViewController` and `VerifyCardAddViewController`, the function `fraudModelResultsVerifyCardAdd` and `fraudModelResultsVerifyCard` will provide an encrypted payload and a field called `extraData`.

`extraData` is a dictionary that contain the results of the local verification. The following will list the available result values:

#### Local Verification Extra Data

| Key                       | Value Type |
| ------------------------- | ---------- |
| "isCardValid"             | Bool       |
| "validationFailureReason" | String?    |

{% tabs %}
{% tab title="Swift" %}
```swift
import UIKit
import CardVerify

extension ViewController: VerifyCardResult {
    func userCanceledVerifyCard(viewController: VerifyCardViewController) {
        dismiss(animated: true)
    }

    func fraudModelResultsVerifyCard(viewController: VerifyCardViewController, creditCard: CreditCard, encryptedPayload: String?, extraData: [String : Any]) {
        // at this point the scan is done, you will receive the encrypted payload
        // and the local verification result in extraData. Parse the extraData dictionary 
        // to extract the local verification values. 

        let localIsValid = extraData["isCardValid"] as? Bool ?? false
        let localValidationFailureReason = extraData["validationFailureReason"] as? String ?? ""
    }
}

extension ViewController: VerifyCardAddResult {
    func fraudModelResultsVerifyCardAdd(viewController: UIViewController, creditCard: CreditCard, encryptedPayload: String?, extraData: [String: Any]) {
        // at this point the scan is done, you will receive the encrypted payload
        // and the local verification result in extraData. Parse the extraData dictionary 
        // to extract the local verification values.

        let localIsValid = extraData["isCardValid"] as? Bool ?? false
        let localValidationFailureReason = extraData["validationFailureReason"] as? String ?? ""
    }

    func userDidScanCardAdd(_ viewController: UIViewController, creditCard: CreditCard) {
        // don't do anything here, wait for the fraud result to come back
    }

    func userDidPressManualCardAdd(_ viewController: UIViewController) {
        preconditionFailure("Manual button not shown")
    }

    func userDidCancelCardAdd(_ viewController: UIViewController) {
        dismiss(animated: true)
    }
}
```
{% endtab %}
{% endtabs %}

## Model Downloading

By default, the SDK will download updates to ML models used to extract card information. To disable model downloading, set the following flag to false before calling configure in your AppDelegate:

{% tabs %}
{% tab title="Swift" %}
```swift
Bouncer.downloadModels = false
Bouncer.configure(apiKey: "API_KEY")
```
{% endtab %}

{% tab title="Objective-C" %}
```swift
Bouncer.downloadModels = false;
[Bouncer configureWithApiKey:@"API_KEY"];
```
{% endtab %}
{% endtabs %}

Subsequently you can initiate a download by using this function. Note, downloading is idempotent -- you can call this function as many times as you'd like and CardVerify will download the models at most once.

```
Bouncer.downloadModelsIfNotCached()
```

It can take a few minutes to download models if your customer has a slow network connection, so we recommend either letting Bouncer download models during the configure call or calling the `downloadModelsIfNotCached` method as early as possible to give the models enough time to download.

## Customizing

This library is built to be customized to fit your UI. We provide translated strings and localization support by default for all UI elements in our flows, and we provide mechanisms for you to set these strings yourself. See the [customization documentation](customizing-the-verify-ui-and-ux.md) for more details.

## Supporting more cards

Though Bouncer Scan supports several cards, you may need to add support for cards specific to your business, instructions can be found in the [card support docs](./).

## Authors

[Sam King](mailto:sam@getbouncer.com), Jaime Park, Adam Wushensky

## License

A licensing agreement is required to use this library.

*   Details of licensing (pricing, etc) are available at

    [https://cardscan.io/pricing](https://cardscan.io/pricing), or you can contact

    us at [license@getbouncer.com](mailto:license@getbouncer.com).

All contributors must agree to the [CLA](https://github.com/getbouncer/cardscan-android/blob/master/Contributor%20License%20Agreement).
