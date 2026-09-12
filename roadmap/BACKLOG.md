# iAdMe Backlog

Last updated: 2026-09-12

The owner authorized implementation of IADME-001 through IADME-004 on 2026-09-12 as part of the tester fixes. They are implemented; backend prerequisites and both migrations were subsequently deployed to staging and production on the same date. Mobile store upload and physical-device acceptance remain pending. This replaces their earlier deferred status.

The owner subsequently authorized development and rigorous local testing of IADME-005 through IADME-008, explicitly excluding staging and production deployment. Ad loading, caching and placement changes are implemented in mobile 1.0.4+36. India relevance includes client context and configuration safeguards; live creative selection still requires AdMob serving verification.

The later release authorization covers production mobile artifacts and backend deployment to both environments. See the [build 36 release record](../test-results/RELEASE_BUILD36_2026-09-12.md) for artifact verification, image digest, migrations, backups and deployment checks. Public mobile release remains pending owner internal testing. **IADME-009 remains incomplete and deferred.**

See [implementation and release verification](../test-results/TESTER_FIXES_2026-09-12.md) for the associated feed, login and monitoring fixes, test evidence, migrations and remaining device checks.

## List

| ID | Item | Status | Scope |
| --- | --- | --- | --- |
| IADME-001 | Registration email: device details and location after permission | Backend deployed; mobile acceptance/release pending | Mobile + backend + migration |
| IADME-002 | Restore a valid session on fresh app launch | Backend deployed; mobile checks passed; device acceptance pending | Mobile + backend for 90-day sessions |
| IADME-003 | Optional Face ID / biometric unlock | Implemented; native device acceptance pending | Mobile |
| IADME-004 | Upload page: explain visible area and exact-location privacy | Implemented; public metadata checked | Mobile copy/UI |
| IADME-005 | Ads do not repeat at the configured interval in Feed/Trending | Implemented locally; verification recorded below | Mobile placement, pagination and readiness |
| IADME-006 | Some devices show no ads | Client recovery implemented; device/serving acceptance pending | Mobile consent, SDK, retries and release configuration |
| IADME-007 | Foreign ads shown instead of India-relevant ads | Client context implemented; AdMob serving verification pending | Mobile + AdMob account/campaign settings |
| IADME-008 | Show a skippable ad slot after every two videos | Implemented locally; no deployment | Feed + Trending placement/configuration |
| IADME-009 | Increase ad preloading with device-aware cache limits | Incomplete — deferred; not implemented | Mobile caching + performance/revenue evaluation |

## IADME-001 — Registration device and location details

**Outcome:** Requests identify the available manufacturer, model, operating system/version and app version/build. The original registration email is sent promptly. A separate registration-details email adds the device fields and the permission result after login, including a later location grant. Email/password, phone OTP, Google and Apple all use the same registration notification and authenticated app-entry flow.

- [x] Readable device details replace the generic Dart User-Agent on new mobile requests. No personal device name, serial number or advertising identifier is collected.
- [x] Location comes from the device only after OS permission; approximate permission is respected.
- [x] Capture locality/city/region/country, accuracy status and capture time. Neither coordinates nor street addresses are sent to this endpoint.
- [x] Separate IP-derived signup estimates from the device area captured after registration.
- [x] Denial, permanent denial, disabled services and lookup failures remain non-blocking and report unavailable location.
- [x] Never substitute the feed/upload fallback coordinates for registration location.
- [x] Associate captures with the authenticated registration; reject updates that cross accounts while permission is pending.
- [x] Versioned updates and deterministic queue IDs prevent ordinary retries/resumes from generating new notification jobs. After a granted capture is delivered, further location captures stop.
- [x] Automated permission, privacy, retry and database checks passed.
- [ ] Verify actual permission prompts, approximate-only settings, email delivery and later permission grant on Android and iOS staging devices.

**Design:** Keep the first email immediate, then send an update. Store only readable area details as part of the registration record; account deletion cascades to that record. This is a registration capture, with an optional later permission upgrade. It does not continuously track location. Existing accounts created before the migration have no capture record and are not backfilled.

**References (from workspace root):** `iadme-mobile/apps/iadme_app/lib/features/auth/data/registration_details_service.dart`; `iadme-backend/services/api/src/main/modules/registration-notifications/`; `iadme-backend/database/migrations/20260912_registration_details.sql`.

## IADME-002 — Session restoration and 90-day login

**Before:** Production used 15-minute access tokens and 30-day refresh tokens. Startup checked only the access token. A valid refresh session could therefore lead to a login screen.

**Implemented:** Startup renews the session before choosing the route. Expired/revoked credentials require sign-in; temporary connection/server failures retain credentials and show Retry. Token pairs are stored atomically, concurrent renewals share one request, and late responses cannot overwrite a newer login or undo logout.

The backend issues rolling 90-day refresh sessions by default, while access tokens remain short-lived. Rotation is transactional; a 60-second recovery window handles concurrent renewal or a lost response. Existing unexpired refresh tokens receive the longer window on their next successful renewal. Already expired/revoked tokens are not revived.

**Verification:** Automated legacy-storage migration, cold launch, concurrent renewal, offline/500/malformed responses, rejected refresh, logout/account-switch races, lost-response recovery, expiry and revocation checks passed. Physical cold-launch and long-background tests remain in the release checklist.

**References:** `iadme-mobile/apps/iadme_app/lib/features/splash/presentation/splash_screen.dart`; `lib/core/network/api_client.dart`; `lib/core/storage/auth_token_storage.dart`; `iadme-backend/services/api/src/main/modules/auth/`.

## IADME-003 — Optional biometric unlock

**Selected approach:** Optional unlock of an existing saved login. Enable it in **Profile → Biometric unlock** after signing in. Supported iOS devices use Face ID/Touch ID; Android uses enrolled device biometrics. The device passcode/PIN is an OS-controlled fallback. Passkey account sign-in is outside this implementation.

The lock surrounds the app router, hides content during background transitions and prevents deep links/back navigation from bypassing unlock. Initial private routes do not mount until unlock succeeds. Cancellation keeps the app locked. Explicit logout/account deletion removes the preference; Sign in instead clears the saved session and returns to normal authentication. Enabling/disabling requires device authentication. An expired/revoked server session still requires ordinary sign-in.

- [x] Android FragmentActivity, biometric permission and compatible native theme.
- [x] iOS Face ID usage text and native plugin integration.
- [x] Automated opt-out, cancellation, approval and background-lock checks.
- [ ] Physical Face ID, Touch ID/fingerprint, PIN fallback, lockout and changed enrollment checks on supported devices.

**Reference:** `iadme-mobile/apps/iadme_app/lib/core/security/biometric_lock.dart`.

## IADME-004 — Upload location privacy explanation

**Implemented copy:** “Only the area you select (sublocality, city or state) is shown on your video. Your exact location is not shared with other users.”

- [x] Appears directly above the selected location after a video is selected and before upload.
- [x] Remains visible when the selection changes and covers city/state selections as well as sublocality.
- [x] Verified existing public feed/video serializers remove latitude/longitude and use the chosen display level; a regression assertion covers the feed payload.
- [ ] Physical upload-page visual check on small screens and with large text.

**References:** `iadme-mobile/apps/iadme_app/lib/features/upload/presentation/upload_screen.dart`; `iadme-backend/services/api/src/main/modules/feed/feed-public-location.ts`; `iadme-backend/services/api/src/main/modules/videos/video-public-location.ts`.

## IADME-005 — Ads do not repeat at the configured interval

**Implemented:** Both surfaces use the same readiness-aware pager. The former rule omitted the final break: 18 videos at a three-video interval planned five breaks. The corrected planner includes the final break; the new default interval is two, giving **nine planned breaks for 18 content videos**. Existing direct-sponsored ads and Google slots share this cadence. Commercials never count toward the next two content videos.

- [x] Stable placement IDs anchored to preceding content; paginated direct campaigns do not replace earlier breaks.
- [x] Only ready native creatives enter the pager. Missing inventory leaves continuous content, with no blank commercial page or automatic bounce.
- [x] Preserve the same visible video when ads arrive, expire or are evicted. Defer changes during a drag.
- [x] Automated 18-video forward passes in Feed and Trending show nine filled ads when fake inventory is available; backward passes, pagination and rapid flings are covered.
- [x] Planned counts, request/load/failure/timeout counters, SDK impression callbacks and crossed unavailable slots distinguish placement from delivery.
- [ ] Owner acceptance on affected Android devices and a current Android/iPhone.

A planned slot is not a guaranteed paid impression: network, consent and ad-network inventory still determine fill. No ad is fabricated or used without permission to make up the count.

## IADME-006 — Some devices show no ads

**Implemented:** Replace the shared two-ad queue with demand for three upcoming placements and one previous placement. Retain recently viewed creatives for backward scrolling within a shared maximum of six native objects and two concurrent loads. Hidden tabs stop requesting. Unmounted cached ads expire after 55 minutes; memory pressure releases them, and a long background absence invalidates expired creatives before reuse.

- [x] Automatic retries after 15/30/60 seconds, capped at 60 seconds, without requiring another swipe.
- [x] Twenty-second application timeout; late success/failure callbacks cannot revive disposed ads or consume extra inventory.
- [x] Consent refresh and SDK initialization recover from temporary errors. Use valid saved UMP consent after starting the launch update; a visible consent form remains under user control.
- [x] Preserve consent denial/revocation; never fabricate permission or device location.
- [x] Reject malformed native IDs and sample IDs in production selection; a store-build preflight rejects missing IDs before invoking Flutter.
- [x] Report actionable SDK code/domain/response identifiers, build/test configuration, counters and retry timing with throttling.
- [x] Automated recovery, teardown, cache bounds, expiry, memory pressure, tab demand and lifecycle tests.
- [x] Real Google sample ads rendered and survived repeated skips and tab detachment on Android 11 with Google Play and iPhone 17/iOS 26.5 simulators; see the dated verification report.
- [ ] Real affected-device network/consent/ad-serving comparison; a client fix cannot guarantee AdMob fill on every request.

## IADME-007 — India-relevant advertising

**Client changes:** Native requests describe iAdMe's India/local-community/short-video content. Development and staging use only Google's official sample units; production requires the correct platform's real Native Advanced unit. Diagnostics explicitly distinguish test inventory. Existing backend direct campaigns already compare their country/region/city targeting against feed request geography; that filtering is retained.

Google's Flutter `AdRequest` has no publisher-side country/language switch that guarantees India-only creatives. Context keywords are relevance hints, not geo-targeting. Foreign-looking sample creatives do not establish what production will serve in India.

- [x] Add truthful content context without sending GPS, device identifiers or user-generated captions to the ad request.
- [x] Validate production versus test-unit selection on Android and iOS.
- [x] Document supported controls and the remaining serving boundary.
- [ ] Review the actual AdMob account's app/ad-unit status, country reports, mediation, blocking settings and any publisher-managed campaign country/language targeting.
- [ ] Verify real creative examples on India-based devices after a separately authorized release.

**No live AdMob configuration was changed.** See the [ad development and verification report](../test-results/ADS_FIXES_2026-09-12.md).

## IADME-008 — One skippable ad slot every two videos

**Implemented:** `ADMOB_NATIVE_FEED_INTERVAL` defaults to **2** in both Feed and Trending. A filled final break is permitted after video 18. Ready ads remain immediately scrollable in both directions; an unfilled break is omitted without interrupting videos.

- [x] Count content videos only; preserve cadence across accumulated pages.
- [x] Test breaks after 2, 4, 6 and onward, including nine breaks for 18 videos and the final page.
- [x] Preserve existing direct-sponsored alternation while preventing excess direct records from starving Google slots.
- [x] Separate cadence tests from inventory/network availability tests.
- [ ] Owner manual acceptance and separately authorized deployment/release.

## IADME-009 — Increase ad preloading with device-aware cache limits

**Status:** Incomplete. Added at the owner's request on 2026-09-12 for future work; no preload settings changed. Staging and production deployment remain excluded.

**Outcome:** Reduce missed ad opportunities during fast scrolling or variable network conditions while preserving video playback and scrolling performance on older and newer Android/iOS devices.

**Proposed experiment:** Increase lookahead from three to five native-ad placements and the shared cache/loading limit from six to eight, retaining at most two simultaneous ad loads. Keep a smaller cache on constrained devices and reduce demand under memory pressure. These are trial settings to validate, not established safe limits or a promised revenue increase.

**Scope:** Mobile Feed/Trending ad demand and cache management. The existing one-skippable-ad-opportunity-per-two-content-videos cadence stays unchanged. The cache refills throughout a session; its size is not a session-wide ad limit. More preloading cannot guarantee inventory, resolve serving restrictions or guarantee Indian creatives.

- [ ] Define and test how cache size adapts to device constraints and memory pressure.
- [ ] Compare the current settings against the proposed larger cache during fast forward/backward scrolling, tab switches, pagination, background/resume and slow/interrupted networks.
- [ ] Preserve readiness-only slots, stable scroll position, consent handling, bounded requests, retries, expiry and disposal of unused ads.
- [ ] Measure ad readiness misses, load latency, displayed impressions per session and revenue per session; increased requests alone do not establish improvement.
- [ ] Measure memory, bandwidth, battery impact, video buffering, scrolling smoothness, crashes and session retention, including affected older Android devices and current Android/iOS devices.
- [ ] Select final limits from the evidence, obtain owner manual acceptance and record results before any separately authorized release or live revenue comparison.

## Maintaining this list

Add future requests with a stable `IADME-###` ID, outcome, scope, acceptance criteria and status. Record test and release evidence separately; implemented locally does not mean deployed.
