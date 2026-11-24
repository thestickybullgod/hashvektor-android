# HashVektor for Android

HashVektor is an Android hashing and integrity app built with .NET MAUI. It uses the shared
HashVektor Core engine and is designed to keep files, hashes, and integrity reports on the
phone.

## Features

### Available now

- Hash files and text with the supported algorithms
- Free hashing with SHA-256, SHA-1, MD5, and CRC32
- HashVektor Pro access to additional algorithms, including native BLAKE3
- Compare hashes and inspect character-level differences
- Display hashes as QR codes
- Verify and create checksum lists
- Cross-check checksum lists
- Check HMAC-SHA256 and HMAC-SHA512 lists
- Open shared files or receive `Open with` requests
- Save finished results as checksum files
- Check Android app and APK signing certificates
- Google Play Billing with restore and refund handling
- Light, dark, and Material You-compatible themes

### Planned

- Folder-scale operations and queued background jobs
- Library indexing and change history
- Duplicates and storage cleanup
- Scheduled integrity checks
- QR code scanning
- APK signature checking enhancements
- Forensic collection and signed manifests
- Additional reports and exports

## Privacy

HashVektor is offline and does not collect or transmit files or hashes. It requests no runtime
storage or camera permissions; files are read only through Android's Storage Access Framework
when the user grants access.

Google Play Billing is the only purchase mechanism. Purchase handling is device-side, and app
backup is disabled.

See [Security and privacy](docs/security-privacy.md) for the complete security checklist,
permission decisions, and Play Data safety answers.

## Technology

- .NET 11 MAUI for Android
- Shared `HashVektor.Core` engine
- CommunityToolkit.Mvvm
- Android Storage Access Framework
- Google Play Billing Library
- Official BLAKE3 C implementation with native Android builds

Android support:

- Minimum SDK: 26 (Android 8.0)
- Target SDK: 37
- CPU architectures: `arm64-v8a` and `x86_64`
- Release output: Android App Bundle (`.aab`)

## Repository layout

```text
src/HashVektor.Mobile/       MAUI Android application
native/blake3/               Native BLAKE3 sources and wrapper
tools/                       Native build and Play release scripts
docs/                       Security, listing, and platform documentation
artifacts/                   Built release files
```

## Prerequisites

The Windows build environment requires:

- .NET SDK 11
- The MAUI Android workload
- Android SDK
- Android NDK
- JDK 17
- A local checkout of `HashVektor.Core`

HashVektor Core is currently referenced from the sibling Windows 2.0 repository:

```text
D:\projects\hashvektor v2\src\HashVektor.Core\HashVektor.Core.csproj
```

Install the workload if needed:

```powershell
dotnet workload install maui-android
```

## Build

The native BLAKE3 libraries are already committed. A normal Debug or Release build does not
require rebuilding them.

```powershell
dotnet build HashVektor.Android.slnx
```

Rebuild native BLAKE3 after changing `native/blake3/` or its upstream implementation:

```powershell
tools\Build-NativeBlake3.ps1
```

The script builds hardened ARM64 and x86_64 libraries with 16 KB page alignment.

## Release

Create the one-time Google Play upload key:

```powershell
tools\New-UploadKey.ps1
```

Build and validate a signed App Bundle:

```powershell
tools\Build-Release.ps1
```

Use `-BumpVersionCode` when the Play version code must advance. Use `-AllowDirty` only for an
explicit test build. Release builds require no vulnerable packages and no uncommitted changes
unless that test-only flag is supplied.

The release script writes the signed bundle and SHA-256 checksum to
`artifacts\release\`.

## Documentation

- [Android implementation plan](android-plan.md)
- [Security and privacy](docs/security-privacy.md)
- [Google Play listing](docs/play/listing/README.md)
- [Native BLAKE3](native/blake3/README.md)

## License

Add the repository license here once the distribution licence is finalized.
