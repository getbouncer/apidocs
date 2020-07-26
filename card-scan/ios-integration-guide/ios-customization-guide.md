---
description: >-
  This document describes how to customize this library to fit your user
  interface design.
---

# iOS customization guide

Overall we provide two different classes for scanning credit and debit cards with varying levels of UI customizability: `SimpleScanViewController` and `ScanViewController`. `SimpleScanViewController` is a functionally complete card scanner, but has a simple UI that you can customize either by setting properties or by subclassing for full UI customization. Because `SimpleScanViewController` uses programmatic UI layout you can subclass it for full customization.

`ScanViewController` is a battle tested UI that has scanned over 50 million cards and exposes some properties that you can set to adjust the UI. However, `ScanViewController` uses storyboards for UI layout, so you can't subclass it. Eventually we'll move it over to programmatic UI layout as well by subclassing `SimpleScanViewController` ourselves, but for now we're confined to property setting only..

## Full UI customization: SimpleScanViewController

This class is all programmatic UI with a small bit of logic to handle the events that `ScanBaseViewController` expects subclasses to implement. Our goal is to have a fully featured Card Scan implementation with a minimal UI that people can customize fully. You can use this directly or you can subclass and customize it. If you'd like to use an off-the-shelf design as well, we suggest using the `ScanViewController`, which uses mature and well tested UI design patterns.

 The default `SimpleScanViewController` UI looks something like this, with most of the constraints shown:

```text
 ------------------------------------
 |   |                          |   |
 |-Cancel                     Torch-|
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
 | |   Enable camera permissions  | |
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
* We have top level `setupUiComponents` and `setupConstraints` functions that do a small bit of setup and call the appropriate setup functions for each components

And to customize the UI you can either override any of these functions or you can access components directly to adjust. Also, you're welcome to copy and paste this code and customize it to fit your needs -- we're fine with whatever makes the most sense for your app.

### Runtime event: Showing card details

As the user scans a card, once our machine learning models start to predict numbers, the UI will display these intermediate results while the models finish up their predictions. Specifically, the ViewController will invoke `showScannedCardDetails` with a `CreditCardPrediction` object as the argument each time it has results you can show. Override this function to customize this behavior.

### Runtime event: The camera "Deny" experience

As a platform, iOS provides apps with full control over the deny experience. However, `SimpleScanViewController` also has a deny experience built in. If the user is prompted and denies access to the camera or they have been previously prompted and denied access, `SimpleScanViewController` will hide all non essential UI elements \(the "description" text and the "torch" button\) and show a button with a link to settings where the user can grant permission to the camera for your app. Please note, if the user updates the settings for your app, iOS will restart your app and you'll lose any in-memory state.

{% hint style="info" %}
iOS will restart your app if the user updates their camera permission settings \(or any settings for that matter\)
{% endhint %}

## Partial UI customization: ScanViewController

For a complete description of our UI customization options, please see the source code of our [ScanViewController](https://github.com/getbouncer/cardscan-ios/blob/master/CardScan/Classes/ScanViewController.swift). Our public interface includes protocols for setting strings and we expose some key elements of the design that you can set directly. However, these interfaces change fairly often and aren't very elegant, so we haven't documented them \(on purpose\) and hope that people who want control of their UI will use `SimpleScanViewContoller` instead. Our expected use for `ScanViewController` is to use a mature, well tested, and high converting UI out of the box, with only minor modifications.

