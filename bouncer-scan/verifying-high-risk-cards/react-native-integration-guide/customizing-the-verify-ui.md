# Customizing the Verify UI

## iOS UI Customization

You can customize `VerifyCardViewController` and `VerifyCardAddViewController` by setting key-values dictionary in your project's `App.js`. Please see the section below for customizing the remaining components.

### Fonts

All font values should be described as `{font-family}-{style}` like `Avenir-Light`. All the available system fonts and their appropriate names can be found [here](http://iosfonts.com/).

### Available Customizations

| Key | Description | Value Type | Value Example | Default |
| :--- | :--- | :--- | :--- | :--- |
| backButtonTintColor | A hex string of the color overlay of the back button | String | “\#FFF” | “\#FFF” |
| backButtonText | A string describing the text of the back button | String | “Close” | “” |
| backgroundColor | A hex string of the background color | String | “\#FFF” | “\#181b20” |
| backgroundColorOpacity | A float between 0 - 1 of the background color opacity | Float | 0.5 | 0.7 |
| cardDetailExpiryTextColor | A hex string of the card's expiration text color | String | “\#FFF” | “\#FFF” |
| cardDetailNameTextColor | A hex string of the card's name text color | String | “\#FFF” | “\#FFF” |
| cardDetailNumberTextColor | A hex string of the card's number text color | String | “\#FFF” | “\#FFF” |
| descriptionHeaderText | A string describing the text of the description header located above the ROI view | String | “Scan Card” | “Scan Card” |
| descriptionHeaderTextColor | A hex string of the description header color | String | “\#FFF” | “\#FFF” |
| descriptionHeaderTextFont | A string describing the font of the description header | String | “Avenir-Bold” | System Default |
| descriptionHeaderTextSize | A float describing the size of the description header font | Float | 30.0 | 30.0 |
| enableCameraPermissionText | A string describing the text show when the camera permissions are denied. This should ask the user to turn on the camera permissions for your app | String | "Enable your camera permissions” | "Enable your camera permissions” |
| enableCameraPermissionTextColor | A hex string of the enable camera permission text color | String | “\#FFF” | “\#FFF” |
| enableCameraPermissionTextFont | A string describing the font of the enable camera permission text | String | “Avenir-Light” | System Default |
| enableCameraPermissionTextSize | A float describing the size of the enable camera permission text | Float | 20.0 | 20.0 |
| enableManualCardEntry | A boolean when true shows the manual card entry button when there is no card base card | Bool | false | true |
| instructionText | A string describing scanning instructions for the user | String | “Position your card” | “” |
| instructionTextColor | A hex string describing the instruction text color | String | “\#FFF” | “\#FFF” |
| instructionTextFont | A string describing the font of the instruction text | String | “Avenir-Light” | System Default |
| instructionTextSize | A float describing the size of the instruction text | Float | 20.0 | 20.0 |
| manualCardEntryText | A string describing the manual card entry text | String | “Enter Card Manually” | “Enter card details manually” |
| manualCardEntryTextColor | A hex string of the manual card entry text color | String | “\#FFC0CB” | “\#FFF” |
| manualCardEntryTextFont | A string describing the font of the manual card entry text | String | “Avenir-Light” | System Default |
| manualCardEntryTextSize | A float describing the size of the manual card entry text | Float | 20.0 | 20.0 |
| roiBorderColor | A hex string describing the border color of the ROI view | String | “\#FFF” | “\#FFF” |
| roiCornerRadius | A float describing the corner radius of the ROI view | Float | 5.0 | 15.0 |
| torchButtonTintColor | A hex string of the color overlay of the torch button | String | “\#FFF” | “\#FFF” |
| torchButtonText | A string describing the text of the torch button | String | “Torch” | “” |
| torchButtonPosition | An integer either `0` or `1`. `0` designates the torch to be in the top right position while `1` designates the torch to be in the bottom middle position. | Int | 1 | 0 |
| wrongCardText | A string describing the text that will replace the description header when the scanning card does not match the base card | String | “Wrong Card” | “Wrong Card” |

### Images

Images must be added to the native iOS project’s `Image.xcasset` folder and the must be named _**exactly**_ as listed above.

| Image Names | Description |
| :--- | :--- |
| bouncer\_verify\_flash\_on | Image of button to turn on flash |
| bouncer\_verify\_flash\_off | Image of button to turn off flash |
| bouncer\_verify\_back | Image of button to go back |

![](https://paper-attachments.dropbox.com/s_A67C98A02E44F0767B57071179897EF0D9DF90C5FCBE9E5BF877D18556A0D9AF_1626819262306_Screen+Shot+2021-07-20+at+3.14.19+PM.png)

### Localization

By default, we provide internationalization support and translations for all of our strings in English, Spanish, French, Portuguese, German, Chinese \(simplified and traditional\), Hindi, Hebrew, Japanese, Korean, Arabic, and Indonesian. However, if you want to set strings yourself, you can do so by setting the available text fields listed above.

#### UI Customization Example

```javascript
// App.js

// Call setiOSVerifyViewStyle before scan
if (Platform.OS === 'ios') {
    CardVerify.setiOSVerifyViewStyle({
      'backButtonTintColor': '#000',
      'backButtonText': 'Close',

      'backgroundColor': '#FFC0CB',
      'backgroundColorOpacity': 0.5,

      'cardDetailExpiryTextColor': '#FFF',
      'cardDetailNameTextColor': '#FFF',
      'cardDetailNumberTextColor': '#FFF',

      'descriptionHeaderText': 'Scan Card',
      'descriptionHeaderTextColor': '#FFF',
      'descriptionHeaderTextFont': 'Avenir-Heavy',
      'descriptionHeaderTextSize': 30.0,

      'enableCameraPermissionText': "Enable your camera permissions",
      'enableCameraPermissionTextColor': '#000',
      'enableCameraPermissionTextFont': 'Avenir-LightOblique',
      'enableCameraPermissionTextSize': 20.0,

      'enableManualCardEntry': true,

      'instructionText': 'Please center your card in the middle',
      'instructionTextFont': 'Avenir-Light',
      'instructionTextColor': '#00ff00',
      'instructionTextSize': 20.0,

      'manualCardEntryText': 'Enter your card manually',
      'manualCardEntryTextColor': '#0000ff',
      'manualCardEntryTextFont': 'Avenir-BookOblique',
      'manualCardEntryTextSize': 25.0,

      'roiBorderColor': '#ff0000',
      'roiCornerRadius': 40.0,

      'torchButtonTintColor': '#000',
      'torchButtonText': 'Torch',
      'torchButtonPosition': 0 <topRight> | 1 <bottom>,

      'wrongCardText': 'This is the wrong card'
    });
}

const { action, scanId, payload, canceledReason } = await CardVerify.scan(bin, lastFour, true);
```

