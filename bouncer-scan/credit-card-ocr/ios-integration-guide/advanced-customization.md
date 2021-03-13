---
description: >-
  This document outlines the advanced customization options available in the
  CardScan framework.
---

# Advanced Customization

Parts of the scan flow can be customized to perform better on slower devices at the expense of some user experience on faster devices. These customization options are available in the `SimpleScanViewController` class.


## Configuration

When instantiating `SimpleScanViewController`, you have the option to set `scanPerformancePriority` and `maxErrorCorrectionDuration`.

_Note that you should only consider configuring the scan flow with devices iOS 13.0 and higher._

### Recommended configuration for markets with older devices

In markets that primarily have older iOS devices, we recommend that you add the following to your code before presenting the `SimpleScanViewController`:

{% tabs %}
{% tab title="Swift" %}
```swift
@IBAction func pressSimpleScanViewControllerButton() {
    let vc = SimpleScanViewController.createViewController()
    vc.scanPerformancePriority = .accurate
    vc.maxErrorCorrectionDuration = 4.0
    vc.delegate = self
    self.present(vc, animated: true)
}
```
{% endtab %}

{% tab title="Objective-C" %}
```swift
- (IBAction)pressSimpleScanViewControllerButton:(id)sender {
    SimpleScanViewController *vc = [SimpleScanViewController createViewController];
    vc.scanPerformancePriority = ScanPerformanceAccurate;
    vc.maxErrorCorrectionDuration = 4.0;
    vc.delegate = self;
    [self presentViewController:vc animated:YES completion:nil];
}
```
{% endtab %}
{% endtabs %}

_Note that settings these values will make the scan take longer overall, but with a much higher chance of extracting the name and expiry._

### Fast Scan Performance 
The fast scanning option is the default behavior for all scanning flows. It has a non-configurable maximum error correction duration of 2 seconds.

| field | default value |
| ----- | ------------- |
| SimpleScanViewController.scanPerformancePriority | `fast` |
| SimpleScanViewController.maxErrorCorrectionDuration | `2.seconds` |

### Accurate Scan Performance 
The accurate scanning option is the configurable behavior for `SimpleScanViewController`. While the overall scan time will be much slower, the longer scan time can improve the accuracy of the name and expiry extraction.

| field | default value |
| ----- | ------------- |
| SimpleScanViewController.scanPerformancePriority | `accurate` |
| SimpleScanViewController.maxErrorCorrectionDuration | `4.seconds` |