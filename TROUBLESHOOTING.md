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