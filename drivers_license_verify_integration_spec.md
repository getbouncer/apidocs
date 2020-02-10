# DriversLicenseVerify

DriversLicenseVerify iOS installation guide

## Contents
* [Example](#example)
* [Requirements](#requirements)
* [Installation](#installation)
* [Permissions](#permissions)
* [Configure DriversLicenseVerify (Swift)](#configure-driverslicenseverify-swift)
* [Using DriversLicenseVerify (Swift)](#using-driverslicenseverify-swift)
* [Authors](#authors)
* [License](#license)

## Example

To run the example project, clone the repo, and run `pod install` from the Example directory first.

## Requirements

* Objective C or Swift 4.0 or higher
* iOS 11 or higher (supports development target of iOS 10.0 or higher)

## Installation

As a first step, you must get access to our private repo for the DriversLicenseVerify
library. To get access [email Sam](mailto:sam@getbouncer.com), who's email
address is sam@getbouncer.com.

DriversLicenseVerify is available through [CocoaPods](https://cocoapods.org). To install
it, simply add the following line to your Podfile:

```ruby
pod 'DriversLicenseVerify', :http => 'https://bouncerpaid.bintray.com/DriversLicenseVerify-iOS/DriversLicenseVerify-ios-1.0.0.tgz'
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

DriversLicenseVerify uses the camera, so you'll need to add an description of
camera usage to your Info.plist file:

![alt text](https://github.com/getbouncer/cardscan-ios/raw/master/Info.plist.camera.png "Info.plist")

The string you add here will be what DriversLicenseVerify displays to your users
when DriversLicenseVerify first prompts them for permission to use the camera.

![alt text](https://github.com/getbouncer/cardscan-ios/raw/master/camera_prompt.png "Camera prompt")

Alternatively, you can add this permission directly to your Info.plist
file:

```xml
<key>NSCameraUsageDescription</key>
<string>We need access to your camera to scan your license</string>
```

## Configure DriversLicenseVerify (Swift)

Configure the library when your application launches:

```swift
import UIKit
import DriversLicenseVerify

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    	 DriversLicenseVerify.configure(apiKey: "YOUR API KEY") 
        // do any other necessary launch configuration
        return true
    }
}
```


## Using DriversLicenseVerify (Swift)

To use DriversLicenseVerify, you create a `VerifyViewController`, display it, and
implement the `VerifyDelegate` protocol to get the results.

```swift
import UIKit
import DriversLicenseVerify

class ViewController: UIViewController, VerifyDelegate {
    override func viewDidLoad() {
        super.viewDidLoad()
	
	// Important! you need to make sure that DriversLicenseVerify supports this hardware
	if !DriversLicenseVerify.isCompatible() {
	    // Deal with the case that this device isn't going to run DriversLicenseVerify
	}
    }
    
    @IBAction func buttonPressed() {
        guard let vc = DriversLicenseVerify.createVerifyViewController(withDelegate: self) else {
            // the library won't create a VerifyLicenseViewController if this hardware
	          // isn't supported, otherwise it will
            return
        }
        self.present(vc, animated: true, completion: nil)
    }

    // MARK: VerifyDelegate protocol
    func userDidCancelVerify(_ viewController: VerifyLicenseViewController) {
        // The user pressed on the back button without passing the challenge
	viewController.dismiss(animated: true, completion: nil)
    }
    
    func userDidScanValidLicense(_ viewController: VerifyLicenseViewController, license scannedLicense: DriversLicense) {
      	// The user scanned a valid license
        // NOTE: a valid license may still mean that the user is under age
        // or the license is expired
    }
    
    func userDidScanInvalidLicense(_ viewController: VerifyLicenseViewController, failureReasons: FailureReasons, license scannedLicense: DriversLicense?) {
    	// The user scanned a license but it failed validation.
      // This means either the license's front didn't match the back, or the image is tampered with
    }
}
```

## Authors

Sam King, sam@getbouncer.com

## License

DriversLicenseVerify is proprietary software and only to be used by people who have a contract in place
with Bouncer.
