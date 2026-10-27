# DiAuto-MG4

Wireless / USB **Android Auto** for the MG4 Android head unit (SWI69), with live
battery SoC forwarded into Google Maps via Android Auto `VehicleEnergyModel`
(sensor types 23/25).

> Fork of [shihabal3amri/DiAuto](https://github.com/shihabal3amri/DiAuto) (AGPL-3.0).
> MG4 telemetry: vendored [EVHardware](https://github.com/fatihdonmezdev/MG4ABRP) in `evhardware/`.
> For Apple CarPlay on the same car, use **DiPlay-MG4** separately — do not run both at once.

## Features (MG4)

- Native wireless Android Auto over the car’s built-in hotspot (Android 9; Wi‑Fi Direct unavailable).
- `car` flavor: platform-signed `android.uid.system` build for MG4 CarProperty / BMS access.
- `phone` flavor: ordinary phone/tablet build for desk testing; it has no system UID.
- Optional **Battery for Google Maps** (default on): SoC + range → AA sensors 23/25.
- **Demo battery values**: manual SoC / range / capacity (skips EVHardware; desk testing).
- Application id: `com.drivehub.diauto.mg4` (`phone` flavor: `com.drivehub.diauto.mg4.phone`).

The phone flavor is not a substitute for the platform-signed car build: it is not
expected to read MG4 SoC/BMS data through CarProperty or force-stop OEM Android Auto
apps. Phone-flavor behavior on an MG4 has not been verified on a vehicle.

## Installing APKs on the MG4

If you haven't sideloaded apps before:

1. Open **Bluetooth Settings** and select the car name.
2. Hold down on the **comma** until the settings appear.
3. Select **Android Keyboard Settings**.
4. Select **Languages**.
5. Use the search box at the top to search for **file**.
6. That should give you access to the **Files** app.
7. From there, select a connected USB device with APKs on it and install as usual.

## Build

**MG4 car** (platform-signed, `android.uid.system`):

```powershell
$env:MG4_PLATFORM_KEYS_DIR = "C:\path\to\platform-keys"  # platform.pk8 + platform.x509.pem
powershell -File scripts\build_mg4.ps1
adb install -r ..\DiAuto-MG4-v1.9.8.apk
```

**Normal phone / tablet** (debug keystore, Demo battery):

```powershell
powershell -File scripts\build_phone.ps1
adb install -r ..\DiAuto-MG4-phone-debug.apk
```

In Android Studio: open `DiAuto-MG4`, pick **phoneDebug** or **carDebug**.

See [docs/MG4_TESTING.md](docs/MG4_TESTING.md) for the vehicle checklist.

## Release / MVP

With JDK 17+, Android SDK configured, and the release keystore available as
`headunit-release-key.jks` in the repository root (or its parent), configure signing
and build the requested variant. Gradle reads `key.properties` first, then
`secrets.properties`; otherwise it uses these PowerShell environment variables:

```powershell
$env:HEADUNIT_KEYSTORE_PASSWORD = "your-keystore-password"
$env:HEADUNIT_KEY_PASSWORD = "your-key-password"
.\gradlew.bat :app:assemblePhoneRelease
.\gradlew.bat :app:assembleCarRelease
```

The release signing config uses the `headunit-revived` alias unless overridden by
`key.properties`. `phoneRelease` is the normal, no-system-UID MVP build. `carRelease`
is for local use and must be signed with the MG4 platform certificate to retain the
system UID; a regular app-signing key does not grant that privilege.

Telegram crash-report upload is optional and off by default in release builds. It is
available only when both `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` are supplied at
build time; leave them unset to keep upload unavailable.

### Google Maps SoC note

Battery % / arrival charge is shown primarily on **Android Auto Maps** (the projected
head-unit UI) once a route is active. The phone’s own Maps screen often has no permanent
SoC badge; that is expected Google behavior when VEM is delivered over AA.

## Upstream

See upstream DiAuto README / docs for general AA host features. MG4-specific changes
live in this fork.
