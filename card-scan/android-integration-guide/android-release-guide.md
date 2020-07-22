---
description: >-
  A guide to releasing new versions of the Android CardScan SDK.
---

# Android release guide

## Contents
* [About](#about)
* [Versioning](#versioning)
* [Releasing a new version](#releasing-a-new-version)
* [Update documentation](#update-documentation)
* [Installing to local maven](#optional-installing-to-local-maven)

## About
The CardScan library is split into multiple android modules, each of which has its own binary `.aar` file that needs to be independently released. This guide applies to all of the CardScan android modules, however the sample bash commands all reference `cardscan-ui`. To apply these commands to other modules, change the name of the module referenced in the command.

## Versioning
CardScan uses [semantic versioning](https://semver.org/) \(MAJOR.MINOR.PATCH\).

## Releasing a new version
1. Create a tag from master branch
   ```bash
    git tag -a <version_number> -m "<version description>"
   ```
1. Push the tag to github
   ```bash
    git push origin --tags
   ```
1. Publish the tag on [github](https://github.com/getbouncer/cardscan-android/releases) to create a new release

This will automatically update the changelog, the `gradle.properties` file, and publish a release to bintray and jcenter. To view the status of the automatic release, view the latest [github action](https://github.com/getbouncer/cardscan-android/actions?query=event%3Arelease).

## Update documentation
Update the [API Docs](https://github.com/getbouncer/apidocs/blob/master/card-scan/android-integration-guide/README.md) to reflect the new version.

## (Optional) Installing to local maven
* execute `./gradlew install` from the root of the repository.
