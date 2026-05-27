# Adyen 3D Secure 2 Android SDK - Data Safety Guide

For App Developers

**Purpose of this document:** This guide helps you complete the Data Safety section in Google Play Console for an app that integrates the Adyen 3D Secure 2 Android SDK.
The SDK collects device data and other data from device as required by the EMVCo 3DS2 specification for Risk-Based Authentication (RBA). All collected data is encrypted and sent to the issuer's Access Control Server (ACS). You **must** disclose this data in your Play Console submission.

## 1. Background

The 3D Secure 2 (EMV 3-D Secure 2.x) protocol requires the SDK to collect a broad set of device environment data elements - formally called the **Device Information** object - and submit them to the issuing bank's ACS as part of the authentication request. This is mandated by the EMVCo 3DS2 specification and is the mechanism that enables frictionless authentication without an One-Time Password or any other type of challenge for low-risk transactions.

Google Play's Data Safety policy requires every app to declare all user data collected, transmitted, or shared - regardless of whether the data is visible to the user.

> **Warning**
> As the integrating developer, you are responsible for the Data Safety declarations of your app, including data collected by third-party SDKs that you embed. Failure to accurately disclose data collection can result in your app being removed from Google Play.

## 2. Complete Data Inventory

The table below lists every data element the SDK may collect, its purpose, whether it is shared with third parties (ACS / payment networks), and whether it is stored persistently on-device.

> **Note**
> The following table represents the **maximum** scope of data that might be collected. Data collection is conditional based on:
> 1. **Permissions:** Many data points are only collected if your app holds the corresponding Android permission. If a permission is not granted, the SDK gracefully skips that data point.
> 2. **Configuration:** You can optionally block specific EMVCo device parameters using the `deviceParameterBlockList` in `AdyenConfigParameters.Builder`. This blocklist accepts EMVCo defined IDs (for example, `A001`), which can be found in the **EMVCo 3-D Secure SDK Device Information** document available on the EMVCo website.
>
> See [Section 3: Android Permissions Reference](#3-android-permissions-reference) for a detailed mapping of how permissions affect data collection. If you exclude parameters or do not hold certain permissions, you should adjust your Play Console declarations accordingly.

| Data Category | Specific Data                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | Purpose | Shared w/ 3rd Party | Persisted on Device |
| --- |-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------| --- | --- | --- |
| **Device or other IDs** | Android ID, IMEI/MEID, IMSI, SIM Serial, Bluetooth MAC (local/bonded), Bonded Device Aliases (Android 11+), Connected Device Address Types (Android 15+), Bluetooth Status/Discoverability, Device Model/Name & Hardware/Build info, OS Version & Locale, IPv4/IPv6, Wi-Fi SSID/BSSID & Capabilities, Cellular/Network Operator info & SIM state, Storage capacity, Display metrics.<br><br>*Note: IMEI/MEID, IMSI, and SIM Serial are collected if the app holds `READ_PRIVILEGED_PHONE_STATE` (on any Android version) or `READ_PHONE_STATE` (on Android 9 / API 28 and below). Collecting the local hardware Bluetooth MAC address strictly requires **both** `LOCAL_MAC_ADDRESS` and `BLUETOOTH_CONNECT` (or `BLUETOOTH`). (`LOCAL_MAC_ADDRESS` and `READ_PRIVILEGED_PHONE_STATE` are privileged system permissions not typically held by third-party apps). If your app does not hold these permissions, do not declare these specific identifiers.* | **App functionality** (risk-based authentication (RBA) fingerprinting) and **Fraud prevention, security, and compliance** | **Yes** | **No** |
| **Personal info** | Phone Number (including Line 1 and Voicemail numbers).<br><br>*Note: Collected if the app holds **any** of the following permissions: `READ_PHONE_STATE`, `READ_PHONE_NUMBERS`, `READ_SMS`, or `READ_PRIVILEGED_PHONE_STATE`. If your app does not hold any of these permissions, do not declare it.*                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | **Fraud prevention, security, and compliance** | **Yes** | **No** |
| **Location** | Latitude & Longitude.<br><br>*Note: Collected if the app holds `ACCESS_COARSE_LOCATION` or `ACCESS_FINE_LOCATION`. If your app does not hold any of these permissions, do not declare it.*                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | **Fraud prevention, security, and compliance** | **Yes** | **No** |
| **App info and performance** | Package name, App UUID, Installer package name, App state & features (for example Safe Mode, Archivable), Device Settings (System/Global/Secure), WebView User Agent, Root status, Emulator flag, Debugger attach.<br><br>*Note: RASP/threat signals are classified as 'App diagnostics'.*                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | **Fraud prevention, security, and compliance** (RBA integrity check & RASP) | **Yes** | **No** |
| **Financial info** | SDK Transaction ID, 3D Secure Server Transaction ID, ACS Transaction ID, ACS Ref Number.<br><br>*Note: **No raw PAN or CVV is collected by the SDK.***                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | **App functionality** (3D Secure 2 session linkage) and **Fraud prevention, security, and compliance** | **Yes** | **No** |
| **App activity** | Challenge flow inputs (for example OTP codes or selection inputs).<br><br>*Note: Collected only during an active challenge flow.*                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | **App functionality** (Secure transmission to issuer ACS) | **Yes** | **No** |

**Key:** All data is encrypted into a JWE (JSON Web Encryption) payload and transmitted to the issuer's ACS. No collected data element is persisted on the device after the authentication session ends.

## 3. Android Permissions Reference

The SDK benefits from the following permissions to collect device data. The SDK relies on the permissions naturally declared by your app.

> **Important**
> The SDK does **not** require your app to hold these permissions. It gracefully skips any data points that rely on permissions you have not declared.
> To minimize your app's Data Safety disclosure scope, only request high-scrutiny permissions (like Location or Phone State) if your app genuinely needs them for its own features.

| Android Permission | Required?    | Scrutiny Level | Notes |
| --- |--------------| --- | --- |
| `ACCESS_COARSE_LOCATION` | **Optional** | **High** | Enables approximate location collection. |
| `ACCESS_FINE_LOCATION` | **Optional** | **High** | Enables precise location collection. |
| `NEARBY_WIFI_DEVICES` | **Optional** | **High** | Enables Wi-Fi usable channels collection (Android 13+). |
| `REQUEST_INSTALL_PACKAGES` | **Optional** | **High** | Enables checking if the device can install non-market apps. |
| `READ_PHONE_STATE` | **Optional** | **High** | Enables collection of hardware IDs (Android 9 and below) and phone number (Android 10 and below). |
| `READ_PHONE_NUMBERS` / `READ_SMS` | **Optional** | **High** | Used as a fallback to retrieve the phone number on newer Android versions. |
| `READ_BASIC_PHONE_STATE` | **Optional** | **Medium** | Enables network premium capabilities check (Android 13+). |
| `ACCESS_WIFI_STATE` | **Optional** | **Medium** | Enables collection of Wi-Fi SSID, BSSID, and network capabilities. |
| `BLUETOOTH` / `BLUETOOTH_CONNECT` | **Optional** | **Medium** | Enables collection of Bluetooth Device Name, bonded device MAC addresses, bonded device aliases (Android 11+), Bluetooth state, discoverability settings, and connected device address types (Android 15+). Also required, together with `LOCAL_MAC_ADDRESS`, to retrieve the local hardware Bluetooth MAC address. |
| `ACCESS_NETWORK_STATE` | **Optional** | **Low** | Enables network type and IP address detection. |
| `READ_PRIVILEGED_PHONE_STATE` | **Optional** | **Privileged** | System apps only. Enables collection of hardware IDs (Android 10+). |
| `LOCAL_MAC_ADDRESS` | **Optional** | **Privileged** | System apps only. Required together with `BLUETOOTH_CONNECT` (or `BLUETOOTH`) to successfully retrieve the local hardware Bluetooth MAC address. |


## 4. Prominent Disclosure and User Consent Requirement

If your app's configuration allows the SDK to collect certain sensitive device identifiers (such as the phone number, or precise location) as detailed in Section 2, and this collection is not reasonably expected by the user in the context of your app's core functionality, Google Play's [User Data policy](https://support.google.com/googleplay/android-developer/answer/11150561) requires you to provide a prominent, in-app disclosure and obtain explicit user consent **before** the data is collected.

This disclosure must:
*   Be displayed in the normal usage of the app (not hidden in the settings or a privacy policy).
*   Describe the data being collected and how it will be used (for example, "This app collects your phone number for fraud prevention and authentication purposes during checkout").
*   Require affirmative user action (for example, tapping an "Accept" or "I Agree" button).

**Action required:** Review the permissions your app requests. If you request permissions like `READ_PHONE_STATE`, `READ_PHONE_NUMBERS`, `ACCESS_FINE_LOCATION`, or access to sensitive device IDs that trigger collection by the 3D Secure 2 SDK, you must implement this prominent disclosure and consent flow in your app's UX prior to initializing a 3D Secure 2 transaction.

> **Note**
> If you hold these permissions but do not want the SDK to collect this specific sensitive data (and thereby avoid the need for this prominent disclosure), you can explicitly block the SDK from collecting them by using the `deviceParameterBlockList` in the `AdyenConfigParameters.Builder`. This blocklist accepts EMVCo defined IDs (for example `A001`), which can be found in the **EMVCo 3-D Secure SDK Device Information** document available on the EMVCo website.

## 5. Privacy Policy Requirements

Your app's Privacy Policy must cover the following points to satisfy both Google Play requirements and applicable privacy regulations (GDPR, CCPA, etc.):

1. **Authentication data collection:** Explain that the app uses the 3D Secure 2 protocol which collects device, network, (optionally) location data and (optionally) phone number for the purpose of authenticating payment transactions.
2. **Third-party sharing:** Disclose that device data is transmitted to the card issuer's ACS and may pass through the payment network (such as Visa or Mastercard) and the 3DS Server.
3. **Encryption:** State that all data collected by the SDK is encrypted in transit using JWE (JSON Web Encryption) before transmission.
4. **No persistent storage:** Clarify that the SDK does not store any collected device data on-device after the authentication session concludes.
5. **Legal basis:** Cite legitimate interests / contractual necessity (the user has initiated a payment transaction that requires strong authentication).
6. **User rights:** Provide a contact mechanism for users to ask questions about data usage, as required by GDPR Art. 13/14.

## 6. Common Mistakes & How to Avoid Them

| Common Mistake | Correct Approach                                                                                                       |
| --- |------------------------------------------------------------------------------------------------------------------------|
| Declaring only first-party data and omitting SDK collections | Review all embedded SDKs. The 3D Secure 2 SDK collects significant data - it must be declared even if your own code does not. |
| Answering 'No' to data sharing | Google defines 'sharing' as any transmission to a third party. The ACS is a third party. Answer Yes.                   |
| Declaring location when the app does not hold location permissions | Only declare location if your app manifest includes `ACCESS_COARSE_LOCATION` or `ACCESS_FINE_LOCATION`.                |
| Omitting phone number | If `READ_PHONE_STATE` (or similar) is in your manifest, declare phone number under Personal info.                      |
| Not declaring RASP/threat signals as App info and performance | Root/emulator/debugger flags are diagnostics data. Check App info and performance → App diagnostics.                   |

## 7. Glossary

| Term | Definition                                                                                                                                              |
| --- |---------------------------------------------------------------------------------------------------------------------------------------------------------|
| **ACS** | Access Control Server - the issuing bank's server that authenticates the cardholder using the device data provided by the 3DS2 SDK.                     |
| **EMVCo 3DS2** | The EMV 3-D Secure 2.x specification that mandates the collection of device environment data elements for Risk-Based Authentication.                    |
| **JWE** | JSON Web Encryption - an IETF standard (RFC 7516) used to encrypt the RBA device data payload before transmission.                                      |
| **RASP** | Runtime Application Self-Protection - SDK logic that detects hostile device states (root, emulator, debugger) and includes them as security flags.      |
| **RBA** | Risk-Based Authentication - the process of silently authenticating a payment transaction based on device risk signals without requiring a 3DS2 Challenge. |
| **3DS Server** | The component that routes the authentication request between the 3D Secure 2 SDK and the issuer's ACS.                                 |
| **Data Safety** | Google Play Console section where developers disclose data collection and handling practices for their apps.                                            |

---
*Disclaimer: This document is provided for guidance only. Google Play policies and EMVCo specifications are subject to change. Always verify current requirements against official Google Play policy documentation and your legal counsel before submitting your Data Safety form.*
