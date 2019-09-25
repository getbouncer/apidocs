# CardVerify

CardVerify iOS installation guide

## Contents
* [Example](#example)
* [Requirements](#requirements)
* [Installation](#installation)
* [Permissions](#permissions)
* [Configure CardVerify (Swift)](#configure-cardverify-swift)
* [Using CardVerify (Swift)](#using-cardverify-swift)
* [Authors](#authors)
* [License](#license)

## Example

To run the example project, clone the repo, and run `pod install` from the Example directory first.

## Requirements

* Objective C or Swift 4.0 or higher
* iOS 11 or higher (supports development target of iOS 10.0 or higher)

## Installation

As a first step, you must get access to our private repo for the CardVerify
library. To get access [email Sam](mailto:sam@getbouncer.com), who's email
address is sam@getbouncer.com.

CardVerify is available through [CocoaPods](https://cocoapods.org). To install
it, simply add the following line to your Podfile:

```ruby
pod 'CardVerify', :http => 'https://bouncerpaid.bintray.com/CardVerify-iOS/cardverify-ios-1.0.5003.tgz'
```

And then add the username and password that you got from Bouncer to
your .netrc file, which will enable curl to do basic auth and download
the tgz file:

```
machine bouncerpaid.bintray.com
  login YOUR_COMPANY@bouncerpaid
  password 1234567890abcdefghijklmnopqrstuvwxyz
```


Next, install the new pod. From a terminal, run:

```
pod install
```

When using Cocoapods, you use the `.xcworkspace` instead of the
`.xcodeproj`. Again from the terminal, run:

```
open YourProject.xcworkspace
```

## Permissions

CardVerify uses the camera, so you'll need to add an description of
camera usage to your Info.plist file:

![alt text](https://github.com/getbouncer/cardscan-ios/raw/master/Info.plist.camera.png "Info.plist")

The string you add here will be what CardVerify displays to your users
when CardVerify first prompts them for permission to use the camera.

![alt text](https://github.com/getbouncer/cardscan-ios/raw/master/camera_prompt.png "Camera prompt")

Alternatively, you can add this permission directly to your Info.plist
file:

```xml
<key>NSCameraUsageDescription</key>
<string>We need access to your camera to scan your card</string>
```

## Configure CardVerify (Swift)

Configure the library when your application launches:

```swift
import UIKit
import CardVerify

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    	 CardVerify.configure(apiKey: "YOUR API KEY") 
        // do any other necessary launch configuration
        return true
    }
}
```


## Using CardVerify (Swift)

To use CardVerify, you create a `VerifyViewController`, display it, and
implement the `VerifyDelegate` protocol to get the results.

```swift
import UIKit
import CardVerify

class ViewController: UIViewController, VerifyDelegate {
    override func viewDidLoad() {
        super.viewDidLoad()
	
	// Important! you need to make sure that CardVerify supports this hardware
	if !CardVerify.isCompatible() {
	    // Deal with the case that this device isn't going to run CardVerify
	}
    }
    
    @IBAction func buttonPressed() {
        // Let the CardVerify library know some details about the current card that
        // it can use to check for a match
        guard let vc = CardVerify.createVerifyViewController(last4: "4242", expiryMonth: "08", expiryYear: "22", network: PaymentCard.Network.VISA, withDelegate: self) else {
            // the library won't create a VerifyCardViewController if this hardware
	    // isn't supported, otherwise it will
            return
        }
        self.present(vc, animated: true, completion: nil)
    }

    // MARK: VerifyDelegate protocol
    func userDidCancel(_ viewController: VerifyCardViewController) {
        // The user pressed on the back button without passing the challenge
	viewController.dismiss(animated: true, completion: nil)
    }
    
    func verifiedCard(_ viewController: VerifyCardViewController, scannedCard: PaymentCard) {
	// The user scanned a card that has the last4 that matches the last4 that you
	// passed in and it passed our integrity checks, let the transaction proceed
	
	// pass the `fraudCheckToken` returned by CardVerify to the server to double check the
	// transaction using a server-to-server call to Bouncer's API server
	let fraudCheckToken = scannedCard.fraudCheckToken
    }
    
    func verifiedDifferentCard(_ viewController: VerifyCardViewController, scannedCard: PaymentCard) {
    	// The user scanned a card and it passed our integrity checks, but the last4 didn't match.
	// The most common option is to use the card details passed back via `scannedCard` to let
	// the user add the new card and pay for the transaction using it.
	
	// IMPORTANT: make sure that the user doesn't change the card number when they add the
	// new card. If they change the card number after the scan, it will invalidate the
	// `fraudCheckToken`
	
	// pass the `fraudCheckToken` returned by CardVerify to the server to double check the
	// transaction using a server-to-server call to Bouncer's API server
	let fraudCheckToken = scannedCard.fraudCheckToken
    }
    
    func failedVerification(_ viewController: VerifyCardViewController, scannedCard: PaymentCard) {
    	// the user scanned a card (details are in the `scannedCard` object) but it failed our
	// integrity check.
	// The most common option is to ask they to try scanning again or to try a different card
    }

}
```

## Authors

Sam King, sam@getbouncer.com

Jaime Park

## License

CardVerify is proprietary software and only to be used by people who have a contract in place
with Bouncer.
