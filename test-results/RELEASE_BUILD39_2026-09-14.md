# Mobile build 39 — 14 September 2026

## Scope and source

This release packages the mobile work completed after build 38 and retains all fixes introduced in builds 36–38. The owner requested production Android AAB and iOS IPA artifacts plus consolidated release notes. No store upload or tester distribution was performed.

- Version: **1.0.4+39**; package/bundle `app.iadme.mobile`.
- Mobile source: `9f6e54f` (build-number commit), including functional source `697fac5`.
- Production API: `https://api.iadme.app`; release identity `iadme-mobile@1.0.4+39`.
- Android uses Google Play billing and native ad unit `ca-app-pub-2924641977385769/3473122948`.
- iOS uses StoreKit and native ad unit `ca-app-pub-2924641977385769/9941107182`.
- Feed and Trending request an ad opportunity after every two content videos. Actual fill and creative variety remain dependent on AdMob inventory, connectivity, consent and device eligibility.

## Changes by build

### Build 36 foundation

- Restored signed-in sessions and extended successful login persistence to 90 days through the matching backend session support.
- Hardened Google/Apple sign-in continuation and first social registration so the verified identity can be reused without an unintended second sign-in.
- Added registration device and permission-dependent approximate location details.
- Added biometric login support where the platform/device has enrolled biometrics.
- Improved video-player lifecycle, scrolling and ad-aware reel behavior.
- Added the two-content-video native-ad opportunity configuration.

### Build 37 upload and profile work

- Refreshed the upload composer with a combined preview/caption card, compact **Show location as** selector and theme-aware presentation.
- Combined the duration and location-privacy guidance: videos must be under four minutes; only the selected area is shown and the exact location stays private.
- Added the wallet balance, **Add Stars**, and **Upload · 5★** actions in a compact responsive layout.
- Preserved caption focus through keyboard resizing and improved large-text/small-screen behavior.
- Prevented phone-registered profiles from failing when email is absent or null.

### Build 38 authentication, discovery and ad preparation

- Added Google and phone authentication progress/recovery handling and blocked repeated social-login taps while a request is active.
- Prepared three upcoming native-ad placements with bounded retention, refill and retry behavior shared by Feed and Trending.
- Corrected Feed/Trending pagination and filtering behavior and redesigned the location filter picker.
- Added upload-processing polling and discovery refresh when a new upload becomes ready.
- Corrected the right-side age/mute alignment and improved native-ad content sizing.

### Build 39 advertising and network reliability

- Replaced the small native-ad card with a viewport-filling, swipeable ad presentation in Feed and Trending.
- Bound native media correctly on Android and iOS, allowing SDK-owned muted video autoplay while retaining static-image creatives and their aspect ratio.
- Added adaptive portrait/landscape layouts with visible advertiser attribution, AdChoices and call-to-action controls.
- Strengthened the ad pool: current plus three upcoming placements, bounded native-object/concurrent-request limits, per-slot retry/backoff and separation of fresh inventory from already shown ads.
- Added recovery when the device changes between Wi-Fi and cellular by refreshing stale native/Dart network paths without replaying an active authentication write.
- Added bounded network and AdMob diagnostics, including request/route context, SDK response details, media observations and a native health probe.
- Added an encrypted, retryable mobile diagnostic queue so captured login, playback, network and advertising inconsistencies can reach the deployed backend incident pipeline after connectivity returns.

## Production artifacts

### Android AAB

- Path: `iadme-mobile/apps/iadme_app/build/app/outputs/bundle/release/iadme-1.0.4-build39-prod.aab`.
- Size: **73,423,658 bytes**.
- SHA-256: `f963dc8867d6148ddd14858f4c898d9d6ead9010e7800dc7cd1778f5319d1bc7`.
- Manifest: package `app.iadme.mobile`, version `1.0.4`, build `39`, minimum API 24, target API 36.
- Architectures: `arm64-v8a`, `armeabi-v7a`, `x86_64`.
- Bundletool validation and JAR signature verification passed. Upload-certificate SHA-256 matches earlier releases: `B9:D1:45:E2:9F:18:B7:EA:3D:19:21:80:E8:01:B6:C4:5F:62:7F:B5:08:43:CD:70:B4:6C:E4:CB:62:A9:61:2F`.

### iOS IPA

- Path: `iadme-mobile/apps/iadme_app/build/ios/ipa/iadme-1.0.4-build39-prod.ipa`.
- Size: **32,911,684 bytes**.
- SHA-256: `04f6ea3f030f488de57b86d51f45db1656c8d0606a0667fe5840ee1df968c800`.
- Bundle: `app.iadme.mobile`, version `1.0.4`, build `39`, minimum iOS 15.0.
- `codesign --verify --deep --strict` passed using macOS trust services.
- Distribution: App Store Connect, team `KU58Q677M3`; the profile disables debugging, enables production push, contains no development-device/enterprise list and expires `2027-06-09T15:47:27 UTC`.
- App.framework UUID `0C7143A3-BFC1-3A60-485E-50E681BE2A24` matches the exported archive.

## Verification

- Production social-login and AdMob release preflight passed. It validates the supplied real unit IDs, rejects missing/malformed/sample production IDs, and verifies repeated social-login taps are blocked.
- The AAB and IPA contain the production API, release, OAuth and platform ad identifiers; the staging API is absent from compiled application code.
- Android release debugging and cleartext traffic are disabled.
- Build 39 functional source previously passed the full Flutter suite: **231 tests passed with two existing skips**. Official Android and iOS native test inventory exercised video autoplay, static media, compact layouts, forward/backward swipes, tab visibility and prepared-ad reuse.

Local private build evidence is retained under `/private/tmp/iadme-build39-20260914/`. Store processing, physical-device behavior and live AdMob fill/variety remain internal-testing acceptance checks. These artifacts have not been uploaded or distributed.

## Official store release notes

What’s new in iAdMe 1.0.4 (Build 39):

- Refreshed the upload experience with clearer caption, duration and location-privacy guidance.
- Improved Google and phone sign-in reliability and kept successful sessions signed in.
- Improved Feed and Trending pagination, filters, video discovery and upload-ready refresh.
- Added full-screen, swipeable native ads with muted video playback and better recovery when ads are temporarily unavailable.
- Improved recovery when switching between Wi-Fi and mobile data.
- Fixed phone-account profile loading and polished video age/mute alignment.
- Added better diagnostics to help resolve login, playback, network and advertising issues faster.
