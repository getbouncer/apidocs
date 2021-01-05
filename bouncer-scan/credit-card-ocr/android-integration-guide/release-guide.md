---
description: A guide to releasing new versions of the Android CardScan SDK.
---

# Release guide

## Contents

* [About](release-guide.md#about)
* [Versioning](release-guide.md#versioning)
* [Releasing a new version](release-guide.md#releasing-a-new-version)
* [Update documentation](release-guide.md#update-documentation)
* [Installing to local maven](release-guide.md#optional-installing-to-local-maven)

## About

The CardScan library is split into multiple android modules, each of which has its own binary `.aar` file is independently released. This guide covers the batch release process for releasing all of the modules at once with the same version number.

## Versioning

CardScan uses [semantic versioning](https://semver.org/) \(MAJOR.MINOR.PATCH\).

## Releasing a new version

1. Create a tag from master branch

   ```bash
   git tag -a <version_number> -m "<version description>"
   ```

2. Push the tag to github

   ```bash
   git push origin --tags
   ```

3. Publish the tag on [github](https://github.com/getbouncer/cardscan-android/releases) to create a new release

This will automatically update the changelog, the `gradle.properties` file, and publish a release to bintray and jcenter. To view the status of the automatic release, view the latest [github action](https://github.com/getbouncer/cardscan-android/actions?query=event%3Arelease).

## Update documentation

Update the [API Docs](https://github.com/getbouncer/apidocs/blob/master/card-scan/android-integration-guide/README.md) to reflect the new version.

## \(Optional\) Installing to local maven

* execute `./gradlew install` from the root of the repository.
