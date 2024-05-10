# 3DS2 Android SDK

With this SDK, you can accept 3D Secure 2.0 payments via Adyen.

<img src="https://user-images.githubusercontent.com/37903534/51109822-c66df780-17f6-11e9-9cd7-0bf74f485682.gif" width="400" />

## Installation

The SDK is available either through [Maven Central][dl] or via manual installation.

### Import from Maven Central

1. Import the SDK by adding this line to your `build.gradle` file.

```groovy
implementation "com.adyen.threeds:adyen-3ds2:2.2.18"
```

### Import manually

1. Copy the SDK package `adyen-3ds2-2.2.18.aar` to the `/libs` folder in your module.
2. Import the SDK by adding this line to your module `build.gradle` file.

```groovy
implementation "com.adyen.threeds:adyen-3ds2:2.2.18@aar"
```

## Usage

### Creating a transaction

First, create an instance of `ConfigParameters` with values from the additional data retrieved from
your call to `/authorise`.
Then use the on `ThreeDS2Service.INSTANCE` to create a transaction.

```kotlin
val configParameters = AdyenConfigParameters.Builder(
    directoryServerId, // Retrieved from Adyen Server
    directoryServerPublicKey, // Retrieved from Adyen Server
    directoryServerRootCertificates, // Retrieved from Adyen Server
).build()

val initializeResult = ThreeDS2Service.INSTANCE.initialize(
    /* Activity */ this,
    configParameters,
    /* Locale */ null,
    /* UI Customization */ null,
)

if (initializeResult is InitializeResult.Failure) {
    // The initialization failed.
    // Submit the initializeResult.additionalDetails and initializeResult.transactionStatus
    // to /authorise3ds2
    return
}

val transactionResult = ThreeDS2Service.INSTANCE.createTransaction(
    null,
    "<MESSAGE_PROTOCOL_VERSION>"
)

if (transactionResult is TransactionResult.Failure) {
    // The creation of transaction failed.
    // Submit the transactionResult.additionalDetails and transactionResult.transactionStatus
    // to /authorise3ds2
    return
}

val transaction = transactionResult.transaction
val authenticationRequestParameters = transaction?.authenticationRequestParameters
// Submit the authenticationRequestParameters to /authorise3ds2.

```

When the transaction is created successfully use the `transaction`'s 
`authenticationRequestParameters` in your call to `/authorise3ds2`.

When the initialize or createTransaction is failed, submit the `additionalDetails` and
the `transactionStatus` in your second call to `/authorise3ds2`.

:warning: _Keep a reference to your `Transaction` instance until the transaction is finished._  
:warning: _When the transaction is finished successfully or not it must be closed._  
:warning: _When the 3DS2 flow is finished the 3DS2 service `ThreeDS2Service.INSTANCE` must be
cleaned up._

### Performing a challenge

When a challenge is required, create an instance of `ChallengeParameters` with values from the
additional data retrieved from your call to `/authorise3ds2`.

:warning: _Because of recent updates to the 3D Secure protocol, we strongly recommend that you
provide the `threeDSRequestorAppURL` parameter as an Android App Link instead of custom link.
This requires your app to handle the provided Android App Link. More details on how to handle
Android App Link can be found on docs [page][applinkdoc]_

```kotlin
val additionalData = resonse.additionalData // Retrieved from Adyen Server

val challengeParameters = ChallengeParameters().apply {
    set3DSServerTransactionID(additionalData.get("threeds2.threeDS2ResponseData.threeDSServerTransID"))
    acsTransactionID = additionalData.get("threeds2.threeDS2ResponseData.acsTransID")
    acsRefNumber = additionalData.get("threeds2.threeDS2ResponseData.acsReferenceNumber")
    acsSignedContent = additionalData.get("threeds2.threeDS2ResponseData.acsSignedContent")
    threeDSRequestorAppURL = "https://{yourdomain.com}/adyen3ds2"
}
```

Use these challenge parameters to perform the challenge with the `transaction` you created earlier:

```kotlin
val challengeStatusHandler = ChallengeStatusHandler { challengeResult ->
    when (challengeResult) {
        is ChallengeResult.Completed -> {
            // Submit the challengeResult.transactionStatus to /authorise3ds2
        }
        is ChallengeResult.Cancelled -> {
            // Challenge cancelled by the user
            // Submit the challengeResult.additionalDetails and challengeResult.transactionStatus 
            // to /authorise3ds2 
        }
        is ChallengeResult.Timeout -> {
            // The user didn't submit the challenge within the given time (default 5 minutes)
            // Submit the challengeResult.additionalDetails and challengeResult.transactionStatus 
            // to /authorise3ds2 
        }
        is ChallengeResult.Error -> {
            // An error occurred
            // Submit the challengeResult.additionalDetails and challengeResult.transactionStatus 
            // to /authorise3ds2 
        }
    }
}

transaction.doChallenge(
    /* Activity */ this,
    challengeParameters,
    challengeStatusHandler,
    /* Timeout in minutes */ 5,
)
```

When the challenge is completed successfully, submit the `transactionStatus` in
the `ChallengeResult.Completed` in your second call to `/authorise3ds2`.

When the challenge is cancelled, times out or an error occurs, submit the `additionalDetails` and 
the `transactionStatus` in the `ChallengeResult.Cancelled`, `ChallengeResult.Timeout` or
`ChallengeResult.Error` in your second call to `/authorise3ds2`.

## Closing the transaction

Every `Transaction` instance is usable only once, so when the transaction is finished successfully
or not, it should be closed like so:

```kotlin
transaction.close()
```

## Cleaning up the 3DS2 service

When the 3DS2 flow is finished, the 3DS2 service `ThreeDS2Service.INSTANCE` must be cleaned up like
so:

```kotlin
ThreeDS2Service.INSTANCE.cleanup(/*Context*/ this)
```

## Customizing the UI

The SDK provides a default challenge flow look and feel that can be customized to fit your App's
look and feel
with one or the combination of the following methods.

### Customize the SDK theme

Override the `ThreeDS2Theme` style attributes by adding the following XML snippet to your
App's `styles.xml` file.

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
To use them, create an instance of `UiCustomization`, configure the desired properties and pass it
during initialization of the `ThreeDS2Service.INSTANCE`.

For example, to change the Toolbar's title text and text color:

```kotlin
val toolbarCustomization = ToolbarCustomization().apply {
    headerText = "Secure Checkout"
    textColor = "#FFFFFF"
}

val uiCustomization = UiCustomization().apply {
    toolbarCustomization = toolbarCustomization
}

ThreeDS2Service.INSTANCE.initialize(
    /* Activity */ this,
    configParameters,
    /* Locale */ null,
    uiCustomization
)
```

## See also

* [Complete Documentation][docs]

* [SDK Reference][javadoc]

* [Troubleshooting][troubleshooting]

## License

This SDK is available under the Apache License, Version 2.0.
For more information, see the [LICENSE][license] file.

[dl]: https://mvnrepository.com/artifact/com.adyen.threeds/adyen-3ds2

[docs]: https://docs.adyen.com/developers/risk-management/3d-secure-2/android-sdk-integration

[applinkdoc]: https://docs.adyen.com/online-payments/classic-integrations/api-integration-ecommerce/3d-secure/native-3ds2/android-sdk-integration#present-a-challenge

[javadoc]: https://adyen.github.io/adyen-3ds2-android/

[troubleshooting]: https://github.com/Adyen/adyen-3ds2-android/blob/main/TROUBLESHOOTING.md

[license]: https://github.com/Adyen/adyen-3ds2-android/blob/main/LICENSE
