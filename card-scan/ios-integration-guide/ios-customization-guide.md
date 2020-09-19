---
description: >-
  This document describes how to customize this library to fit your user
  interface design.
---

# iOS customization guide
Overall we provide two different classes for scanning credit and debit cards with varying levels of UI customizability:
`SimpleScanViewController` and `ScanViewController`. `SimpleScanViewController` is a functionally complete card scanner,
but has a simple UI that you can customize either by setting properties or by subclassing for full UI customization.
Because `SimpleScanViewController` uses programmatic UI layout you can subclass it for full customization.

`ScanViewController` is a battle tested UI that has scanned over 50 million cards and exposes some properties that you
can set to adjust the UI. However, `ScanViewController` uses storyboards for UI layout, so you can't subclass it.
Eventually we'll move it over to programmatic UI layout as well by subclassing `SimpleScanViewController` ourselves, but
for now we're confined to property setting only..

## Localization

The following strings are public, static, and mutable, and can be reassigned to meet your localization needs:

| Name | Description | Default Value |
| :--- | :--- | :--- |
| **SimpleScanViewController.descriptionString** | Instructions shown above the scan window | Scan Card |
| **SimpleScanViewController.enableCameraPermissionString** | Title of the camera permission request message | Enable camera access |
| **SimpleScanViewController.enableCameraPermissionsDescriptionString** | Message of the camera permission request message | To scan your card you'll need to update your phone settings |
| **SimpleScanViewController.closeButtonString** | Close button shown in the top left corner of the scan window | Close |
| **SimpleScanViewController.torchButtonString** | Torch button shown in the top right corner of the scan window | Torch |

## Full UI customization: SimpleScanViewController

This class is all programmatic UI with a small bit of logic to handle the events that `ScanBaseViewController` expects
subclasses to implement. Our goal is to have a fully featured Card Scan implementation with a minimal UI that people can
customize fully. You can use this directly or you can subclass and customize it. If you'd like to use an off-the-shelf
design as well, we suggest using the `ScanViewController`, which uses mature and well tested UI design patterns.

The default `SimpleScanViewController` UI looks something like this, with most of the constraints shown:

```text
 ------------------------------------
 |   |                          |   |
 |-Close                      Torch-|
 |                                  |
 |                                  |
 |                                  |
 |                                  |
 |                                  |
 |------------Scan Card-------------|
 |                |                 |
 |  ------------------------------  |
 | |                              | |
 | |                              | |
 | |                              | |
 | |--4242    4242   4242   4242--| |
 | ||           05/23             | |
 | ||-Sam King                    | |
 | |     |                        | |
 |  ------------------------------  |
 | |              |               | |
 | |              |               | |
 | |     Enable camera access     | |
 | |              |               | |
 | |              |               | |
 | |---To scan your card you...---| |
 |                                  |
 |                                  |
 |                                  |
 ------------------------------------
```

 For the UI we separate out the key components into three parts:

* Five `*String` variables that we use to set the copy
* For each component or group of components we have:
  * `setup*Ui` functions for setting the visual look and feel
  * `setup*Constraints` for setting up autolayout
* We have top level `setupUiComponents` and `setupConstraints` functions that do a small bit of setup and call the
appropriate setup functions for each components

And to customize the UI you can either override any of these functions or you can access components directly to adjust.
Also, you're welcome to copy and paste this code and customize it to fit your needs -- we're fine with whatever makes
the most sense for your app.

### Strings

| Name | Description | Default value |
| :--- | :--- | :--- |
| `descriptionString` | The string above the card scanning viewport | "Scan Card" |
| `closeButtonString` | The string that we display for the "Cancel" button, which cancels the scan | "Cancel" |
| `torchButtonString` | The string that we display on the button for toggling the camera's torch. | "Torch" |
| `enableCameraPermissionString` | The string that we display on the button that we show when the user denies camera access. Pressing it takes them to the settings page for their app where they can enable camera permissions | "Enable camera access" |
| `enableCameraPermissionsDescriptionString` | The string that describes what happens when they press the "enable camera access" button | "To scan your card you'll need to update your phone settings" |

### UI component visual setup and layout

You can override any of these functions if you subclass `SimpleScanViewController` to customize the look, feel, and
layout for each component

| Name | Description |
| :--- | :--- |
| `setupPreviewViewUi` and `setupPreviewViewConstraints` | The view for showing the camera preview to the user |
| `setupBlurViewUi` and `setupBlurViewConstraints` | The view that puts a subtle blur on the preview view outside of the region of interest viewport |
| `setupRoiViewUi` and `setupRoiViewConstraints` | The region of interest rectangle, which we use for both delineating the blur effect and for telling our ML where to pull the card image from  |
| `setupCloseButtonUi` and `setupCloseButtonConstraints` | The close button |
| `setupTorchButtonUi` and `setupTorchButtonConstraints` | The button for toggling the torch |
| `setupDescriptionTextUi` and `setupDescriptionTextConstraints` | The text that we display above the region of interest rectangle to explain what the user is supposed to do |
| `setupCardDetailsUi` and `setupCardDetailsConstraints` | These components are how we display intermediate card scanning results to the user including the card number, expiration, and legal name. See our "Showing card details" section for more details. |
| `setupDenyUi` and `setupDenyConstraints` | These components are what we show users who deny camera access. See our "The camera Deny experience" section for more details. |

### Runtime event: Showing card details

As the user scans a card, the UI will display intermediate results as they become available while the models finish up
their predictions. Specifically, the ViewController will invoke `showScannedCardDetails` with a `CreditCardPrediction`
object as the argument each time it has results you can show. Override this function to customize this behavior.

### Runtime event: The camera "Deny" experience

`SimpleScanViewController` has a camera permission deny experience built in. If the user is prompted and denies access
to the camera or they have been previously prompted and denied access, `SimpleScanViewController` will hide all non
essential UI elements \(the "description" text and the "torch" button\) and show a button with a link to settings where
the user can grant permission to the camera for your app. Please note, if the user updates the settings for your app,
iOS will restart your app and you'll lose any in-memory state.

{% hint style="info" %}
iOS will restart your app if the user updates their camera permission settings
{% endhint %}

Apps can choose to handle permissions completely outside of the Card Scan SDK if they prefer.

## Partial UI customization: ScanViewController

For a complete description of our UI customization options, please see the source code of our
[ScanViewController](https://github.com/getbouncer/cardscan-ios/blob/master/CardScan/Classes/ScanViewController.swift).
Our public interface includes protocols for setting strings and we expose some key elements of the design that you can
set directly. However, these interfaces change fairly often, so we haven't documented them and hope that people who want
control of their UI will use `SimpleScanViewContoller` instead. Our expected use for `ScanViewController` is to use a
mature, well tested, and high converting UI out of the box, with only minor modifications.

## Regional cards

By default, Bouncer supports most international credit and debit cards, but we also have support for adding regional
cards, like Russian MIR cards, or any banks that you'd like.

Our current implementation is limited to banks that use 16 digit card numbers and these numbers pass the Luhn check \(an
industry standard checksum over the card number\). You add regional banks by adding the card number prefix, or BIN, for
that bank to our library.

| Variable or Function | Description | Default |
| :--- | :--- | :--- |
| `CreditCardUtils.addMirSupport()` | Adds support for MIR cards | N/A |
| `CreditCardUtils.prefixesRegional` | Array of strings where the strings are the set of additional card number prefixes
that you want to support. See our implementation of
[addMirSupport](https://github.com/getbouncer/cardscan-ios/blob/af56971ec0b1bc7bd14901a1c98690e52bd2f9cc/CardScan/Classes/CreditCardUtils.swift#L28)
for more information on how to use it.
