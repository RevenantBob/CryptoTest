# CryptoTest

This project demonstrates a major bug in the NCrypt property `NCRYPT_PIN_PROPERTY` when using `MS_PLATFORM_CRYPTO_PROVIDER`.

The sample will allow a user to set a PIN then attempt to programmatically set the PIN before sign operation. Despite being set the UI will still prompt for a PIN showing a PIN mismatch.

This is an error due to `MS_PLATFORM_CRYPTO_PROVIDER` requiring NULL terminated PIN unlike `MS_KEY_STORAGE_PROVIDER`.

## The Bug

This code is slightly modified from the Strong Key Protection sample from microsoft found at: https://github.com/microsoft/Windows-classic-samples/blob/main/Samples/Security/StrongKeyProtection/cpp/StrongKeyProtection.cpp

Within the sample from Microsoft, `MS_KEY_STORAGE_PROVIDER` is used to create a key using a `UIPolicy`. `NCRYPT_PIN_PROPERTY` is then set to a PIN so that the user is not prompted for a PIN when using the key for the signing operation.

This works fine with `MS_KEY_STORAGE_PROVIDER`.

When implemented with `MS_PLATFORM_CRYPTO_PROVIDER` to make use of TPM, the PIN must be NULL terminated when setting NCRYPT_PIN_PROPERTY value, which is NOT the case using `MS_KEY_STORAGE_PROVIDER`. Second the NULL terminated PIN never matches the PIN provided in the UI.

## Summary
It is *impossible* to programmatically set a PIN when using `MS_PLATFORM_CRYPTO_PROVIDER` if the UI is used in the flow.

### Case 1: UIPolicy used to create the PIN.

If the UIPolicy is used to create the PIN, setting the PIN before Sign operation requires an added NULL terminator or it will be considered an invalid PIN. This will not match the PIN used in UIPolicy.

### Case 2: Programmatically set PIN during creation.

If the key is created with a PIN set programmatically using `NCRYPT_PIN_PROPERTY` but NOT before sign operation, the UI prompt before Sign will appear and the user cannot type a PIN to match the programmatically assigned PIN.

### Expected Behavior

The `MS_PLATFORM_CRYPTO_PROVIDER` should behave the same as `MS_KEY_STORAGE_PROVIDER` allowing `NCRYPT_PIN_PROPERTY` without a NULL terminator.

However, this behavior does not match documentation for `NCRYPT_PIN_PROPERTY` which specifies it should be NULL terminated despite the sample not requiring it. This mismatch in documentation and implementation may be the cause of the bug.

`NCRYPT_PIN_PROPERTY` documentation: https://learn.microsoft.com/en-us/windows/win32/seccng/key-storage-property-identifiers#ncrypt_pin_property