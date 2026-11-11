# Patched TEESimulator-RS engine

`TEESimulator-RS-v6.0.1-367-Release-NoEnhanced-patched.zip` is the
`MashiroSann/TEESimulator-RS` v6.0.1-367 Release-NoEnhanced engine with one
change in `classes.dex`:

`org.matrix.TEESimulator.interception.keystore.shim.KeyMintSecurityLevelInterceptor.trackAndEnforceOpLimit()`
returns `null` immediately. This disables the synthetic per-UID StrongBox
limit of 4 concurrent/recent operations.

## Why

The limiter is not a hardware limit; the real KeyMint HAL already governs its
operations. Applications that open more than four StrongBox operations during
startup — notably React Native apps using `react-native-keychain` AES-GCM keys,
such as Цифра Банк — were failing with:

```text
android.security.KeyStoreException: -29
Error::Km(TOO_MANY_OPERATIONS)
```

and remained stuck on their splash screen. With the limiter disabled the
keychain calls complete and the app proceeds to its normal UI.

## Provenance

- Base: `TEESimulator-RS-v6.0.1-367-Release-NoEnhanced.zip` from
  `MashiroSann/TEESimulator-RS`.
- Patch: `classes.dex` disassembled with baksmali 2.5.2, the method body
  prefixed with `const/4 v0, 0x0` / `return-object v0`, reassembled with
  smali 2.5.2, and the zip entry replaced.
- Everything else in the engine zip is unchanged.

`build.sh` uses this file as the default `--tee-file` when it exists, so the
CI build packages the patched engine into all AlwaysStrong variants.
