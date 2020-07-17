# Android release guide

A guide to releasing new versions of the Android CardScan SDK.

## Contents

* [About](android-release-guide.md#about)
* [Versioning](android-release-guide.md#versioning)
* [Creating a new Release](android-release-guide.md#creating-a-new-release)
* [Installing to local maven](android-release-guide.md#installing-to-local-maven)
* [Installing to BinTray](android-release-guide.md#installing-to-bintray)

## About

The CardScan library is split into multiple android modules, each of which has its own binary `.aar` file that needs to be independently released. This guide applies to all of the CardScan android modules, however the sample bash commands all reference `cardscan-ui`. To apply these commands to other modules, change the name of the module referenced in the command.

## Versioning

The release version of this library is determined by the value of the `version` field in the `gradle.properties` file. CardScan uses [semantic versioning](https://semver.org/) \(MAJOR.MINOR.PATCH\).

## Creating a new release

1. Increment the version field in `gradle.properties` and create a new github pull request with your changes.
2. Get your PR reviewed, approved, and merged.
3. Create a tag from master branch

   ```bash
    git tag <version_number> -a
   ```

4. Write a description for the tag
5. Push the tag to github

   ```bash
    git push origin --tags
   ```

## Updating the ChangeLog

1. Install the [github-changelog-generator](https://github.com/github-changelog-generator/github-changelog-generator). See the README for that project for installation instructions.
2. put your github personal access token in a file called `github_token` in the root directory of the repository.
3. run:

   ```bash
    github_changelog_generator -u getbouncer -p cardscan-ui-android -t `cat github_token`
   ```

4. Create a new pull request with the updated changelog, get it approved, and merged.

## Installing to local maven

* execute `./gradlew cardscan-ui:install` from the root of the repository.

## Installing to BinTray

1. Update or create a file `local.properties` with your BinTray user and api key

   ```text
    bintray.user=<your_bintray_user>
    bintray.apikey=<your_bintray_apikey>
   ```

2. build and upload the library

   ```bash
    ./gradlew cardscan-ui:test
    ./gradlew cardscan-ui:connectedCheck
    ./gradlew cardscan-ui:ktlint
    ./gradlew cardscan-ui:checkStyle
    ./gradlew cardscan-ui:bintrayUpload
   ```

3. Verify that the new version of the binary is available in [jcenter](https://jcenter.bintray.com/com/getbouncer/).
4. Update the [API Docs](https://github.com/getbouncer/apidocs) to reflect the new version.

