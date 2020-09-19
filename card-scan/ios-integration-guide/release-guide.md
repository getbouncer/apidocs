---
description: A guide to releasing new versions of the iOS CardScan SDK.
---

# Release Guide

## Contents

* [Versioning](release-guide.md#versioning)
* [Releasing a new version](release-guide.md#releasing-a-new-version)
* [Bumping a model](release-guide.md#bumping-a-model)
* [Update documentation](release-guide.md#update-documentation)

## Versioning
CardScan uses [semantic versioning](https://semver.org/) \(MAJOR.MINOR.PATCH\).

## Releasing a new version

### Prepare the release
1. Bump library version in the `CardScan.podspec` file.

1. Update the cocoapods lock file and commit the new lock file.
   ```bash
   pod update
   ```

### Run required tests
1. Run [CardScan systems iOS test](system-test-guide.md). Make sure to watch the videos to double check that everything
   looks good.

1. Verify Carthage build is working in the same directory as `.xcodeproj`

   ```bash
   brew install carthage
   carthage build --no-skip-current
   ```
   
   *  If you get the error: `no shared framework schemes`, reclick `shared` on the project schemes in xcode. (product → schemes → manage schemes)
   *  Make sure to commit files that are modified after reclicking `shared` 

1. Run the Cocoapods linter to make sure that everything is going to pass

   ```bash
   pod lib lint
   ```

### Release the new version
1. Create a new github tag with the release version

   ```bash
   git tag <version>
   git push --tags
   ```

1. Update the changelog
   ```bash
   # checkout https://github.com/github-changelog-generator/github-changelog-generator for installation instructions
   # put your github token in a file called github_token in the base directory
   # run:
   github_changelog_generator -u getbouncer -p cardscan-ios -t `cat github_token` 
   ```

1. Check the updated Changelog manually and update any entries that need updating.

1. Publish to CocoaPods

   ```bash
   pod trunk push
   ```

## Bumping a model

1. Put the new model in the `OriginalModels` directory

1. Remove the old compiled version from the resources directory

   ```bash
   rm -rf CardScan/Assets/FindFour.mlmodelc
   ```

1. Compile the new model

   ```bash
   xcrun coremlc compile OriginalModels/FindFour.mlmodel CardScan/Assets
   ```

## Update documentation
Update the [API Docs](https://github.com/getbouncer/apidocs/blob/master/card-scan/android-integration-guide/README.md)
to reflect the new version.
