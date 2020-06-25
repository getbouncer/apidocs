---
description: ID verification iOS installation guide
---

# ID verification for iOS

## Example

To run the example project, clone the repo, and run `pod install` from the Example directory first.

## Requirements

* Objective C or Swift 4.0 or higher
* iOS 11 or higher \(supports development target of iOS 10.0 or higher\)

## Installation

Follow the installation instructions for [Card Verify](../card-verify/ios-integration-guide.md) to install the Card Vefiy cocoapod.

## Permissions

Follow the instructions for setting permissions for [Card Verify](../card-verify/ios-integration-guide.md) to ask the user for permission to use the camera.

## Configure ID Verify

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

## Using ID Verify

To use ID Verify, you create a `ScanId` object, display its `viewController`, and implement the `ScanIdProtocol` protocol to get the results.

```swift
import UIKit
import CardVerify

class ViewController: UIViewController, ScanIdProtocol {
    override func viewDidLoad() {
        super.viewDidLoad()
	
	// Important! you need to make sure that CardVerify supports this hardware
	if !CardVerify.isCompatible() {
	    // Deal with the case that this device isn't going to run CardVerify
	}
    }
    
    @IBAction func idFrontPressed() {
    	// To scan the front of the ID use this code snippet
        let scanId = ScanId(delegate: self, scanBackOfId: false)
        guard let vc = scanId.viewController else { return }
        present(vc, animated: true)
        self.present(vc, animated: true, completion: nil)
    }

    @IBAction func idBackPressed() {
    	// To scan the back of the ID use this code snippet
        let scanId = ScanId(delegate: self, scanBackOfId: true)
        guard let vc = scanId.viewController else { return }
        present(vc, animated: true)
        self.present(vc, animated: true, completion: nil)
    }

    // MARK: ScanIdProtocol protocol
    func userDidScanId(identityResults: IdentityResults) {
        if identityResults.isFront() {
	    // store the front of ID results to use later
            frontOfIdResults = identityResults
        } else {
	    // we should have both the front and back results, post
	    // get the results back from Bouncer
	    let backOfIdResults = identityResults

	    guard let front = frontOfIdResults else { return }
	    guard let back = backOfIdResults else { return }
        
	    FraudCheckApi.idVerify(frontOfIdResults: front, backOfIdResults: back) { [weak self] result, error in
            guard let result = result, error == nil else {
                print("API call failed")
                if let error = error { print(error) }
                return
            }

            let verified = (result["drivers_license_verified"] as? Int) == 1
            print("verified: \(verified)\n" + result.map { "\($0.0) -> \($0.1)\n" }.joined())
        }
    }
    
    func userDidCancelScanId() {
        dismiss(animated: true)
    }

}
```

## Authors

Sam King, [sam@getbouncer.com](mailto:sam@getbouncer.com)

## License

CardVerify is proprietary software and only to be used by people who have a contract in place with Bouncer.

