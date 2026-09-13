# Login, Feed, ads and publication fixes — 13 September 2026

## Scope and release state

Implements the owner-authorized follow-up to [production triage](PROD_TRIAGE_2026-09-13.md), including the request to preload three upcoming ads and refill with fresh placements. Source changes are local to mobile/backend. No staging/production deployment or migration, live AdMob setting change or store submission occurred. The subsequent physical test applied existing additive migrations only to the backed-up local development database; no new migration is required by these fixes. Existing build 37 production artifacts are not the new implementation.

## Implemented behavior

### Android Google and phone authentication

- One fresh-connection retry for authentication failures proven to occur before connecting, including connection timeout and selected DNS/no-route/refused socket failures. Reads retain their safe retry. Receive/send timeouts and connection resets do not automatically replay auth writes, avoiding duplicate OTP delivery or reuse of a consumed Apple credential.
- Preserve the final HTTP response if recovery reaches the server; do not mask a real account/validation response with the original connection error.
- Google initialization shares an in-flight future and resets after failure. Cancellation stays quiet; native interruption, UI/configuration and connection failures have distinct actionable messages. Native sign-in is never automatically launched twice.
- Phone submission normalizes Indian numbers, guards repeated taps, disables editing during a request, and permits entry of an already-received code after an uncertain network outcome. Background device registration no longer delays successful phone sign-in. Late failures cannot access disposed screen context.
- Sanitized diagnostics use an eight-event secure-storage outbox, survive offline failure, serialize capture/acknowledgement, and flush on a later successful API connection. No provider token, phone, email, raw native error text or exact location is added to these events.

**Limit:** the recording's exact native Google error remains unknown. This addresses demonstrated recovery gaps and adds evidence for the next attempt; it does not establish that every native/provider failure on the affected phone is resolved. No TLS bypass, authentication weakening or forced logout was introduced.

### Ads in Feed and Trending

- A visible ad no longer consumes one of the three lookahead positions: retain the current ad, three upcoming placements and a previous placement within the shared maximum of six native objects and two simultaneous loads.
- New placements receive distinct native objects. Never-mounted/unimpressed loaded inventory orphaned by a changed demand can transfer to an upcoming slot. Mounted or previously presented objects cannot be rotated through new placements. Backward scrolling may revisit the same stable placement and its own object.
- Simultaneous no-fill failures count as one failed refill round (15 seconds initially), followed by bounded 30/60-second recovery. Unavailable-slot diagnostics identify the surface and readiness/loading counters.
- Page changes now freeze from pointer-down, before drag recognition. Existing two-content-video cadence, ready-only pages, stable scroll position, consent, expiry, memory-pressure cleanup and disposal remain in force. No empty ad is inserted just to satisfy a count. Three ads are a replenished buffer, not the session-wide inventory or a loop of three shown objects.
- Both active surfaces fetch the next batch when six content videos remain, including after page reconciliation. Native/direct ads do not count toward that threshold. This gives upcoming placements time to load before the batch boundary; hidden surfaces do not prefetch.
- Trending refresh now clears the previous generation's pagination lock. A deterministic test first reproduced the failure: the old page was correctly ignored after refresh, but subsequent page requests stayed blocked. The corrected run loads the new cursor and excludes the stale response.

**Read-only AdMob observation:** account approved/serving enabled; Android app marked Ready. Dashboard's displayed last-seven-days aggregate: 342 requests, 104 impressions, 47.37% match rate. Android: 155 requests, 16 impressions, 22.58% match rate. These are aggregate historical snapshots, not a measurement of this unreleased code or proof of every reported failure's cause. The Android unit showed no active mediation group and no frequency cap applicable to its native format. Full app policy/floor/country review was not completed; no account settings were changed.

Google controls the creative returned and may repeat an advertiser or return no-fill. A larger buffer cannot guarantee a unique creative, India-only inventory, a paid impression or revenue. See Google's [native ad integration](https://developers.google.com/admob/flutter/native?hl=en) and [match-rate definitions](https://developers.google.com/admob/api/reference/rest/v1/accounts.mediationReport/generate#Metric). The larger five-lookahead/eight-object device-adaptive experiment remains incomplete in IADME-009.

### Feed filters and missing videos

- Replaced mutable Redis-offset selection with database keyset discovery for Feed v2. Sort order freezes watched status at the pass snapshot, then geographic priority (All), publication time and ID. A new pass sees newly ready videos directly; pagination does not depend on Redis pool hydration.
- Old cursor IDs are retained while incorrect old offsets are ignored. New compact cursors do not drop the first IDs after 200 videos; query/user binding, bounded decompression and timestamp/coordinate validation reject malformed cursors safely.
- Nearby uses a real 10 km radius and freezes its original GPS centre through a pagination pass. Local/City/State/Country are strict; International excludes the chosen country. Missing required location is returned explicitly. Existing reviewer fallback area behavior remains identified by the mobile fallback notice.
- Mobile freezes location during a pass, ignores stale filter/refresh/page responses, deduplicates overlapping results and preserves the selected scope when retrying. The picker explains each scope, shows its current choice and supports theme/text-size changes.
- Existing hydration, public-location serialization and direct-ad assembly remain in use. Owner-only processing-status polling adds a ready/View message without jumping an actively watched feed; processing failure directs the owner to My Videos. Polling pauses in background, retries transient errors, ends after three minutes and is disposed with the authenticated app shell.

**Coverage limit:** the confirmed missing-video cursor defect was in Feed. The reported uploads were already ready and present in Trending's global pool. Trending ranking, Profile visibility and the user's specific physical-device examples still require manual acceptance; a new upload is not promised the first Trending rank.

## Automated verification

- Full mobile suite: **214 passed, two configuration-gated tests skipped**. Those two social-login progress/registration-continuation tests separately passed with production-mode and synthetic OAuth configuration (**216 mobile unit/widget tests verified in total**); no real account authentication occurred. The final focused run also passed all three Feed/Trending request-generation tests.
- Changed Dart files: **no analyzer issues**. Full-project analysis additionally reports one pre-existing informational `prefer_initializing_formals` lint in `silent_sync_coordinator.dart` outside this change.
- Backend: **34 passed**, including 12 real-PostgreSQL discovery cases, eight owner/status endpoint cases and existing registration/profile compatibility tests. TypeScript build passes.
- Read-only production schema validation confirmed column names/types used by the new query. Integration rows live only in an isolated loopback PostgreSQL instance on port 55439, using temporary tables.
- Native iOS: iPhone 17 simulator / iOS 26.5, actual Google sample SDK inventory: **passed**. Four requests produced the current placement plus three distinct upcoming native objects; native view rendered, immediate forward/backward skips and detach/reattach passed. The harness now settles after the final asynchronous preload and derives gesture coordinates from the actual viewport; its initial unsynchronized first-swipe assertion failed before this correction.
- Native Android 11, official Google Play image: **passed**. Four requests produced four distinct ready sample objects; the native ad rendered, repeated forward/backward skips and detach/reattach passed (one recorded SDK impression). This image reports WebView 91.0.4472.114 and Play services 20.18.17. This is an emulator check, not a physical Google-login or live-inventory acceptance test.
- Native Android 11, Google APIs image without Play Store: app built/installed, but the sample SDK returned internal code 0 before any loaded ad reached the app (“Incorrect native ad response. Click actions were not properly specified”). It reports the same WebView and Play services package versions. This failed check is retained. The successful Play-enabled comparison narrows the issue to runtime/ad-delivery differences, but does not prove the exact cause or that all devices without Play Store behave the same way.

Representative regressions cover 30 videos/nine previously watched (10+10+10), 260 videos, sub-millisecond publication ties, mutable viewing history, old/malformed/wrong-scope/wrong-user cursors, GPS drift/dateline radius, blocks/reports/private/deleted/processing content, late publication, stale mobile responses and Trending refresh during pagination, connection recovery/no unsafe auth replay, provider error mapping, offline outbox acknowledgement races, phone screen disposal, three future distinct ad objects, no-fill recovery, two-video cadence, earlier content-count-based prefetch, pointer-down/drag/background interruptions, pagination/fast flings, expiry and teardown, and small-screen/light/dark/large-text layout.

## Physical acceptance before release

Follow-up testing on the owner's USB-connected OnePlus 9 Pro / Android 14 is recorded in [the physical device report](ANDROID_DEVICE_DEV_2026-09-13.md). Google login, real-SMS phone registration/login and phone-only Profile opening passed locally. The same session corrected the age/mute header alignment and a clipped native-ad action button. One physical device on a USB-forwarded development API does not replace the remaining network, inventory and device matrix below.

1. Reproduce the original Android Google/phone flow on the affected installation and on Wi-Fi/mobile data. Verify cold start, provider cancel/return, OTP send/verify and session restoration; inspect new sanitized diagnostics if a failure persists.
2. With real production inventory on India-based test devices, scroll through multiple Feed/Trending pages, forward/backward and rapidly; verify filled opportunities after each two content videos, no blank/bounce and bounded memory. Distinguish no-fill from rendering/placement failures.
3. Verify Nearby and each named geographic scope with precise, approximate and denied location; verify empty-state explanation and picker in both themes.
4. Upload a video, observe processing/ready, open View, refresh Feed and inspect My Videos/Trending eligibility. Do not equate a completed source-file upload with completed processing.

Release requires a new backend image and a new mobile build, followed by owner manual acceptance. This development round adds no database migration. Existing production/staging versions remain unchanged.
