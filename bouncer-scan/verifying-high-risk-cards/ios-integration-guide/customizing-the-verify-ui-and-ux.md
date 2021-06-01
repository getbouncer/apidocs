# Customizing the Verify UI and UX

Our flows subclass `SimpleScanViewController` and you can customize them by either subclassing `VerifyCardViewController` or `VerifyCardAddViewController` and overriding functions, or you can instantiate these classes and access member variables directly. Please see our [Customization guide]() for more details on customizing the `SimpleScanViewController` base class, or see this section for customizing the remaining components.

## Localization

By default, we provide internationalization support and translations for all of our strings in English, Spanish, French, Portuguese, German, Chinese \(simplified and traditional\), Hindi, Hebrew, Japanese, Korean, Arabic, and Indonesian. However, if you want to set strings yourself, you can do so by using the settings in [CardScan](), or by setting the following fields to meet your localization needs:

| Name | Description | Default Value |
| :--- | :--- | :--- |
| **VerifyCardAddViewController.manualCardEntryText** | Button displayed at the bottom of the scan window for users to enter their details manually instead of by scanning. | Enter details manually |
| **VerifyCardViewController.wrongCardString** | Instructions displayed when a card is scanned that does not match the required card. | Card doesn't match |
| **VerifyCardExplanationViewController.confirmationText** | Title text of the explanation view controller | We need you to confirm this card |
| **VerifyCardExplanationViewController.confirmationSubtext** | Subtitle text of the explanation view controller | Get your card ready so you can scan it with your phone. This helps us keep your account secure. |
| **VerifyCardExplanationViewController.closeButtonText** | Close button text | Close |
| **VerifyCardExplanationViewController.scanCardButtonText** | Scan button text | Scan my card |
| **VerifyCardExplanationViewController.tryAnotherCardButtonText** | Pay another way button text | Try to pay another way |

## Updating UI components

In addition to customizing the UI components from `SimpleScanViewController` you can also update the components from the Verify ViewControllers. Please reach out to Bouncer to discuss options for this and to get more information.

