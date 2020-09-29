# Linking native dependencies

For react-native version 0.59 and below, it is necessary to link the native libraries into react-native.

## Automatic

The following bash command will automatically link react-native-cardverify into your react-native app.

```bash
$ react-native link react-native-cardverify
```

## Manual

If instead, you want to manually link react-native-cardverify, perform the following steps.

**iOS**

1. In XCode, in the project navigator, right click `Libraries` ➜ `Add Files to [your project's name]`
2. Go to `node_modules` ➜ `react-native-cardverify` and add `RNCardVerify.xcodeproj`
3. In XCode, in the project navigator, select your project. Add `libRNCardVerify.a` to your project's `Build Phases` ➜ `Link Binary With Libraries`
4. Run your project `Cmd+R`

**Android**

1. Insert the following lines inside the dependencies block in `android/app/build.gradle`:

   ```text
   implementation project(':react-native-cardverify')
   ```

2. Append the following lines to `android/settings.gradle`:

   ```text
   include ':react-native-cardverify'
   project(':react-native-cardverify').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-cardverify/android')
   ```

3. Open `android/app/src/main/java/[...]/MainApplication.java`
   * Add `import com.getbouncer.RNCardVerifyPackage;` to the imports at the top of the file
   * Add `new RNCardVerifyPackage()` to the list returned by the `getPackages()` method

