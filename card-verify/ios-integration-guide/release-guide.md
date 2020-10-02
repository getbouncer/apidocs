---
description: A guide to releasing new versions of the iOS CardVerify SDK.
---

# Release guide

## Contents

* [Versioning](release-guide.md#versioning)
* [Releasing a new version](release-guide.md#releasing-a-new-version)
* [Bumping a model](release-guide.md#bumping-a-model)
* [Update documentation](release-guide.md#update-documentation)

## Versioning

CardVerify uses [semantic versioning](https://semver.org/) \(MAJOR.MINOR.PATCH\).

## Releasing a new version

### Prepare the release

1. Bump library version in the `CardVerify.podspec` file.

    A. If you're also planning to release a new version of CardScan, make sure to:
    
    - Bump CardScan first
    - Update the CardScan dependency in the CardVerify.podspec to the new version
    - Make sure the example Podfile is pointing to production CardScan in Cocoapod

### Run required tests

1. Run [CardScan systems iOS test](../../card-scan/ios-integration-guide/system-test-guide.md).
Make sure to watch the videos to double check that everything looks good.

2. Run the Cocoapods linter to make sure that everything is going to pass

   ```bash
   pod lib lint
   ```

### Release the new version

1. Create a new github tag with the release version

   ```bash
   git tag <version>
   git push --tags
   ```

2. Update the changelog

   ```bash
   # checkout https://github.com/github-changelog-generator/github-changelog-generator for installation instructions
   # put your github token in a file called github_token in the base directory
   # run:
   github_changelog_generator -u getbouncer -p cardverify-ios -t `cat github_token`
   ```

3. Check the updated Changelog manually and update any entries that need updating.

4. Run the `deploay.sh` script to create a new release

## Bumping a model

1. Put the new model in the `OriginalModels` directory
2. Remove the old compiled version from the resources directory

   ```bash
   rm -rf CardVerify/Assets/FindFour.mlmodelc
   ```

3. Compile the new model

   ```bash
   xcrun coremlc compile OriginalModels/FindFour.mlmodel CardVerify/Assets
   ```

## Update documentation

Update the [API Docs](../android-integration-guide/README.md) to reflect the new version.

