# Troubleshooting

## iOS

### `ld: warning: Could not find auto-linked library 'swiftFoundation'`
This can occur when adding a Swift pod to a pure objective-c application. Add a bridge header to support swift pods. See
https://stackoverflow.com/a/56176956 for an example.

### `error: Failed to build iOS project. We ran "xcodebuild" command but it exited with error code 65`
There are many possible reasons for this error. Here are a few potential work-arounds all taken from
[react-native issue #25240](https://github.com/facebook/react-native/issues/25240):

* Clear the build folder and rebuild by running the following commands

    ```bash
    cd ios
    rm -rf build/
    pod install
    cd ..
    react-native run-ios
    ```

* Switch to the legacy build system

    See https://stackoverflow.com/a/51089264

* Unlink react-native

   ```bash
   react-native unlink react-native-vector-icons
   react-native run-ios
   ```

* Reinstall cocoapods

    ```bash
    cd ios
    rm -rf Pods;
    rm -rf build/
  
    sudo gem uninstall cocoapods
    sudo gem install cocoapods
  
    pod install
    cd ..
    react-native run-ios
    ```

### `Undefined symbols for architecture x86_64`
This is a [known issue with react-native](https://github.com/react-native-community/upgrade-support/issues/77). There
are two ways to remediate this problem:

* Add `use_frameworks!` to your podfile:
  ```ruby
    use_frameworks!
  ```

  _Note that `Flipper` is not compatible with `use_frameworks!`, and will have to be disabled if in use by deleting these lines_:
  ```ruby
    use_flipper!
    post_install do |installer|
      flipper_post_install(installer)
    end
  ```

* Update your `LD_RUNPATH_SEARCH_PATHS`:

  Add the following line to all of your build configurations across your project:
  ```
  LD_RUNPATH_SEARCH_PATHS = "$(inherited) $(TOOLCHAIN_DIR)/usr/lib/swift/$(PLATFORM_NAME) /usr/lib/swift @executable_path/Frameworks @loader_path/Frameworks";
  ```

## Android

### `Error: spawnSync /.../Andoroid/sdk/platform-tools/adb ENOENT`

* react-native was unable to start the app after installing it. To start it manually, run the following command:
```bash
adb reverse tcp:8081 tcp:8081 && adb shell am start -n com.example/com.example.MainActivity
```