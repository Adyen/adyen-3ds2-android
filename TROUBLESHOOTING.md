# 3DS2 Android SDK Troubleshooting

### The App fails to build when integrating the SDK.

In case the App is dependent on `appcompat-v7` version lower than `28.0.0` (e.g. `com.android.support:appcompat-v7:27.1.1`) and since the SDK is dependent on `appcompat-v7:28.0.0`, there might be a need to resovle the `appcompat-v7` dependencies.

#### Option 1: Exclude the 3DS2 SDK appcompat-v7 dependency.

This option excludes the `appcompat-v7` dependency so the 3DS2 SDK will use the App's `appcompat-v7` dependency.

```groovy
implementation ("com.adyen.threeds:adyen-3ds2:<latest-version>") {
    exclude group: 'com.android.support', module: 'appcompat-v7'
}
```

#### Option 2: Resolve the appcompat-v7 dependency for the entire project.

This option resolves all the transitive `appcompat-v7` dependency for the entire project and it's dependencies, also the 3DS2 SDK one.

```groovy
android {
    configurations.all {
        resolutionStrategy.force "com.android.support:appcompat-v7:27.1.1"
    }
}
```

### The App and the 3DS2 challenge look and feel are different.

Override the `ThreeDS2Theme` style attributes by adding the following XML snippet to your App's `styles.xml` file.

:warning: _The SDK theme `ThreeDS2Theme` must inherit from one of AppCompat's theme variants._
* Theme.AppCompat.Light.DarkActionBar
* Theme.AppCompat.Light
* Theme.AppCompat

```xml
<style name="ThreeDS2Theme" parent="Theme.AppCompat.Light.DarkActionBar">
    <!-- Customize the SDK theme here. -->
    <item name="colorPrimary">@color/colorPrimary</item>
    <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
    <item name="colorAccent">@color/colorAccent</item>
</style>
```

### The 3DS2 challenge fails, acsSignedContent cannot be verified.

In case the App is been tested against Android API level 23 and lower the challenge flow will fail
with the following message "acsSignedContent cannot be verified" this is due to the fact that Adyen's
Visa 3DS2 Simulator directory server root CA is of version 1 and not version 3 (will be fixed in the future).
This issue will not affect live transactions since all the live directory servers has root CA of version 3.
For now the only solution is to test against Android API level 24 and higher.