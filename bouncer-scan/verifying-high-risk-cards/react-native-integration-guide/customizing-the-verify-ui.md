# Customizing the Verify UI

## iOS UI Customization

You can customize `VerifyCardViewController` and `VerifyCardAddViewController` by overriding member variables directly. Please see the section below for customizing the remaining components.

### Localization

By default, we provide internationalization support and translations for all of our strings in English, Spanish, French, Portuguese, German, Chinese \(simplified and traditional\), Hindi, Hebrew, Japanese, Korean, Arabic, and Indonesian. However, if you want to set strings yourself, you can do so by setting the following fields to meet your localization needs:

| Name | Description | Default Value |
| :--- | :--- | :--- |
| **VerifyCardAddViewController.manualCardEntryText** | Button displayed at the bottom of the scan window for users to enter their details manually instead of by scanning. | Enter details manually |
| **VerifyCardViewController.wrongCardString** | Instructions displayed when a card is scanned that does not match the required card. | Card doesn't match |

The following strings are available in both `VerifyCardViewController` and `VerifyCardAddViewController` which can be reassigned:

| Name | Description | Default Value |
| :--- | :--- | :--- |
| **VerifyViewController.descriptionString** | Instructions shown above the scan window | Scan Card |
| **VerifyViewController.enableCameraPermissionString** | Title of the camera permission request message | Enable camera access |
| **VerifyViewController.enableCameraPermissionsDescriptionString** | Message of the camera permission request message | To scan your card you'll need to update your phone settings |
| **VerifyViewController.closeButtonString** | Close button shown in the top left corner of the scan window | Close |
| **VerifyViewController.torchButtonString** | Torch button shown in the top right corner of the scan window | Torch |

#### Localization Example

{% tabs %}
{% tab title="Objective-C" %}
```swift
#import "AppDelegate.h"
#import "CardVerify/CardVerify-Swift.h"

@interface AppDelegate ()

@end

@implementation AppDelegate


- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [Bouncer configureWithApiKey:@"API_KEY"];

    VerifyCardAddViewController.descriptionString = @"description text";
    VerifyCardAddViewController.enableCameraPermissionString = @"enable camera";
    VerifyCardAddViewController.enableCameraPermissionsDescriptionString = @"enable camera description";
    VerifyCardAddViewController.torchButtonString = @"torch";
    VerifyCardAddViewController.closeButtonString = @"close";
    VerifyCardAddViewController.manualCardEntryText = @"manual entry";

    VerifyCardViewController.descriptionString = @"description text";
    VerifyCardViewController.enableCameraPermissionString = @"enable camera";
    VerifyCardViewController.enableCameraPermissionsDescriptionString = @"enable camera description";
    VerifyCardViewController.torchButtonString = @"torch";
    VerifyCardViewController.closeButtonString = @"close";
    VerifyCardViewController.wrongCardString = @"wrong card";

    return YES;
}
```
{% endtab %}
{% endtabs %}

