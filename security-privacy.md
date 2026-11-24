# HashVektor for Android: security and privacy

The bar is **optimal security**. HashVektor is an integrity tool, so it has to be trustworthy
itself. It is offline, collects nothing and asks for no permissions. This file maps Google's two
checklists to what the app does, item by item:

- [Security tips](https://developer.android.com/privacy-and-security/security-tips)
- [Privacy and security overview](https://developer.android.com/privacy-and-security/about)

**Status:** ✅ done · 🔜 planned (with the feature that needs it) · ➖ not applicable (and why).
Last reviewed 2026-11-20 (Play Billing added), against the Release build's merged manifest
(`obj/Release/net11.0-android/android/AndroidManifest.xml`).

## What the app is, security-wise

- **No network.** There's no `INTERNET` permission, so the app can't send anything anywhere.
  Play Billing works through the Play Store app (IPC). The Billing Library adds `INTERNET` and
  `ACCESS_NETWORK_STATE` for its own diagnostic logging to Google (DataTransport); both are
  removed in `AndroidManifest.xml` (decided 2026-11-20), so those logs can't leave the phone.
- **No runtime permissions.** The final manifest has two permissions, neither shown to the user:
  `com.android.vending.BILLING` (install-time, Play Billing) and AndroidX's internal
  `DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION` (signature level, used inside the app only).
- **Folders only for the session.** Verify can read a folder the user picks
  (ACTION_OPEN_DOCUMENT_TREE) to find a list's files; no persistable grant is taken, so the
  access ends with the app. Remembered folders ("Places") will be an explicit choice.
- **Checksum lists are untrusted input.** They're parsed by Core's tested parser, capped at
  64 MB, and names are only looked up inside what the user chose: `..` can't climb above the
  chosen folder, and full paths from another computer are matched by file name only.
- **Installed apps: launchable ones only, never sent anywhere.** App signatures (Tools) lists
  apps through a manifest `<queries>` entry for the launcher intent, not the Play-restricted
  QUERY_ALL_PACKAGES. Not a permission; the list and the certificates are read on the phone
  only. An APK is read by Android's own parser through its file descriptor; when a provider
  can't give one, a private cache copy is made and deleted at once.
- **Purchases are checked on the phone.** Each Pro purchase's Google Play signature is verified
  against the app's Play public key (`Licensing/PurchaseSignature.cs`), so a fake "Play Store"
  can't unlock Pro. Play is asked again on every return to the app, so refunds remove Pro.
- **Files only as the user grants them.** The Storage Access Framework hands the app one file
  or one folder at a time; there's no broad storage permission.
- **No accounts, no identifiers, no analytics, no ad networks, no third-party SDKs** that collect
  data. Free users see a *house ad*, HashVektor's own built-in banner for Pro: no SDK, no
  network, nothing collected (decided 2026-11-19, plan §3). Adding an ad network would change
  this whole file: internet access, the `AD_ID` permission, a consent flow, and Data safety
  disclosures.

## Security tips

| Area | Item | Status |
|---|---|---|
| Authentication | Credential Manager, passkeys, biometrics, autofill | ➖ No accounts or sign-in. |
| App integrity | Play Integrity API | ➖ There's no server, and Integrity verdicts need a backend to be meaningful. Purchases are verified on the device instead: Google Play's RSA signature on each purchase (`Licensing/PurchaseSignature.cs`). |
| Internal storage | Private by default; no `MODE_WORLD_*` | ✅ Nothing is written yet. The index, settings and keys will live in the app's private storage. |
| External storage | Only non-sensitive data; validate what's read; no code | ✅ The app never writes to shared storage. It only reads the files the user picks. What it reads is hashed, never run, and checksum files are parsed by Core's tested parsers. |
| Content providers | `android:exported="false"` unless shared | ✅ MAUI's FileProvider, the Mono runtime provider and androidx-startup are all `exported="false"`. |
| | Never accept the app's own provider back | ✅ Incoming URIs from the app's own authorities are refused (`Documents.IsAcceptableIncoming`). |
| | Parameterised queries | ✅ Nothing yet. 🔜 The SQLite index (Core's `FileIndex`) already uses parameters. |
| Permissions | Request the minimum | ✅ None. MAUI's `USE_BIOMETRIC` / `USE_FINGERPRINT` are removed with `tools:node="remove"`. |
| | Avoid device identifiers | ✅ None are read. |
| | Custom permissions for IPC | ➖ No IPC is offered to other apps beyond Share / Open with. |
| Networking | HTTPS only; no localhost sockets; distrust HTTP | ✅ No network at all. In case one is added: `usesCleartextTraffic="false"` and `network_security_config.xml`, which refuses cleartext and trusts only system CAs (no user-installed CAs). |
| Telephony | No SMS | ➖ |
| Input validation | Validate all untrusted input | ✅ Shares and Open with: only `content://`, at most 1000 files, text cut at 1 M characters (`MainActivity.Receive`). Clipboard: only hash-shaped text is used (Core's `LooksLikeHash`, at most 300 characters). Checksum and manifest files go through Core's parsers. |
| | Native code: buffers, pointers | ✅ The one native library is the official BLAKE3 C code (unmodified 1.8.7), with a five-function wrapper that only passes a pointer and a length from .NET spans. Hardened (see Native code). |
| User data | Minimise; nothing to third parties; work on the device | ✅ Everything happens on the phone and nothing leaves it. |
| | No PII in logs | ✅ The app's code doesn't log. `AddDebug` logging exists in Debug builds only. |
| WebView | JavaScript, `addJavascriptInterface` | ➖ No WebView. |
| Credentials, API keys | Keystore, no keys in source | ✅ No API keys. The Play upload key stays out of the repo (`.gitignore`: `*.keystore`, `*.jks`). Use Play App Signing. |
| Cryptography | No home-made crypto; AES-256; `KeyStore` for long-term keys | ✅ Hashes come from Core (vetted libraries, tested against published vectors) and the official BLAKE3. 🔜 Manifest signing (plan §4): Ed25519 via BouncyCastle, with the private key wrapped by a non-exportable AES-256-GCM key in the **Android Keystore**. |
| IPC / intents | Explicit intents; set `exported`; filters aren't security | ✅ Two exported activities. `ShareReceiverActivity` takes Share and Open with: it never displays anything, validates the URIs and text, and forwards them to `MainActivity` with an **explicit** intent, passing on the read grant through `ClipData` + `FLAG_GRANT_READ_URI_PERMISSION`. `MainActivity` (the launcher, `singleTask`) has no data filters, and validates the forwarded intent again, because any app can address an exported activity. The app sends no implicit intents with data. |
| | Services and receivers not exported | ✅ MAUI's `KeepAliveService` has no filter (not exported). The Essentials receivers are `exported="false"`. AndroidX's `ProfileInstallReceiver` is exported but needs `android.permission.DUMP` (system/adb only). |
| Dynamic code | Don't load code from outside the APK | ✅ None. |
| Native code | Prefer the SDK; follow Linux practice | ✅ Native code only where it matters (BLAKE3: 5.6× the managed speed on the test phone). `tools/Build-NativeBlake3.ps1` builds with `-fstack-protector-strong`, `-D_FORTIFY_SOURCE=2`, `-fstack-clash-protection`, full RELRO (`-z relro -z now`) and a non-executable stack, **and checks each one** in the output. Native results are checked against Core's managed BLAKE3 at first use, with fallback on any difference. |
| Dependencies | Keep up to date; watch for vulnerabilities | ✅ `dotnet list package --vulnerable --include-transitive`: none in the app or in Core (2026-11-19). MAUI pinned to 11.0.10, QRCoder 1.9.0. Re-run before each release. |

## Privacy checklist

| Checklist | Item | Status |
|---|---|---|
| Minimise permissions | Minimum, only when needed, degrade gracefully, review SDKs' permissions | ✅ No permissions. The SDK-added biometric permissions were found and removed. 🔜 QR scanning (step 5) will ask for the camera only when the user taps Scan, and works without it (paste instead). |
| Location | | ➖ Never used. |
| Handle data safely | Audit data access | 🔜 Run Android's data access auditing (`AppOpsManager.OnOpNotedCallback`) once the camera and clipboard features are all in. |
| | Package visibility | ✅ No queries. 🔜 The app signature checker (step 5) will use a `<queries>` launcher intent, **not** `QUERY_ALL_PACKAGES` (a restricted Play permission), and accept picked APK files. |
| | Scoped storage | ✅ The Storage Access Framework only. No `MANAGE_EXTERNAL_STORAGE` (plan §4). |
| | Resettable identifiers | ✅ None used. |
| | Prominent disclosure | ✅ Nothing is collected. The clipboard is read only right after hashing, only while the app is in front, and Android 12+ shows its own "pasted" notice. 🔜 A Settings switch to turn the clipboard comparison off. |
| | Data safety form | ✅ See the answers below. |
| | Sensitive data only through explicit intents, with one-time grants | ✅ Nothing is sent. 🔜 Exported reports (step 5) will be shared through the FileProvider with a one-time `FLAG_GRANT_READ_URI_PERMISSION`. |
| | No sensitive data in Logcat | ✅ See "No PII in logs". |
| Identifiers | No IMEI or serial; ad ID only for ads | ✅ None. The advertising ID is never read, and `com.google.android.gms.permission.AD_ID` is removed with `tools:node="remove"`, so no library can add it. Play Console → "Does your app use advertising ID?": **No**. |
| User-facing privacy | Privacy dashboard, camera and microphone indicators | ✅ Android shows them by itself. 🔜 The camera only while scanning, in the foreground. |
| | App hibernation | ➖ Scheduled jobs (step 5) use WorkManager, which Android exempts from hibernation's restrictions, and the app keeps no long-lived permissions to lose. |

## Backups and data transfer

`android:allowBackup="false"`, and `dataExtractionRules` (Android 12+) and `fullBackupContent`
(Android 8–11) exclude every domain. Nothing goes to Google's cloud backup or to a new phone.
The index will hold file names and hashes, and the signing key couldn't be restored on another
device anyway (it's Keystore-bound). Restoring needs no backup: settings are few, and hashes
can be recomputed.

## Google Play Data safety form (answers)

- **Does the app collect or share any of the required user data types?** No.
- **Is all user data encrypted in transit?** Not applicable: no data is transmitted.
- **Can users request that data be deleted?** Not applicable: nothing is collected. On-device
  data goes when the app is uninstalled, or with "Clear storage".
- Play Billing is Google's own service, covered by Google's privacy policy, and not counted as
  the app collecting data.

## Before each release

1. `dotnet list src/HashVektor.Mobile package --vulnerable --include-transitive`, and the same for Core.
2. Read the merged Release manifest: permissions, `exported` components, backup and network attributes.
3. If BLAKE3 changed: `tools\Build-NativeBlake3.ps1` (it checks alignment and hardening).
4. Check for logging: `grep` for `Log.`, `Console.Write`, `Debug.Write` in `src/`.
5. Update this file's tables for any new feature (camera, network, files written, intents sent).
6. Read the published policies against the app:
   [privacy](https://www.planetarchives.org/hashvektor/android/privacy/index.html) and
   [terms](https://www.planetarchives.org/hashvektor/android/terms/index.html). Every claim must
   still be true for the build being released. Update the page *before* the release, with a new
   effective date. As of 2026-11-20 they say: files are written or deleted only when the user asks
   (checksum files, chosen duplicates), and only in folders the user granted; the files being
   hashed are never changed; QR scanning is "coming in a later version" and needs the camera only
   when used. When QR scanning ships, drop "coming in a later version".
