# iAdMe Backlog

Last updated: 2026-09-14

**14 September development follow-up:** IADME-016, IADME-020 and IADME-023 are now implemented locally. Google profile enrichment preserves verified provider names/pictures without overwriting user edits; playback-start monitoring separates simulator/debug observations from repeated physical-release incidents; and Android network callbacks now serialize the ordered capabilities supplied by the OS. Automated checks passed. The additive profile-source migration, backend changes and mobile changes have not been deployed or released. Physical-device acceptance remains open. See the [development verification record](../test-results/BACKLOG_DEVELOPMENT_2026-09-14.md).

**Build 38 critical/high follow-up:** The owner authorized the network-recovery, ad-replenishment/native-media and automatic incident-reporting work after the [production device investigation](../test-results/ANDROID_BUILD38_PROD_DIAGNOSIS_2026-09-13.md). These changes are implemented locally with [automated/native test evidence](../test-results/RELIABILITY_MONITORING_FIXES_2026-09-13.md); no new deployment or store release has occurred. See the [incident-reporting runbook](../runbooks/mobile-incident-reporting.md). Physical-device acceptance and the underlying intermittent Wi-Fi cause remain open.

**Build 37 testing follow-up:** Android login failures, missing ads after tab switches/pagination, filter behavior and missing uploads were reported on 2026-09-13. Ad acceptance is reopened; previous local test results are not physical-device acceptance. See the [production investigation](../test-results/PROD_TRIAGE_2026-09-13.md) for confirmed defects, logs, reproduction and remaining questions. The subsequent fix implementation is documented in [development verification](../test-results/LOGIN_FEED_ADS_FIXES_2026-09-13.md). Development is local; no new deployment or store release has occurred.

The owner authorized implementation of IADME-001 through IADME-004 on 2026-09-12 as part of the tester fixes. They are implemented; backend prerequisites and both migrations were subsequently deployed to staging and production on the same date. Mobile store upload and physical-device acceptance remain pending. This replaces their earlier deferred status.

The owner subsequently authorized development and rigorous local testing of IADME-005 through IADME-008, explicitly excluding staging and production deployment. Ad loading, caching and placement changes are implemented in mobile 1.0.4+36. India relevance includes client context and configuration safeguards; live creative selection still requires AdMob serving verification.

The later release authorization covers production mobile artifacts and backend deployment to both environments. See the [build 36 release record](../test-results/RELEASE_BUILD36_2026-09-12.md) for artifact verification, image digest, migrations, backups and deployment checks. Public mobile release remains pending owner internal testing. **IADME-009 now includes the three-upcoming-ad refinement authorized on 2026-09-13; a larger device-adaptive cache remains incomplete.**

See [implementation and release verification](../test-results/TESTER_FIXES_2026-09-12.md) for the associated feed, login and monitoring fixes, test evidence, migrations and remaining device checks.

Follow-up [physical Android development testing](../test-results/ANDROID_DEVICE_DEV_2026-09-13.md) covers the owner's OnePlus 9 Pro: Google and phone login/session restoration, real-SMS phone registration, phone-only Profile, Feed's 20-video/10-sample-ad traversal and Trending's 38-video/19-sample-ad traversal passed. Both surfaces passed backward scrolling. Geographic empty-state recovery and light/dark checks passed. The same session fixed age/mute alignment and native-ad button clipping. Wider device/network and production ad-serving acceptance remains open.

The owner subsequently authorized [backend deployment to staging and production on 13 September](../test-results/RELEASE_BACKEND_2026-09-13.md), plus testing the current mobile code against production. Backend `37edc10` is deployed in both environments; the mobile changes remain pending store release. AdMob mediation is a separate backlog item.

## List

| ID | Item | Status | Scope |
| --- | --- | --- | --- |
| IADME-001 | Registration email: device details and location after permission | Backend deployed; mobile acceptance/release pending | Mobile + backend + migration |
| IADME-002 | Restore a valid session on fresh app launch | Backend deployed; mobile checks passed; device acceptance pending | Mobile + backend for 90-day sessions |
| IADME-003 | Optional Face ID / biometric unlock | Implemented; native device acceptance pending | Mobile |
| IADME-004 | Upload page: explain visible area and exact-location privacy | Implemented; public metadata checked | Mobile copy/UI |
| IADME-005 | Ads do not repeat at the configured interval in Feed/Trending | Additional cache/refill fixes implemented; physical acceptance pending | Mobile placement, pagination and readiness |
| IADME-006 | Some devices show no ads | Client recovery implemented; device/serving acceptance pending | Mobile consent, SDK, retries and release configuration |
| IADME-007 | Foreign ads shown instead of India-relevant ads | Client context implemented; AdMob serving verification pending | Mobile + AdMob account/campaign settings |
| IADME-008 | Show a skippable ad slot after every two videos | Implemented locally; no deployment | Feed + Trending placement/configuration |
| IADME-009 | Increase ad preloading with device-aware cache limits | Three upcoming placements implemented; larger adaptive cache deferred | Mobile caching + performance/revenue evaluation |
| IADME-010 | Shorten upload location label to “Show location as” | Implemented for build 37; device acceptance pending | Mobile copy/UI |
| IADME-011 | Android Google / phone login stalls or fails | Pre-API transport failures confirmed; network-change recovery implemented locally, physical acceptance pending | Native auth + network diagnostics/recovery |
| IADME-012 | Redesign Feed filters and correct their behavior | Backend deployed to staging/prod; Android picker verified against dev and prod, store release pending | Backend selection + mobile UI |
| IADME-013 | Missing uploads / Feed pagination skips eligible videos | Backend deployed and eligible Feed coverage verified; physical upload acceptance pending | Backend pagination + mobile readiness/refresh acceptance |
| IADME-014 | Add Meta Audience Network alongside Google through AdMob mediation | Backlog; deferred until build 38 acceptance | AdMob/Meta configuration + Android/iOS adapters + app-ads.txt |
| IADME-015 | Add Facebook social login | Backlog; not implemented | Mobile + backend + provider migration + Meta configuration |
| IADME-016 | Import and preserve social-login names and profile avatars | Implemented locally; migration/deployment/mobile acceptance pending | Mobile + backend profile handling |
| IADME-017 | Automatic GitHub incidents for AdMob and other observed inconsistencies | Implemented locally; device acceptance and deployment pending | Android/iOS diagnostics + backend/worker + migration |
| IADME-018 | Full-screen native ads and muted video autoplay | Implemented and simulator-tested locally; physical acceptance and release pending | Android/iOS presentation + lifecycle |
| IADME-019 | Add long-lived caching for immutable HLS media | Backlog; issue #25 investigation complete, not implemented | CloudFront + S3 + MediaConvert/backend |
| IADME-020 | Improve video-startup monitoring and incident thresholds | Implemented locally; physical release-mode tuning pending | Mobile diagnostics + backend incident grouping |
| IADME-021 | Prepare the next video without exhausting device decoders | Backlog; design and device experiment required | Mobile playback + Android/iOS native behavior |
| IADME-022 | Test HLS renditions for faster first-frame startup | Backlog; encoding experiment required | MediaConvert configuration + playback quality/performance |
| IADME-023 | Remove the Android network-callback capability race | Implemented locally; affected-device acceptance pending | Android native network diagnostics/recovery |

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
- [x] Preserve the same visible video when ads arrive, expire or are evicted. Defer structural changes from pointer-down through drag/scroll completion.
- [x] Automated 18-video forward passes in Feed and Trending show nine filled ads when fake inventory is available; backward passes, pagination and rapid flings are covered.
- [x] Planned counts, request/load/failure/timeout counters, SDK impression callbacks and crossed unavailable slots distinguish placement from delivery.
- [x] Fetch the next Feed/Trending batch with six content videos remaining so upcoming placements can preload; exclude commercial pages from the threshold and inactive tabs from prefetch.
- [x] Clear Trending's stale pagination lock when refreshing during an in-flight page request; regression reproduced before the fix and passed after it.
- [ ] Owner acceptance on affected Android devices and a current Android/iPhone.

A planned slot is not a guaranteed paid impression: network, consent and ad-network inventory still determine fill. No ad is fabricated or used without permission to make up the count.

## IADME-006 — Some devices show no ads

**Implemented:** Replace the shared two-ad queue with demand for three upcoming placements and one previous placement. Retain recently viewed creatives for backward scrolling within a shared maximum of six native objects and two concurrent loads. Hidden tabs stop requesting. Unmounted cached ads expire after 55 minutes; memory pressure releases them, and a long background absence invalidates expired creatives before reuse.

- [x] Automatic retries after 15/30/60 seconds, capped at 60 seconds, without requiring another swipe.
- [x] Twenty-second application timeout; late success/failure callbacks cannot revive disposed ads or consume extra inventory.
- [x] Consent refresh and SDK initialization recover from temporary errors. Use valid saved UMP consent after starting the launch update; a visible consent form remains under user control.
- [x] Preserve consent denial/revocation; never fabricate permission or device location.
- [x] Reject malformed native IDs and sample IDs in production selection; a store-build preflight rejects missing IDs before invoking Flutter.
- [x] Report actionable SDK code/domain/response identifiers, build/test configuration, counters and retry timing. The build 38 follow-up replaces event-dropping throttling with a bounded durable queue and counted/coalesced issues.
- [x] Automated recovery, teardown, cache bounds, expiry, memory pressure, tab demand and lifecycle tests.
- [x] Real Google sample ads rendered and survived repeated skips and tab detachment on Android 11 with Google Play and iPhone 17/iOS 26.5 simulators; see the dated verification report.
- [ ] Real affected-device network/consent/ad-serving comparison; a client fix cannot guarantee AdMob fill on every request.

## IADME-007 — India-relevant advertising

**Client changes:** Native requests describe iAdMe's India/local-community/short-video content. Development and staging use only Google's official sample units; production requires the correct platform's real Native Advanced unit. Diagnostics explicitly distinguish test inventory. Existing backend direct campaigns already compare their country/region/city targeting against feed request geography; that filtering is retained.

Google's Flutter `AdRequest` has no publisher-side country/language switch that guarantees India-only creatives. Context keywords are relevance hints, not geo-targeting. Foreign-looking sample creatives do not establish what production will serve in India.

Build 39 subsequently delivered a correctly rendered full-reel production ad on iOS whose copy used non-Indian text. No response ID was captured for that creative, so the observation cannot yet be traced to a specific buyer or campaign. It is included in the [AdMob support case](../test-results/ADMOB_SUPPORT_TICKET_2026-09-14.md) together with the Android no-fill evidence.

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

**Status:** Partially implemented after the owner authorized three or more preloaded ads on 2026-09-13. While an ad is visible, keep that ad plus three upcoming placements and the nearest previous placement within the existing shared six-object limit. Each new placement gets a distinct native object; only never-mounted, unimpressed orphan inventory may transfer to another slot. The cache continually refills and does not cycle the same three displayed objects through the session. A larger device-adaptive cache remains deferred. No deployment in this development round.

**Outcome:** Reduce missed ad opportunities during fast scrolling or variable network conditions while preserving video playback and scrolling performance on older and newer Android/iOS devices.

**Proposed experiment:** Increase lookahead from three to five native-ad placements and the shared cache/loading limit from six to eight, retaining at most two simultaneous ad loads. Keep a smaller cache on constrained devices and reduce demand under memory pressure. These are trial settings to validate, not established safe limits or a promised revenue increase.

**Scope:** Mobile Feed/Trending ad demand and cache management. The existing one-skippable-ad-opportunity-per-two-content-videos cadence stays unchanged. The cache refills throughout a session; its size is not a session-wide ad limit. More preloading cannot guarantee inventory, resolve serving restrictions or guarantee Indian creatives.

- [ ] Define and test how cache size adapts to device constraints and memory pressure.
- [ ] Compare the current settings against the proposed larger cache during fast forward/backward scrolling, tab switches, pagination, background/resume and slow/interrupted networks.
- [ ] Preserve readiness-only slots, stable scroll position, consent handling, bounded requests, retries, expiry and disposal of unused ads.
- [ ] Measure ad readiness misses, load latency, displayed impressions per session and revenue per session; increased requests alone do not establish improvement.
- [ ] Measure memory, bandwidth, battery impact, video buffering, scrolling smoothness, crashes and session retention, including affected older Android devices and current Android/iOS devices.
- [ ] Select final limits from the evidence, obtain owner manual acceptance and record results before any separately authorized release or live revenue comparison.

## IADME-010 — Shorten the upload location label

**Status:** Implemented for mobile 1.0.4+37 as part of the approved upload-screen redesign. Physical-device acceptance remains pending. See [build 37 verification](../test-results/RELEASE_BUILD37_2026-09-12.md).

**Requested change:** Replace the upload-page label “Show video location as” with “Show location as”.

**Scope:** Mobile copy only. Preserve the existing location-display options, selected value and privacy explanation.

- [x] Update the label to “Show location as”, including loading and unavailable-location states.
- [x] Verify the shared Flutter layout in light/dark themes, on small screens and with large text.
- [ ] Confirm the new screen on physical Android and iOS devices using build 37.

## IADME-011 — Android login failures

**Status:** The OnePlus Play installation was initially build 37, then updated and verified as build 38. Google and phone login both succeeded on cellular; independent device TCP/TLS probes also failed intermittently on Wi-Fi, outside Flutter. Wi-Fi subsequently recovered without an app/backend change. The fault within the Android/router/ISP path is still unproven, and the iPhone route was not measured. See the dated production investigation.

The follow-up mobile implementation replaces idle connection pools on native network changes, preserves in-flight writes, retains credentials during transport failures and captures native-versus-Dart health probes. Authentication requests still retry only once for a provable pre-connection failure; uncertain OTP/one-use credential outcomes are not blindly replayed. Native Google recovery, double-tap guards and the option to enter an already-received phone code remain. Physical acceptance of the new recovery path is open.

- [x] Identify the failing phone, build and installation source, and compare mobile data/Wi-Fi.
- [x] Capture safe native Google error codes and phone request network-stage errors in a bounded offline outbox, without tokens or phone/email contents.
- [ ] Fix the identified failure and verify native Google and phone OTP flows, cancellation, retries and app resume on affected and current Android devices.

## IADME-012 — Feed filter semantics and redesign

**Status:** Implemented locally. All discovers ready public videos with geographic priority; Nearby uses a 10 km radius; Local, City, State and Country remain within the selected area; International means outside the selected country. Missing required location is explicit. The theme-aware picker describes each option and the header shows the active choice.

- [x] Define and implement distinct scope behavior without silently widening an explicit geographic filter.
- [x] Redesign the picker and active-filter indication; light/dark, 320 px screens and 100–300% text scale tests pass.
- [x] Automated checks cover scoped results, missing coordinates, explicit empty results and stale responses after rapid filter changes.
- [ ] Physical GPS/approximate-permission acceptance on Android and iOS.

## IADME-013 — Missing videos and pagination coverage

**Status:** Feed cursor repaired locally. Stable keyset pagination freezes view preference at the start of a pass, excludes inaccessible content before selecting a page and returns an explicit end cursor. Tests now return all 30 videos exactly once, and cover 260 rows, microsecond ties, changing view history, GPS drift and old cursors. New ready videos enter the next refresh without waiting on Redis pool rebuilding. An owner-only publication-status endpoint and bounded mobile watcher distinguish processing from readiness and offer a direct View action.

- [x] Fix Feed pagination so reordering cannot advance past unreturned eligible videos.
- [x] PostgreSQL and mobile tests cover deduplication, exhaustion, blocked/reported content, late-ready uploads and request-generation races.
- [ ] Confirm the owner's specific missing examples in Feed, Trending and My Videos.
- [x] Test publication polling, temporary failures, terminal failure, timeout and disposal.
- [ ] Physical upload → processing → ready/View acceptance; an active feed is not forcibly moved while watching.

## IADME-014 — Add Meta Audience Network through AdMob mediation

**Status:** Added at the owner's request on 2026-09-13, then refined to select Meta Audience Network as the first additional partner alongside Google. Explicitly deferred: validate build 38 before a separate implementation and release. No mediation configuration or adapters have been enabled.

**Outcome:** Broaden eligible native-ad demand in Feed and Trending, with the aim of improving fill and creative variety while preserving the skippable opportunity after every two content videos. More networks do not guarantee unique advertisers, India-only creatives or higher revenue. The two observed Google sample creatives are not evidence of production inventory being limited to two ads.

- [ ] Measure current production fill, ad-source delivery, visible repetition, latency and revenue before enabling Meta demand.
- [ ] Add Meta Audience Network as a bidding source alongside Google for the existing Feed/Trending native placements; complete Meta account/property/placement setup, platform-specific AdMob mapping and app-ads.txt entries.
- [ ] Preserve one skippable opportunity after every two videos. Let mediation select one eligible ad for each placement, and retain the existing shared three-upcoming-placement preload limit.
- [ ] Validate Meta's native-media rendering and impression tracking with the current layout on Android and iOS, including India serving eligibility and available demand.
- [ ] Integrate compatible mobile adapters, initialization and consent/privacy messaging for the selected partners.
- [ ] Retain individual native-ad requests for mediated units: Google's multiple-native-ad batch APIs do not support mediation. The separate Google-only batch-loading proposal is not included in this item.
- [ ] Test each network using its test-device/test-ad configuration, then validate production serving after separate release authorization.
- [ ] Verify Feed/Trending pagination, three upcoming placements, forward/backward skips, no-fill recovery, theme/layout, memory and startup on affected older Androids and current Android/iOS devices.
- [ ] Compare fill, latency, creative repetition, impressions, revenue per user, app size, memory and crashes against the build 38 baseline before broad rollout; document remaining creative-repeat limitations.

References: [Meta bidding integration for Flutter](https://developers.google.com/admob/flutter/mediation/meta), [AdMob mediation for Flutter](https://developers.google.com/admob/flutter/mediation), [native batch-loading limitations](https://developers.google.com/admob/android/native), [test devices and mediation](https://developers.google.com/admob/flutter/test-ads).

## IADME-015 — Add Facebook social login

**Status:** Added on 2026-09-13 after the owner requested backlog items only. No SDK, auth route, provider configuration or database change has been made. This is a separate feature from Meta ad mediation.

**Outcome:** Offer Continue with Facebook alongside the current Google, Apple and phone/email flows, with the same registration consent and session behavior.

**Current gap:** Mobile has Google/Apple flows only; backend routes, identity types and registration continuation support only those providers. The database identity constraint in `003_create_auth_provider_identities.sql` also accepts only `google` and `apple`.

- [ ] Configure the Meta app for Android/iOS, including required signing identifiers, redirects, permissions and public availability requirements; select a maintained compatible mobile integration.
- [ ] Add mobile login/progress/cancel/error handling and backend verification of the supported Facebook credential types, including the applicable iOS Limited Login path.
- [ ] Add the Facebook provider through a new additive migration and update identity/registration-continuation handling. Verify provider subject, app/audience, expiry and applicable nonce; do not attach an existing account based solely on an unverified matching email.
- [ ] Support absent email and account-linking conflicts without duplicate accounts, duplicate signup rewards or a second forced login. Reuse current session persistence and logout behavior.
- [ ] Import permitted name/avatar data through IADME-016 while preserving user edits. Keep profile import optional and non-blocking.
- [ ] Verify first signup, existing login, cancel, denied/missing fields, offline recovery, repeated taps, account switching, token rejection, session restart and account deletion on Android/iOS. Regression-test existing Google/Apple/phone/email paths.

References: Meta's [Android SDK](https://github.com/facebook/facebook-android-sdk) and [iOS SDK](https://github.com/facebook/facebook-ios-sdk). Recheck detailed provider requirements when implementation begins.

## IADME-016 — Social-login names and profile avatars

**Status:** Implemented locally on 2026-09-14 after the owner authorized the pending backlog fixes. No application deployment, database migration, production data change or mobile release has occurred.

**Findings from the current source:**

- Google mobile authentication returns only the ID token from the account object; it does not forward the SDK's display name/photo URL separately. The backend does read verified `claims.name` when creating a new social user, but ignores `claims.picture` entirely. Social profile creation inserts only a name, so the avatar is not imported.
- Existing-provider login and linking a provider to an existing email account do not update the profile name/avatar. Signing in again therefore does not replace an old generated name or populate a missing avatar.
- The backend name sanitizer removes everything except ASCII letters, digits and spaces, truncates to 15 characters, and generates a friendly nickname when empty. Names in scripts such as Telugu or Hindi can therefore become generated names; longer names can be cut off.
- Apple mobile requests full name/email and forwards the returned name. Apple generally supplies the name only at the first authorization, and Sign in with Apple does not supply a profile photo. A missing or previously lost Apple name cannot be assumed recoverable through ordinary repeat login. The current registration continuation preserves the name it receives, but cannot reconstruct one Apple did not return.
- These are code-level findings, not a claim that every missing-name account has the same cause. No customer-specific identity payload or production profile was modified during this investigation.

**Outcome:** Prefill available social profile information correctly, preserve user-chosen details and offer a graceful editable fallback when a provider omits information.

- [x] Import a Google name and picture from verified provider claims when available and preserve them through registration consent/continuation. Missing names/pictures do not block login.
- [x] Preserve Apple's first-authorization name through registration/retry. The existing profile editor supplies an editable name and user-uploaded avatar when Apple omits them; Apple photo import is not claimed.
- [x] Replace the 15-character ASCII-only rule with a 100-code-point Unicode rule across mobile registration, backend registration, provider import and profile editing. Control/format characters are removed and whitespace normalized.
- [x] Track display-name and avatar origins. Provider data can refresh provider/generated values and fill a reliably missing legacy avatar; a user edit or cleared photo is marked `user` and cannot be overwritten by a later social login. The migration marks only exact known generated-name patterns for safe enrichment.
- [x] Accept provider avatars only from HTTPS Google-hosted URLs, strip fragments and retain existing initials/upload fallback behavior. Profile enrichment remains optional to authentication data and never substitutes unverified client fields.
- [ ] Extend the same behavior to Facebook when IADME-015 is implemented.
- [x] Automated checks cover continuation, omitted values, non-Latin/long names, unsafe avatar URLs and existing login/profile contracts.
- [ ] Complete new/existing/linked Google and Apple physical-device acceptance, including registration interruption, changed provider photos and preservation of manual edits on Android/iOS.

**Source references (from workspace root):** `iadme-mobile/apps/iadme_app/lib/features/auth/data/social_auth_service.dart`; `iadme-backend/services/api/src/main/modules/auth/social-auth.service.ts`; `iadme-backend/services/api/src/main/modules/auth/registration-continuation.ts`; `iadme-backend/services/api/src/main/modules/profile/profile.repository.ts`.

Provider references: [Google name/picture claims](https://developers.google.com/identity/openid-connect/reference), [Apple first-authorization details](https://developer.apple.com/documentation/signinwithapple/authenticating-users-with-sign-in-with-apple), [Apple staff explanation of photo availability](https://developer.apple.com/forums/thread/121998).

## IADME-017 — Automatic diagnostic GitHub incidents

**Status:** Implemented, committed and pushed on 2026-09-13. Backend migration/API/worker deployed to staging and production as `release-2026.09.13-7eee307`; controlled staging delivery and restart recovery passed. A new mobile release and physical acceptance remain pending. The owner requested AdMob support data for no-fill as well as tickets for other observed inconsistencies.

- [x] Capture no-fill, network/load timeout, missed placement and native-media observations with available SDK response/source IDs, versions, UTC timing, device/build, consent, network and per-surface fresh-inventory counters.
- [x] Queue diagnostics securely while offline and retain counted repeats instead of dropping all repeats for five minutes. Bound storage, redact sensitive values and reuse event UUIDs across uncertain acknowledgements.
- [x] Persist on the backend before acknowledgement, with durable GitHub retries, concurrent-worker claims, grouped counts and recent samples. Preserve human issue notes and reopen matching recurrences.
- [x] Connect final API failures, existing auth diagnostics, captured Flutter/runtime errors and application error logging. Information-only success events do not create issues.
- [x] Create the confirmed production reports: [Android no-fill #19](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/19), [iOS no-fill #20](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/20), [Android black media #21](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/21), [Android Wi-Fi login transport #22](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/22).
- [x] Deploy the additive migration/API/worker and verify a controlled staging incident, grouped updates and restart recovery. See [deployment evidence](../test-results/RELEASE_MONITORING_2026-09-13.md).
- [ ] Complete physical Android/iOS acceptance and mobile rollout.

**Limits:** No-fill is an observed serving outcome, not automatic proof of an AdMob defect. Counts do not measure unique creatives or paid impressions. Uninstrumented visual defects and abrupt native crashes are not guaranteed to reach this queue. See the [reporting runbook](../runbooks/mobile-incident-reporting.md) for payload limits, privacy, monitoring and rollout order.

## IADME-018 — Full-screen native ads and muted video autoplay

**Status:** Implemented, committed and pushed as mobile `697fac5` on 2026-09-13. No backend or AdMob account setting change is required for this presentation change; a new mobile release is required. Android/iOS native video, static-image and compact-layout checks passed; the full mobile suite passed 231 tests with two existing skips.

- [x] Replace the 400 × 400 card with a viewport-filling native view in Feed and Trending, with a dark media canvas and compact advertiser/action footer.
- [x] Preserve creative proportions and accept any aspect ratio, retaining landscape/image inventory as well as portrait/video inventory.
- [x] Use SDK-owned muted autoplay, native controls, media binding and playback observations. Offscreen/hidden-tab views detach with safe cached-ad ownership; temporary visible focus loss preserves the native view.
- [x] Keep SDK attribution, AdChoices and advertiser asset click handling. The ad remains vertically skippable.
- [x] Add small-phone/tablet/landscape, theme, enlarged-text and lifecycle regression coverage.
- [x] Complete native-video/static-image/compact-footer checks on Android 11 and iOS simulators.
- [ ] Complete physical acceptance and a new mobile release.

See [full-screen ad validation](../test-results/FULLSCREEN_NATIVE_ADS_2026-09-13.md). Full-screen presentation does not force every creative to be 9:16 or video, and cannot guarantee paid fill or override SDK playback restrictions.

## IADME-019 — Long-lived caching for immutable HLS media

**Status:** Added from the investigation of [mobile issue #25](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/25). No CloudFront, S3, MediaConvert, backend or production change has been made.

**Finding:** Production CloudFront uses the managed caching policy with a one-day default TTL. The inspected HLS master manifests, rendition manifests and transport-stream segments have no explicit `Cache-Control` or expiry metadata. During the investigation, cold manifest requests took approximately 0.65–2.05 seconds while immediate CloudFront hits took approximately 0.05 seconds. Media requests bypass the API, whose Feed/Trending responses were healthy during the incident window.

**Outcome:** Keep completed, immutable HLS media cached long enough to avoid unnecessary S3-origin retrieval while retaining a safe way to publish corrected or reprocessed media.

- [ ] Define separate CloudFront behavior and TTLs for HLS manifests and segments. Use versioned media URLs or an explicit invalidation/reprocessing policy before marking long-lived objects immutable.
- [ ] Apply the policy to existing media as well as future MediaConvert output; changing only the future upload path does not cover the current catalogue.
- [ ] Preserve correct content types, range requests, access controls and incomplete-processing behavior. Never expose source uploads through a broader public cache rule.
- [ ] Verify cold and warm requests from representative India locations, cache headers, cache age, origin-request reduction and playback after a controlled media replacement.
- [ ] Measure CloudFront/S3 cost, cache-hit behavior and first-frame timing before and after the change on physical Android and iOS devices.

## IADME-020 — Video-startup monitoring and incident thresholds

**Status:** Implemented locally. The issue's three 3.4–8.9 second samples came from an iPhone 17 debug simulator using production APIs; related physical-device samples were approximately 2.4–2.7 seconds. Simulator/emulator and debug/profile observations remain diagnostic events but cannot create production incidents. No deployment or mobile release has occurred.

**Outcome:** Preserve actionable playback alerts while preventing successful simulator/debug starts above a fixed 1.5-second threshold from being presented as production device incidents.

- [x] Add explicit simulator/emulator versus physical-device, debug/profile/release build and configured backend-environment fields to playback events and GitHub issue titles/bodies.
- [x] Record separate durations for controller queue/disposal, controller initialization/configuration, play request, first positive position and first presented Flutter frame.
- [x] Apply provisional physical release thresholds by validation and network class: 4 seconds on validated Wi-Fi/Ethernet, 5 seconds on validated cellular/VPN and 6 seconds otherwise. Non-release/non-physical observations use 10 seconds and remain information-only. Incidents require at least two severe reports.
- [x] Keep initialization failures, playback failures and prolonged user-visible stalls independently actionable; performance aggregation applies only to successful slow-start events.
- [x] Bound slow-start reporting to three events per app session and separate incident identities by device/build context.
- [ ] Verify event privacy, bounded reporting, offline delivery, issue grouping and release attribution on simulator and physical Android/iOS test matrices.

## IADME-021 — Guarded next-video preparation

**Status:** Backlog experiment. The current single-controller design deliberately serializes disposal and initialization because some older Android devices cannot initialize a second HLS decoder safely.

**Outcome:** Reduce swipe-to-first-frame time by preparing at most the next likely video without reintroducing decoder exhaustion, stale-controller races, excessive data use or feed-position changes.

- [ ] Prototype a one-video-ahead preparation path that can warm network/media state without owning a second active decoder on constrained devices; document platform-specific behavior where Flutter's video plugin does not expose safe preparation APIs.
- [ ] Keep the existing serialized single-player fallback for affected/low-memory Android devices and cancel obsolete preparation immediately after rapid swipes, tab changes, ads or route changes.
- [ ] Bound memory, decoder count, connections and downloaded bytes. Disable or reduce preparation on constrained networks, Low Data Mode/data saver, memory pressure and backgrounding.
- [ ] Test Feed and Trending forward/backward scrolling, pagination, ad transitions, A→B→A races, long sessions and app resume on older Androids and current Android/iOS devices.
- [ ] Compare first-frame percentiles, buffering, scroll smoothness, memory, battery and mobile-data use against the current single-controller baseline before selecting an implementation.

## IADME-022 — HLS first-frame encoding experiment

**Status:** Backlog experiment. Current output uses two-second HLS segments with 360p, 720p and 1080p renditions. The inspected first segments ranged from approximately 195–256 KB at 360p, 679 KB–1.01 MB at 720p and 1.43–2.09 MB at 1080p.

**Outcome:** Reduce the amount of media needed for first playback while maintaining acceptable visual quality and stable adaptive streaming.

- [ ] Benchmark current MediaConvert outputs and confirm which rendition and how much buffered media Android and iOS select before the first frame under fast, slow and changing networks.
- [ ] Test lower 720p/1080p maximum bitrates and QVBR quality, an intermediate rendition where useful, and shorter initial/segment duration options. Change one variable at a time.
- [ ] Compare first-frame time, rebuffering, rendition switching, perceptual quality, storage, MediaConvert time and CloudFront request/transfer cost across portrait, landscape, 30 fps and 60 fps inputs.
- [ ] Preserve source aspect ratio, audio, thumbnail alignment and compatibility with older Android decoders and iOS AVPlayer.
- [ ] Apply a new encoding profile to future uploads only until a separately approved existing-catalogue re-encode plan proves its cost, cache and publication behavior.

## IADME-023 — Android network-callback capability ordering

**Status:** Implemented locally from the investigation of [mobile issue #22](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/22). Native Android tests pass; no mobile release has occurred.

**Finding:** `onCapabilitiesChanged(network, capabilities)` currently discards the ordered `NetworkCapabilities` supplied by Android and calls the general `snapshot()` method. That method synchronously queries `activeNetwork` and `getNetworkCapabilities()` again. During Wi-Fi/cellular handover, the second query can observe a different or stale active network and publish an inconsistent transport, validation or availability state.

**Outcome:** Publish Android network changes from the callback data Android delivered for that network, while retaining a separately requested current-state snapshot for Flutter startup and diagnostics.

- [x] Pass the callback's `Network` and `NetworkCapabilities` directly into the event serializer for `onCapabilitiesChanged`; no synchronous capability re-query occurs inside that callback.
- [x] Define synchronized handling for `onAvailable`, `onCapabilitiesChanged` and `onLost`, with monotonically increasing revisions, engine epochs and protection against late callbacks from old networks.
- [x] Keep `snapshot()` for explicit method-channel requests while callback and snapshot payloads use consistent transport, availability, validation and metering fields.
- [x] Add native tests for Wi-Fi/cellular/VPN transitions, unvalidated Wi-Fi, rapid loss/recovery, stale networks and callbacks after teardown or a new engine.
- [ ] Verify the resulting Flutter connection-pool recovery and login behavior on the affected OnePlus device and current Android versions without changing the successful iOS path.

## Maintaining this list

Add future requests with a stable `IADME-###` ID, outcome, scope, acceptance criteria and status. Record test and release evidence separately; implemented locally does not mean deployed.
