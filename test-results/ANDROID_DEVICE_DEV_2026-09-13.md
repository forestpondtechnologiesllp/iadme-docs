# Connected Android development testing — 13 September 2026

## Setup and scope

Physical OnePlus 9 Pro (LE2121), Android 14, connected over USB. Latest local mobile changes were built as a debug APK and installed in place. This is a development build reporting version 1.0.4+37, not a replacement for the named production build 37 AAB/IPA. No store submission or staging/production deployment occurred.

The app uses `http://127.0.0.1:3000` through `adb reverse tcp:3000 tcp:3000`. The Mac runs the latest API source, local PostgreSQL on port 5433 and Redis on port 6379. Only Feed, Trending and feed-ad hydration workers run. Registration email and issue creation are disabled for this session; user-authorized phone OTP delivery uses real MSG91. Google uses the configured development OAuth client; ads use official Google sample inventory.

The existing local database was backed up before applying three already-existing additive migrations: `20260907_add_video_location_display_level.sql`, `20260912_registration_details.sql` and `20260912_refresh_recovery.sql`. These were local schema catch-up, not new migrations introduced by this development round.

## Results

| Check | Result and evidence |
| --- | --- |
| Google login | Passed with the account selected by the owner on the device. One `/auth/google` request returned 200 in 303 ms. Feed video playback and Profile opened. |
| Google session restoration | Passed after in-place APK update and a fresh launch; Feed opened without another login prompt. |
| Phone registration | Passed. The owner received and entered the OTP; `/auth/phone/register/verify-otp` returned 201 in 229 ms. The new phone-only Profile opened. |
| Subsequent phone login | Passed after explicit logout. The owner entered a second received OTP; one `/auth/phone/login/verify-otp` request returned 200 and Feed opened. |
| Phone session restoration | Passed after force-stopping and relaunching the app. It routed directly to `/home`, loaded Feed and played a video without another login prompt. This does not simulate 90 days elapsing. |
| Native sample-ad harness | Passed on the physical device: four distinct native objects ready, current placement plus three upcoming, four requests and one impression. Repeated forward/backward skips and detach/reattach passed. |
| Full-app Feed pagination and ads | Passed forward traversal of all 20 distinct eligible content videos and 10 sample-ad placements, including the final break. Page-change/playback logs show the exact `VVA` cadence through the additional batches; no unavailable-slot, unhandled Flutter exception or overflow markers occurred. The initial validator-obstructed attempt is excluded from this count. |
| Full-app Trending pagination and ads | Passed forward traversal of 38 distinct content videos and 19 sample-ad placements. The provider loaded 20 + 18 videos, then correctly stopped after an empty final request. Page-change/playback sequence is exactly `VVA` repeated 19 times. The final placement visibly rendered a Google Ads sample creative. No unavailable-slot, unhandled Flutter exception, overflow or `NO_ACTIVE_PLAYER` markers occurred. |
| Reverse scrolling | Passed through earlier content and native-ad placements on both surfaces: 13 recorded Feed backward page transitions and 15 Trending transitions. Some scripted swipes during this debug session did not produce a page change; no reversed content sequence or automatic bounce was recorded. This is not a release-mode frame-time benchmark. |
| Geographic filter recovery | Passed City and International empty-state checks against this dev dataset. City Refresh retained `scope=city`; International remained scoped; switching to All restored videos. The bottom of the sheet scrolls to International. |
| Light/dark appearance | The filter picker and aligned header remained readable in both app themes. Restored the original Dark setting afterward. Large text scales were covered by widget tests, not by changing this device's system font setting. |
| Age pill / mute alignment | Fixed and visually verified on this device. Regression checks verify right edge, equal vertical centers and spacing at 100%, 200% and 300% text scale on a 320 px screen. |
| Native ad action-button clipping | Fixed and visually verified: increase the medium native card's maximum height from 320 to 400 logical pixels while retaining the available-height bound. |

The two layout fixes passed 43 focused widget/unit tests; analysis of the three changed Dart files was clean. Earlier full mobile/backend verification is recorded in [the development report](LOGIN_FEED_ADS_FIXES_2026-09-13.md).

## Test-only adjustment

The installed debug APK disables Google's native-ad validator popups with `com.google.android.gms.ads.flag.NATIVE_AD_DEBUGGER_ENABLED=false`. The popups displayed “No implementation issues found” but intercepted repeated gestures in the full app. The flag was applied only while building this test APK; the debug manifest source was restored immediately afterward and the production manifest was not changed. Sample inventory remains enabled. No advertiser action button was clicked.

## Limits and remaining acceptance

- This device and USB connection do not establish recovery on the originally failing installation, on mobile data or on every older Android/iPhone.
- Sample-ad readiness cannot establish production fill, India relevance, unique advertisers or revenue. Paid-inventory acceptance remains required.
- Do not start the general development worker for upload testing yet: the saved development configuration points its MediaConvert event queue at the staging queue. The local full worker was stopped; only scoped feed workers were started. No evidence of MediaConvert polling was found in the stopped image's inspected logs. End-to-end upload processing remains unverified in this session.
- Physical geographic permission variants and production media visibility remain separate acceptance checks.

## Local evidence and continued testing

Temporary logs, screenshots, local database backup and build configuration are under `/private/tmp/iadme-device-20260913`. This directory contains private test data and must not be committed or published. Relevant logs include `api.log`, `device-app.log`, `device-app-v2.log`, `native-ad-device.log` and `device-fix-tests.log`.

The dev APK requires the Mac API and USB forwarding to remain available. It does not connect to production. The API, scoped feed workers and local database/cache were left running for owner testing; the app was left signed into the local phone account on Feed. Existing named production AAB/IPA build 37 artifacts are unchanged.

Saved APK, relative to the mobile repository: `apps/iadme_app/build/app/outputs/flutter-apk/iadme-1.0.4-build37-dev-device-20260913.apk`.

SHA-256: `6880fff4038d3d228198049b0678526aecff05c9de2a0b2ce255c3e059459268`.
