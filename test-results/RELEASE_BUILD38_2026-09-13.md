# Mobile build 38 — 13 September 2026

## Scope and source

The owner requested an IPA and APK for internal testing after the backend rollout and production-connected Android checks. An AAB is included for the Google Play internal-testing track. Store uploads and tester distribution remain with the owner.

- Version: **1.0.4+38**; package/bundle `app.iadme.mobile`.
- Source: mobile `0b41ad7` (build-number increment), including functional changes from `8fa2f02`.
- `APP_ENV=prod`, `APP_RELEASE=iadme-mobile@1.0.4+38`; API `https://api.iadme.app`. No local API override.
- Android uses Google Play billing and native unit `ca-app-pub-2924641977385769/3473122948`.
- iOS uses Apple StoreKit and native unit `ca-app-pub-2924641977385769/9941107182`.
- Both are release builds with an ad opportunity after every two videos. Live-unit configuration differs from the earlier debug device-test APK, which used sample units. Ad supply, variety and fill remain controlled by the ad network.
- OAuth configuration is retained from the successful production-connected device check. Dependency lockfiles and existing native signing configuration are retained. No Sentry configuration, mediation setting or backend deployment is changed by this packaging operation.
- The backend these builds target already runs `release-2026.09.13-37edc10`; see [the rollout and Android report](RELEASE_BACKEND_2026-09-13.md).

## Changes included

- Google/phone authentication recovery and progress handling, including prevention of repeated social sign-in taps.
- Three upcoming native-ad placements preloaded with bounded retention, fresh refill and retries, shared across Feed and Trending.
- Pagination, feed-filter semantics and the redesigned location picker.
- Upload processing-status polling and discovery refresh once a video is ready.
- Right-side age/mute alignment and improved native-ad content height.
- Earlier build 37 upload-screen simplification and defensive profile parsing remain included.

## Validation

- Six production-configured social-login and store-ad preflight tests passed for this packaging run.
- Both Android/iOS ad-selection regressions passed after packaging, including rejection of missing/malformed/sample production units and selection of the supplied live unit in production release mode: **eight focused checks passed in this run**.
- Prior functional verification: 216 mobile unit/widget cases, focused follow-up layout checks, emulator/simulator native-ad checks and the connected Android passes recorded in the linked reports. Those are not represented as new release-artifact runtime tests.
- Android AAB validated with Google's [bundletool 1.18.3](https://github.com/google/bundletool/releases/tag/1.18.3); its download matched the checksum published on that release.
- AAB/APK manifests contain version 1.0.4 / build 38, minimum Android API 24, target API 36. Both include `arm64-v8a`, `armeabi-v7a` and `x86_64` application code. Debugging and cleartext traffic are disabled.
- APK signature and AAB JAR signature verified against the existing upload certificate: `B9:D1:45:E2:9F:18:B7:EA:3D:19:21:80:E8:01:B6:C4:5F:62:7F:B5:08:43:CD:70:B4:6C:E4:CB:62:A9:61:2F`.
- Production API/release/OAuth/ad identifiers were found in each Android architecture's compiled code. The staging API was absent. AOT can retain unused sample-unit literals; their presence does not establish which unit is requested. Selection is checked through release mode, production definitions and the existing ad-selection regression.

## Android artifacts

Paths below are relative to the mobile repository.

| Artifact | Path | Bytes | SHA-256 |
| --- | --- | --- | --- |
| AAB | `apps/iadme_app/build/app/outputs/bundle/release/iadme-1.0.4-build38-prod.aab` | 73,250,990 | `d3442853813f2d5d3be35b76e6867a65ed39d96f390ee38059bfdd446e28e4e6` |
| APK | `apps/iadme_app/build/app/outputs/flutter-apk/iadme-1.0.4-build38-prod.apk` | 72,416,215 | `bb4939fa7bc3729b73c0955040037823ae92107603706b9f31d79710573850d6` |

## iOS artifact

- IPA: `apps/iadme_app/build/ios/ipa/iadme-1.0.4-build38-prod.ipa`, relative to the mobile repository.
- Size: **32,841,578 bytes**.
- SHA-256: `0ddd2c87e02f2e1cbafc9d22d976f44ed86d7bde63ba178beefbeaad9a46cde7`.
- Exported with `app-store-connect`, automatic signing and automatic build renumbering disabled. The archive and exported bundle report **1.0.4 / 38**, bundle `app.iadme.mobile`, minimum iOS **15.0**.
- `codesign --verify --deep --strict` passed. Identity: **Apple Distribution: FORESTPOND TECHNOLOGIES LLP (KU58Q677M3)**.
- Provisioning matches the bundle/team, expires **2027-06-09 15:47:27 UTC**, enables production push and disables debugging. It has no device-list or enterprise distribution flags.
- App.framework UUID `0C7143A3-879F-1B89-485E-50E602632098` matches the final archive. The compiled framework contains the production API/release/OAuth/native-unit identifiers; the staging API is absent.

All eight prior named artifacts were rehashed after packaging and match their original hashes, including builds 36/37 and the earlier device-test APKs. These new files have not been installed on the device, uploaded to either store or distributed to testers by this operation.

## Internal-testing focus

Use the AAB for the Google Play internal track and the IPA for TestFlight; the APK is for direct installation. Check Google and phone login, profile loading, app restart persistence, Feed/Trending traversal beyond the first page, backward/rapid scrolling, live-ad readiness and variety, geographic filters, and a fresh video upload through processing to discovery. Test on an affected older Android as well as a newer Android and iPhone.

Earlier debug passes verified the two-video ad cadence with sample inventory. Some automated debug swipes did not move a page, and one brief Feed index correction was recorded. Release-mode scrolling and actual inventory are acceptance checks for this build; new production upload publication also remains to be exercised manually.

Private build and verification evidence is stored in `/private/tmp/iadme-build38-20260913/` and is not committed.
