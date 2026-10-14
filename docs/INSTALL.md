# Building and Installing

This document explains how to set up your environment, build **ASComplete**, and configure build
environments and release signing.

---

## Prerequisites

Before building the project, ensure you have the following installed:

- **JDK 17** (required by Gradle and standard Kotlin Compose builds)
- **Android Studio** (Ladybug or newer recommended)
- **Android SDK** with target API 37 (Android 15)
- An Android device (Android 7.0 / API 25+) or emulator

---

## Environment Differences (Debug vs. Release)

Pay attention to which build variant you are using, as they behave differently regarding Application
ID and backend servers:

| Feature                 | Debug Build (`assembleDebug`)                                | Release Build (`assembleRelease`)                       |
|:------------------------|:-------------------------------------------------------------|:--------------------------------------------------------|
| **Application ID**      | `org.codeberg.patkek.ascomplete.debug`                       | `org.codeberg.patkek.ascomplete`                        |
| **Version Name**        | `{version}-debug`, e.g. `0.3.0-debug`                        | `{version}`, e.g. `0.3.0`                               |
| **OSM Server Endpoint** | **Dev Server** (`https://master.apis.dev.openstreetmap.org`) | **Production Server** (`https://api.openstreetmap.org`) |

> [!NOTE]
> **Note on Debug Accounts:**  
> Because debug builds connect to the OpenStreetMap **dev instance**, your standard OpenStreetMap
> account will not work. You must create a separate development account
> at [master.apis.dev.openstreetmap.org](https://master.apis.dev.openstreetmap.org) to log in and
> test
> features in debug mode.

---

## Building the App

1. **Clone the repository:**

```bash
   git clone https://codeberg.org/Patkek/ASComplete.git
   cd ASComplete
```

2. **Build Debug APK:**

```bash
./gradlew assembleDebug
```

The generated debug APK will be located at:

```text
app/build/outputs/apk/debug/app-debug.apk
```

3. **Build Release APKs:**

```bash
./gradlew assembleRelease
```

*Note: If no keystore configuration is provided, Gradle will complete the build with unsigned or
default-signed release binaries.*
The output APKs (Universal and per-ABI splits) can be found at:

```text
app/build/outputs/apk/release/
```

---

## Release Signing Configuration (Optional)

To sign release builds with your own private key, create a `keystore.properties` file in the **root
directory** of the project:

```properties
storeFile=path/to/your/release-key.jks
storePassword=your_keystore_password
keyAlias=your_key_alias
keyPassword=your_key_password
```

> [!WARNING]
> **Security Note:** `keystore.properties` and your `.jks` files are ignored by git. Never commit
> these files to the repository.

---

## Advanced: ABI Splits and Versioning

The project uses ABI splitting to optimize APK download size while generating a Universal APK for
general distribution:

- **Supported Architectures:** `arm64-v8a`, `armeabi-v7a`, `x86`, `x86_64`
- **Version Code Calculation:** Version codes automatically append an ABI modifier (
  `10 * versionCode + abiModifier`) to avoid update collisions on devices across different
  architectures.
- **F-Droid Build Override:** To target a single ABI specifically (e.g. for F-Droid packaging), pass
  the `buildAbi` project property:

```bash
./gradlew assembleRelease -PbuildAbi=arm64-v8a
```