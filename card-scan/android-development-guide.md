# Development
A guide to developing the CardScan SDK and using some of its more advanced features.

## Contents
* [Code Organization](#code-organization)
* [Building](#building)
* [Running Unit Tests](#running-unit-tests)
* [Running Android Instrumentation Tests](#running-android-tests)
* [Releasing](#releasing)

## Code Organization
The CardScan SDK is built from multiple android modules, each of which builds upon more base modules. The structure of the modules used in CardScan looks like this:

![module structure](../.gitbook/assets/development/cardscan/android/module_dependencies.png)

The CardScan SDK consists of these modules as well as a demo app:
* [scan-framework](#scan-framework)
* [scan-camera](#scan-camera)
* [scan-payment](#scan-payment)
* [scan-ui](#scan-ui)
* [cardscan-ui](#cardscan-ui)
* [cardscan-demo](#cardscan-demo)

### scan-framework
The [scan-framework](https://github.com/getbouncer/scan-framework-android) contains the framework used to scan images. See the [architecture document](https://github.com/getbouncer/scan-framework-android/tree/master/docs/architecture.md) for details on how it processes images from the camera.

### scan-camera
The [scan-camera](https://github.com/getbouncer/scan-camera-android) contains the camera interfaces for setting up the camera on the device and receiving images from it. It also handles converting the images from the camera to a processable format.

### scan-payment
The [scan-payment](https://github.com/getbouncer/scan-payment-android) contains the code for validating and managing payment cards.

### scan-ui
The [scan-ui](https://github.com/getbouncer/scan-ui-android) contains some common functionality shared between user interfaces for scanning cards. This provides the card viewfinder, debug overlay, and base scan activity.

### cardscan-ui
The [cardscan-ui](https://github.com/getbouncer/cardscan-ui-android) android module builds on top of the [scan-card-ui](https://github.com/getbouncer/scan-card-ui-android) and [scan-payment](https://github.com/getbouncer/scan-payment) modules to provide user interfaces for scanning payment cards. Anything specific to the user interface of the CardScan product lives in this library.

### cardscan-demo
The [cardscan-demo](https://github.com/getbouncer/cardscan-demo-android) android app provides a demonstration for launching the CardScan SDK as a separate activity and for integrating the CardScan library into an existing activity.

## Building
Check out the project using `git`. Note that this project makes use of submodules, so a `recursive` clone is recommended.
```bash
git clone --recursive https://github.com/getbouncer/cardscan-ui-android
```

To build the project, run the following command:
```bash
./gradlew cardscan-ui:build
```

To create an AAR release of the app, run the following command:
```bash
./gradlew cardscan-ui:assembleRelease
```
This will place an AAR file in `cardscan-ui/build/outputs/aar`

## Contributing
These libraries follow a standard github contribution flow.

1. Create a new github feature branch
    ```bash
    git checkout -b <your_github_name>/<your_feature_name>
    ```

1. Make your code changes

1. Push your branch to origin
    ```bash
    git push --set-upstream origin <your_branch_name>
    ```

1. Create a new pull request on github, and tag appropriate owners.

1. Once you have approval, merge your branch into master and delete your feature branch from github.

## Running Unit Tests
Unit tests can be run from android studio or from the command line. To execute from the command line, run the following command:
```bash
./gradlew test
```

## Running Android Tests
Android tests can be run from android studio or from the command line. To execute from the command line, run the following command:
```bash
./gradlew connectedAndroidTest
```

Note that these tests require that you have an emulator running or a physical device connected to your machine via `ADB`.

## Releasing
See the [release](android-release-guide.md) documentation.