---
description: >-
  This document outlines the advanced customization options available in the
  CardVerify framework.
---

# Advanced customization guide

Parts of the verify flow can be customized to perform better on slower devices at the expense of some user experience on faster devices. These customization options are available in the `VerifyCardViewController` and `VerifyCardAddViewController` class.

## Configuration

When instantiating `VerifyCardViewController` or `VerifyCardAddViewController`, you have the option to set `scanPerformancePriority` and `maxErrorCorrectionDuration`.

_Note that you should only consider configuring the verify flow with devices iOS 13.0 and higher._

### Recommended configuration for markets with older devices

In markets that primarily have older iOS devices, we recommend that you add the following to your code before presenting the `VerifyCardViewController` or `VerifyCardAddViewController`:

{% tabs %}
{% tab title="Swift" %}
```swift
@IBAction func pressVerifyCardViewControllerButton() {
    let vc = VerifyCardViewController(userId: "user_id", lastFour: "7890", bin: "123456", cardNetwork: .UNKNOWN)
    vc.scanPerformancePriority = .accurate
    vc.maxErrorCorrectionDuration = 4.0
    vc.verifyCardDelegate = self
    self.present(vc, animated: true)
}

@IBAction func pressVerifyAddCardViewControllerButton() { 
    let vc = VerifyCardAddViewController(userId: "user_id")
    vc.scanPerformancePriority = .accurate
    vc.maxErrorCorrectionDuration = 4.0
    vc.cardAddDelegate = self
    self.present(vc, animated: true)
}
```
{% endtab %}

{% tab title="Objective-C" %}
```swift
- (IBAction)pressVerifyCardViewControllerButton:(id)sender {
    VerifyCardViewController *vc = [[VerifyCardViewController alloc] initWithUserId:@"userid" lastFour:@"7890" bin:@"1234"];
    vc.scanPerformancePriority = ScanPerformanceAccurate;
    vc.maxErrorCorrectionDuration = 4.0;
    vc.verifyCardDelegate = self;
    [self presentViewController:vc animated:YES completion:nil];
}

- (IBAction)pressVerifyAddCardViewControllerButton:(id)sender {
    VerifyCardAddViewController *vc = [[VerifyCardAddViewController alloc] initWithUserId:@""];
    vc.scanPerformancePriority = ScanPerformanceAccurate;
    vc.maxErrorCorrectionDuration = 4.0;
    vc.cardAddDelegate = self;
    [self presentViewController:vc animated:YES completion:nil];
}
```
{% endtab %}
{% endtabs %}

_Note that settings these values will make the scan take longer overall, but with a much higher chance of extracting the name and expiry._

### Fast Scan Performance

The fast scanning option is the default behavior for all scanning flows. It has a non-configurable maximum error correction duration of 2 seconds.

| field | default value |
| :--- | :--- |
| VerifyViewController.scanPerformancePriority | `fast` |
| VerifyViewController.maxErrorCorrectionDuration | `2.seconds` |

### Accurate Scan Performance

The accurate scanning option is the configurable behavior for `VerifyCardViewController` and `VerifyCardAddViewController`. While the overall scan time will be much slower, the longer scan time can improve the accuracy of the name and expiry extraction.

| field | default value |
| :--- | :--- |
| VerifyViewController.scanPerformancePriority | `accurate` |
| VerifyViewController.maxErrorCorrectionDuration | `4.seconds` |

