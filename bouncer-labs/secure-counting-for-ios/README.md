# Secure Counting

Bouncer's secure counting is a counter that is based on security hardware found on modern smartphones. Our secure counter is a novel abstraction where Bouncer tracks events, like cards added, on a per-device basis. These events help app builders detect attacks and track devices that attackers have used previously. As a first-class design consideration our secure counters maintain end-user privacy and remain intact even across factory resets of the device.

You can learn more about our secure counting abstraction:

* [Whitepaper](../../assets/secure_counting_whitepaper.pdf) that describes why one would want to count and the basics behind how it works from a conceptual level.
* [API docs](secure_counting_rest.md) that describe the REST interfaces for accessing secure counters.
* [iOS integration guide](secure-counting-for-ios.md) that show how to provision a DeviceCheck certificate for your iOS app so that Bouncer can use it for secure counting.

## Setting up secure counting

Before you get started with secure counting, reach out to Bouncer to get setup. You'll need to identify which events you want to count and the maximum value for the count -- we'll help provide guidance on what's appropriate for your app.

