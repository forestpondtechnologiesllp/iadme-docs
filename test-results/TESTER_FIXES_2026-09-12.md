# Tester fixes — 12 September 2026

## Status

**Later release update:** The owner subsequently authorized backend deployment and production mobile builds. Both backend migrations and the API/worker release are now deployed to staging and production; see [RELEASE_BUILD36_2026-09-12.md](RELEASE_BUILD36_2026-09-12.md). The build-35 results and no-deployment statements below remain the historical development record. Owner internal testing and public mobile release remain pending.

Implemented locally in the mobile and backend repositories. Mobile version: **1.0.4+35**. Production was inspected read-only; nothing has been deployed or published. The [backlog](../roadmap/BACKLOG.md) now records implementation and remaining acceptance checks.

The subsequent [ad development pass](ADS_FIXES_2026-09-12.md) combines these fixes in mobile **1.0.4+36**, also undeployed. The version-35 results below remain the historical verification for this earlier pass.

Baseline commits: mobile `7adefdb`, backend `00ef5f2`. Changes remain in the working trees for review.

## Findings and resulting behavior

| Request | Finding | Change |
| --- | --- | --- |
| Backward scrolling fails on some Android devices | The supplied 31-second Trending recording shows an unavailable native-ad slot bouncing a backward swipe forward onto the same video. Both Feed and Trending always skipped unavailable ads with index + 1. | Skip the empty slot in the direction of the swipe, with bounds checks. Preserve feed child identity on updates. |
| Blank/stuck player, including issue 17/18 | Inactive widgets could use a replaced controller; new Android decoders could initialize before the old decoder was disposed. A rapid return to a card could lose its initialization request. | Serialize native initialization/disposal, cancel superseded requests, dispose the previous decoder first, reclaim ownership on reactivation and retain reactivation requests while cancellation unwinds. Retry a failed active decoder once, then expose manual Retry. |
| Social buttons appear idle | Buttons became disabled without visible progress during native provider handoff. | Paint a spinner and “Connecting…” before invoking Google/Apple; block repeated taps across the login choices. |
| First social signup asks for login twice | The consent path restarted native Google/Apple authentication. Apple authorization codes are single-use. | Backend returns an encrypted, five-minute continuation of the verified identity. After legal consent, the app completes registration without opening the provider again. |
| Keep successful logins for at least 90 days | Production access tokens last 15 minutes; refresh tokens were 30 days. Startup only checked access-token validity. Temporary refresh failures could clear saved credentials. | Restore/renew before routing, preserve credentials on temporary failures, store token pairs atomically, protect against late responses and account switches, and issue rolling 90-day refresh sessions. Rotation is transactional, with 60-second recovery for concurrent renewal/lost responses. |
| Registration device/location email | The original Dart User-Agent has no phone model; device permission can arrive after account creation. | Readable device headers plus an authenticated, registration-specific details endpoint and update email after the permission result. No exact coordinates, street address or fallback location is transmitted to this endpoint. |
| Biometric backlog | No independent in-app session lock existed. | Opt-in Face ID/Touch ID/fingerprint unlock in Profile, OS passcode fallback, account-bound preference, background obscuring and a lock outside the router. This unlocks a saved session; it is not passkey account sign-in. |
| Upload privacy backlog | Users were not told which location level other viewers see. | Add the explanation directly above the selected location after video selection. Existing public video serializers strip exact coordinates and honor the chosen area/city/state. |
| Ambiguous age label | Months appeared as 1m. | Months now appear as 1 mon, 2 mon, etc. |
| Diagnostic reliability | Seven monitoring requests in the inspected production window returned 500 because older clients submitted source flutter_admob_native, outside the backend enum. | Accept that legacy source as flutter, reject invalid events with 400, and include the actual mobile app/build plus device information in new monitoring events. Inactive/stale controller transitions no longer emit false NO_ACTIVE_PLAYER errors. |

## Production and GitHub evidence

Inspected the preceding 60 minutes of `iadme-prod-api` and `iadme-prod-worker` logs through `ssh iadme-prod` during investigation. The logs included 11 successful feed/v2 responses, two successful refresh responses, Google registration-required (409) followed by success, and successful Apple login. These observations support the client navigation/auth-flow findings; they do not demonstrate playback correctness on each device. Raw logs contain account data and remain outside the repository.

[Issue 17](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/17) and [issue 18](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/18) were read. Issue 18 has 121 recurrence comments; the latest inspected comment is **2026-09-07 07:31:50 UTC**. This is not evidence of 121 currently affected users. The supplied metadata identifies an inactive/stale widget and a different active controller, with an Android MediaCodec error. The events carry the backend release `iadme-api@prod-r1`, so they cannot reliably identify the mobile version. Both issues remain open; no comments or closures were posted.

The next mobile build reports `iadme-mobile@1.0.4+35` and device metadata, making new incidents distinguishable from older installations. Genuine active-player failures remain reported.

## Automated verification

- Full Flutter suite: **94 passed**, with two social-login tests skipped in the unconfigured run. The two tests also passed separately with a test Google client ID; **96 distinct Flutter tests passed** across the two configurations.
- Added coverage includes legacy secure-storage migration; simultaneous refresh; offline, 500 and malformed refresh responses; revoked sessions; late refresh after logout/account switch; account-bound metadata; backward ad skipping; decoder handoff; rapid A→B→A at provider and widget level; pending initialization teardown; decoder failure; social spinner and one provider invocation after consent; permission denial/revocation/approximate location; and biometric cancellation/background/account switching.
- Backend: TypeScript build passed; **9 unit tests passed** for encrypted continuation expiry/tampering, location validation/privacy, public feed coordinates and monitoring compatibility.
- Isolated PostgreSQL integration passed: 90-day token expiry, concurrent rotation returning one replacement, lost-response recovery, replay-window expiry, revocation, disabled accounts, notification deduplication and delayed permission upgrade. No test email was sent and no production database was changed.
- Both additive migrations were exercised against the isolated local database.
- OpenAPI YAML parses and every internal reference resolves.
- Dart analysis: no errors or warnings in the changed code. One pre-existing informational lint remains in `silent_sync_coordinator.dart:26` (`prefer_initializing_formals`).
- Android debug APK and iOS simulator debug app built successfully. The local iOS cache contained incomplete Google Ads/UMP frameworks; those pinned dependencies were re-downloaded. No dependency versions were upgraded as part of cache repair.

These are development builds with the local default configuration, not signed production distribution builds. Automated controller tests cannot reproduce every vendor MediaCodec implementation; native provider sheets and real biometrics require device acceptance below.

## Rollout order

1. Apply these migrations to staging, then to production through the normal database release procedure with stop-on-error enabled:
   - `iadme-backend/database/migrations/20260912_registration_details.sql`
   - `iadme-backend/database/migrations/20260912_refresh_recovery.sql`
2. Deploy the backend API **and worker** together. Set `AUTH_SESSION_DAYS=90` explicitly for operational clarity; the code also defaults to 90 and enforces a minimum of 90. Keep the existing short access-token lifetime. The legacy `JWT_REFRESH_EXPIRY=30d` value no longer controls new issuance.
3. Keep registration email enabled and verify the notification worker, Redis and email provider. The first email remains immediate; device/permission results use a separate update email, and a later permission grant can send one further update.
4. Run staging device acceptance, then build the current verified mobile candidate (now 1.0.4+36) using the established production API, Google sign-in, Firebase, ads and signing configuration. Release the mobile build after the backend is available. Old mobile clients retain their existing endpoints and request shapes. New clients need the new social continuation endpoint to avoid the second provider prompt.
5. Confirm refresh responses and registration-detail jobs after rollout. Triage only events carrying the new mobile release when deciding whether the old player issues can be closed.

Existing unexpired refresh tokens receive the longer lifetime on their next renewal. Expired/revoked credentials, account deletion, explicit logout, reinstall/keychain loss and security-secret rotation cannot be made persistent by this change. No already expired token is resurrected. Biometrics do not replace server authorization.

## Required device acceptance

Use at least one affected older/lower-memory Android and one current Android, plus a supported iPhone. Existing platform floors remain unchanged (Android SDK 24 via Flutter; iOS 15).

- Swipe backward/forward repeatedly across available and unavailable native ads in **Feed and Trending**; include rapid A→B→A, long scroll, tab changes, background/resume and poor connectivity. Verify audio belongs to the visible reel and manual Retry recovers after a forced player failure.
- Exercise premium previews, unlock/resume and in-stream ads because they share the single-player controller. Confirm seek restoration and mute state.
- Google and Apple: first registration from Login, direct Register, consent cancellation, native cancellation and repeat taps. Confirm one provider prompt per successful registration and immediate visible progress.
- Force an access-token expiry in staging; cold-launch, resume, toggle Wi-Fi/mobile data, simulate a refresh response lost after success, and test offline startup. Verify the saved session survives temporary failures. Test explicit logout, revoked session and switching accounts separately.
- Exercise email/password and phone OTP registration as well as social. Test permission granted, approximate-only, denied, permanently denied, disabled services, geocoder failure and later Settings grant. Confirm readable device fields and clearly separated IP/device location in the actual email; denied cases must contain no device area.
- Opt into biometrics; test successful Face ID/Touch ID/fingerprint, cancel, lockout, passcode fallback, changed enrollment, deep-link entry, background app preview and switching accounts. Unsupported/not-enrolled devices must retain normal login.
- Check upload copy on small screens/large text and confirm chosen locality/city/state matches the public label. Verify month ages read `1 mon`.

Physical acceptance and production release remain pending. Do not close issues 17/18 solely because unit tests and builds pass.
