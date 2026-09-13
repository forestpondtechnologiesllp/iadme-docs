# Android build 38 production login and ads investigation — 13 September 2026

## Scope

Investigated the owner's connected OnePlus 9 Pro (LE2121) and production API after the reported login failure. Subsequently inspected the owner's manual Feed/Trending ad test. No application code, backend deployment, DNS, TLS configuration or advertising configuration was changed. A Google Play update was applied in place; Wi-Fi was temporarily disabled for the network comparison.

## Installed artifact

- Initial Play Store installation reported version code **37**, installed at 17:31:35 IST. Its arm64 `libapp.so` SHA-256 exactly matched the saved production build 37 AAB: `f3c417a8266195fb3358da7a8ed03b1ad028f84d129b51af627c9191c5858370`.
- Google Play offered an update on the internal testing listing. The update completed at **17:44:14 IST**, reporting version code **38**.
- The updated arm64 application code exactly matched the saved production build 38 AAB/APK: `3e5e5c41945a64b56e5d228d3d1621f849bba796e5fd44ca0b5d288daef8e263`. The compiled release identifier changed from `iadme-mobile@1.0.4+37` to `iadme-mobile@1.0.4+38`.
- Both installed versions contained the production API address, not a local or staging API address. Internet permission was present; the device's iAdMe settings allowed both Wi-Fi and mobile data. App data was preserved by the Play update.

## Login and network evidence

| Check | Result |
| --- | --- |
| Original recorded attempts | Google, phone and email displayed a server-connection error. No authentication requests appeared in the inspected production API window. |
| Build 38 Google attempt on Wi-Fi | Failed before receiving an API response. The diagnostic later uploaded by build 38 records `/auth/google`, `connectionTimeout`, captured at **17:45:15 IST**. |
| Phone's independent HTTP client on Wi-Fi | Reproduced failure outside Flutter/iAdMe. One TLS trace connected to `98.84.254.173:443`, received ServerHello and EncryptedExtensions, then timed out. Other attempts timed out establishing TCP. |
| Same public API health check on cellular | Passed, certificate verification enabled, TLS 1.3 and HTTP 200. The carrier used a NAT64 address (`64:ff9b::6254:fead`), a different network path from the Wi-Fi IPv4 attempt. |
| Wi-Fi restored and health check repeated | Failed again. |
| Google login on cellular, same build 38 | Owner selected the same account and confirmed the app opened. Production `/auth/google` returned **200 in 106 ms at 17:49:46 IST**. The prior Wi-Fi failure diagnostic uploaded after connectivity recovered. |
| Phone login on cellular, same build 38 | Owner completed phone OTP entry. `/auth/phone/login/send-otp` returned **200 in 343 ms at 17:59:57 IST**; `/auth/phone/login/verify-otp` returned **200 in 88 ms at 18:00:21 IST**. A subsequent force-stop/relaunch opened Feed without another login. |
| Final Wi-Fi recovery | After reconnecting again, the independent phone HTTP client completed certificate-verified TLS 1.3 and returned HTTP 200. A fresh browser health URL with a diagnostic query parameter also reached the API and returned 200, excluding a cached-page explanation for that final browser result. No server/app configuration change occurred between failure and recovery. |
| App restart after Wi-Fi recovered | Opened Feed with the phone-authenticated session intact. Nginx requests attributed to `iAdMe/1.0.4+38` and `LE2121` confirm HTTP 200 for `/feed/v2`, `/users/me/registration-details`, blocked-user and unread-count requests around **18:05:18–18:05:50 IST**. |
| Other checks | Mac HTTPS and HTTP/1.1 checks passed. The Android browser displayed the API health response on Wi-Fi; this single observation does not establish sustained connectivity or an identical transport/cache path. |

The failure is reproducible below authentication and was intermittent: Wi-Fi failed repeatedly, but later recovered after reconnection. A Google OAuth configuration change or OTP backend rewrite is not supported by this evidence. The precise fault within the Android Wi-Fi/router/ISP path remains unproven. Packet loss, filtering, packet-size problems or device/network handling differences require controlled follow-up; none is asserted as the confirmed cause. Recovery is not evidence of a permanent fix.

The iPhone was not connected for this investigation. Its DNS results, network route and cellular fallback were not measured. iPhone success therefore does not establish that both phones used the same path. Apple documents Wi-Fi Assist, but its participation in this iPhone session has **not** been established, and it does not apply to every app/use case.

## Live ads: acceptance not passed

The owner reported only approximately four or five ads across the two surfaces, with long gaps. The desired repeated ad opportunity after every two videos was **not validated**.

Visual evidence:

- Feed captures contain three Olymptrade placements with rendered media, text and action buttons. They repeat the same advertiser/creative appearance.
- Trending captures contain Blinkit and Tango. The Blinkit card has a black media rectangle in consecutive captures, while its advertiser, descriptive text and action button are visible. Tango's media and text render.
- These establish three observed advertiser brands, not a count of all unique creatives served by AdMob.
- No advertiser action button was intentionally clicked. Initial injected slow swipes did not reliably advance the feed; the owner clarified that they helped scroll. Those gestures are not treated as independent evidence of an app scrolling regression or a complete cadence pass. Later observations use the owner's manual traversal.

Production SDK snapshots (Android, build 38, `testAdUnit=false`):

| Time IST | Event | Snapshot |
| --- | --- | --- |
| 17:53:05 | `ADMOB_NATIVE_LOAD_FAILED` | Android SDK code **3**, `No fill`; 6 requests, 5 successful loads, 3 impression callbacks, 0 in-flight loads, approximately 15-second refill cooldown. |
| 17:54:47 | `ADMOB_NATIVE_INVENTORY_GAP`, Trending | 10 requests, 5 successful loads, 5 impression callbacks, 0 in-flight loads; no ready creative for the placement. |

These are snapshots, not final session totals. Repeated diagnostics are suppressed for five minutes per event name, across surfaces. `readyAds` counts available cached entries, including already-shown creatives; the value of four at the gap does **not** mean four fresh creatives were available for the next placement. An SDK impression callback does not prove every media asset rendered correctly or establish billable revenue.

## Proposed corrective work

1. **Connection diagnosis and recovery:** reproduce on a second Wi-Fi network and compare both phones with cellular disabled. Capture DNS, routes and a narrowly scoped server-side TCP/TLS trace during a failing request before changing infrastructure. Improve mobile reporting of transport/connection stage and recovery on network changes. Preserve sessions on transport failure and avoid blindly repeating OTP or one-use-token exchanges. Switching to cellular worked, and Wi-Fi later recovered after reconnection; neither establishes a permanent fix. Build 38 already retries qualifying connection failures, so simply adding another blind retry is not sufficient.
2. **Ad availability and refill:** distinguish fresh upcoming inventory from previously shown cache entries. Review shared cooldown behaviour with measured load/no-fill/recovery scenarios while preserving the existing bounds and backward-scroll guarantees. Increasing the preload count alone cannot create advertiser inventory.
3. **Ad media display:** investigate the loaded-but-black media case independently of no-fill using device-native SDK diagnostics. Do not treat `onAdLoaded` as proof that all media has painted, or use absence of an impression as an automatic render-failure signal.
4. **Broader inventory:** the existing AdMob/Meta mediation backlog remains relevant. Additional demand can improve availability, but cannot guarantee a paid ad after every two videos. Strict visual cadence would also need an explicitly agreed direct/house-ad fallback when paid inventory is unavailable; that would not manufacture paid impressions.
5. **Acceptance:** repeat on the affected Android and a physical iPhone, across Wi-Fi/cellular, first load, pagination, tab return and fast/backward scrolling. Record per-placement request, load, display and failure outcomes. Current Android ad cadence and Blinkit media display remain open findings.

## Private evidence and sources

Private APK copies, app-scoped logs, TLS traces and screenshots are under `/private/tmp/iadme-build38-android-login-20260913`. The original recording includes private login information and must not be published. Device screen recording through the command-line tool failed; periodic screenshots were used instead.

Final state: build 38 installed from Google Play, phone session signed in, original Wi-Fi setting restored and working at the final check. No advertiser configuration, app source or production infrastructure change was made. The intermittent Wi-Fi cause and the ad availability/media findings remain unresolved; successful recovery is not represented as a permanent correction.

- [Android network state and network changes](https://developer.android.com/develop/connectivity/network-ops/reading-network-state)
- [Apple Wi-Fi Assist](https://support.apple.com/en-us/102228)
- [AdMob mediation](https://developers.google.com/admob/flutter/mediation)
