---
description: A guide to running end-to-end system tests on the CardScan SDK.
---

# System tests guide

The system tests are end-to-end tests that check the scanning performance of the CardScan SDK. These tests run on Google's FireBase service across a wide range of devices.

This guide details how to run the system tests for the iOS CardScan and CardVerify SDKs.

## Setup

1. Clone the repository

   ```text
   git clone https://github.com/getbouncer/cardscan-system-tests
   ```

2. Install Cocoapods dependencies

   ```text
   cd cardscan-system-tests/ios/CardScanSystemTest
   pod install
   ```

3. Build the test project

   Open the `.xcworkspace` file in XCode, then clean and build the project

   * Product → Clean build folder
   * Product → Build

   _Note: if CardScan changes break CardVerify, modify the CardVerify Podfile to point to `pod CardScan` instead of `pod Cardscan => master`_

## Running the tests

Execute the testlab script

```text
   ./run_testlab.sh
```

This will produce a URL which you can use to check the status of the test and view the results:

```text
   Test results will be streamed to [https://console.firebase.google.com/project/bouncer-identity-service/testlab/histories...]
```

## Troubleshooting

### `CardScanSystemTest.xcworkspace does not exist`

This error occurs if the cocoapods dependencies are not set up. Install the cocoapods dependency in the `ios/CardScanSystemTest` directory.

```text
pod install
```

### `Creating individual test executions...failed`

This error occurs if the project has not been built or the build failed. Repeat the `Build the test project` step above.

### `Matrix [matrix-utqrtitz7ri6a] failed during validation: The XCTest zip file was malformed.`

This is the same error as above. Repeat the `Build the test project` step above.

