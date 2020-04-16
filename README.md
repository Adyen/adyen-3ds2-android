# 3DS2 Android SDK

With this SDK, you can accept 3D Secure 2.0 payments via Adyen.

<img src="https://user-images.githubusercontent.com/37903534/51109822-c66df780-17f6-11e9-9cd7-0bf74f485682.gif" width="400" />

## Installation

The SDK is available either through [jcenter][dl] or via manual installation.

### Import from jcenter

1. Import the SDK by adding this line to your `build.gradle` file.
```groovy
implementation "com.adyen.threeds:adyen-3ds2:2.1.0-rc08"
```

### Import manually

1. Copy the SDK package `adyen-3ds2-2.1.0-rc08.aar` to the `/libs` folder in your module.
2. Import the SDK by adding this line to your module `build.gradle` file.
```groovy
implementation "com.adyen.threeds:adyen-3ds2:2.1.0-rc08@aar"
```

## Usage

### Creating a transaction

First, create an instance of `ConfigParameters` with values from the additional data retrieved from your call to `/authorise`.
Then, use the on `ThreeDS2Service.INSTANCE` to create a transaction.

```java
ConfigParameters configParameters = new AdyenConfigParameters.Builder(
        directoryServerId, // Retrieved from Adyen.
        directoryServerPublicKey // Retrieved from Adyen.
    ).build();

ThreeDS2Service.INSTANCE.initialize(/*Activity*/ this, configParameters, null, null);

Transaction mTransaction = ThreeDS2Service.INSTANCE.createTransaction(null, null);

AuthenticationRequestParameters authenticationRequestParameters = mTransaction.getAuthenticationRequestParameters();
// Submit the authenticationRequestParameters to /authorise3ds2.
```

Use the `mTransaction`'s `authenticationRequestParameters` in your call to `/authorise3ds2`.

:warning: _Keep a reference to your `Transaction` instance until the transaction is finished._
:warning: _When the transaction is finished successfully or not it must be closed._
:warning: _When the 3DS2 flow is finished the 3DS2 service `ThreeDS2Service.INSTANCE` must be cleaned up._

### Performing a challenge

In case a challenge is required, create an instance of `ChallengeParameters` with values from the additional data retrieved from your call to `/authorise3ds2`.

```java
Map<String, String> additionalData = ...; // Retrieved from Adyen.

ChallengeParameters challengeParameters = new ChallengeParameters();
challengeParameters.set3DSServerTransactionID(additionalData.get("threeds2.threeDS2ResponseData.threeDSServerTransID"));
challengeParameters.setAcsTransactionID(additionalData.get("threeds2.threeDS2ResponseData.acsTransID"));
challengeParameters.setAcsRefNumber(additionalData.get("threeds2.threeDS2ResponseData.acsReferenceNumber"));
challengeParameters.setAcsSignedContent(additionalData.get("threeds2.threeDS2ResponseData.acsSignedContent"));
```

Use these challenge parameters to perform the challenge with the `mTransaction` you created earlier:
```java
mTransaction.doChallenge(/*Activity*/ this, challengeParameters, new ChallengeStatusReceiver() {
                @Override
                public void completed(CompletionEvent completionEvent) {
                    String transactionStatus = completionEvent.getTransactionStatus();
                    // Submit the transactionStatus to /authorise3ds2.
                }

                @Override
                public void cancelled() {
                    // Cancelled by the user or the App.
                }

                @Override
                public void timedout() {
                    // The user didn't submit the challenge within the given time, 5 minutes in this case.
                }

                @Override
                public void protocolError(ProtocolErrorEvent protocolErrorEvent) {
                    // An error occurred.
                }

                @Override
                public void runtimeError(RuntimeErrorEvent runtimeErrorEvent) {
                    // An error occurred.
                }
            }, 5);
```

When the challenge is completed successfully, submit the `transactionStatus` in the `completionEvent` in your second call to `/authorise3ds2`.

## Closing the transaction

Every `Transaction` instance is usable only once, so when the transaction is finished successfluy or not, it should be closed like so:

```java
mTransaction.close();
```

## Cleaning up the 3DS2 service

When the 3DS2 flow is finished, the 3DS2 service `ThreeDS2Service.INSTANCE` must be cleaned up like so:

```java
ThreeDS2Service.INSTANCE.cleanup(/*Activity*/ this);
```

## Customizing the UI

The SDK provides a default challenge flow look and feel that can be customized to fit your App's look and feel
with one or the combination of the following methods.

### Customize the SDK theme

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

### Using UiCustomization class

In case that further UI customizations are needed, the SDK provides some customization options.
These customization options are available through the `UiCustomization` class.
To use them, create an instance of `UiCustomization`, configure the desired properties and pass it during initialization of the `ThreeDS2Service.INSTANCE`.

For example, to change the Toolbar's title text and text color:
```java
ToolbarCustomization toolbarCustomization = new ToolbarCustomization();
toolbarCustomization.setHeaderText("Secure Checkout");
toolbarCustomization.setTextColor("#FFFFFF");

UiCustomization uiCustomization = new UiCustomization();
uiCustomization.setToolbarCustomization(toolbarCustomization);

ThreeDS2Service.INSTANCE.initialize(getContext(), configParameters, null, uiCustomization);
```

## See also

 * [Complete Documentation][docs]

 * [SDK Reference][javadoc]

 * [Troubleshooting][troubleshooting]
 
## License
 
This SDK is available under the Apache License, Version 2.0.
For more information, see the [LICENSE][license] file.

[dl]: http://jcenter.bintray.com/com/adyen/threeds/adyen-3ds2/
[docs]: https://docs.adyen.com/developers/risk-management/3d-secure-2/android-sdk-integration
[javadoc]: https://adyen.github.io/adyen-3ds2-android/
[troubleshooting]: https://github.com/Adyen/adyen-3ds2-android/blob/master/TROUBLESHOOTING.md
[license]: https://github.com/Adyen/adyen-3ds2-android/blob/master/LICENSE
