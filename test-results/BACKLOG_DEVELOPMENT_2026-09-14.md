# Backlog development verification — 14 September 2026

Development only. No staging/production deployment, database migration, store upload or AdMob/Meta account change was performed in this work.

## Completed locally

### Build 39 tester fixes

- Upload caption keyboard now exposes **Done**, dismisses on background tap and keeps the Upload button reachable without requiring a location change.
- MediaConvert honors MP4/MOV rotation metadata before producing HLS and thumbnails, preventing portrait phone video from being published sideways.
- Native-ad replenishment remains request-bounded after repeated failure and pauses all placement requests for 60 seconds when the Google SDK reports its recent-failure throttle.
- Approved evidence issues were published as [mobile #32](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/32) and [mobile #33](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/33).

### IADME-016 — social profile enrichment

- Google names and profile pictures come only from verified ID-token claims. Google avatar URLs must use HTTPS on `googleusercontent.com`; URL fragments and unsupported hosts are rejected.
- Apple’s first-authorization name remains in the encrypted registration continuation. Apple supplies no provider photo, so the existing editable name and avatar upload remain the fallback.
- Display names now retain Unicode scripts and accents, remove control/format characters and use a 100-code-point limit across registration, social import and profile editing.
- The additive `20260914_add_profile_enrichment_sources.sql` migration records whether each name/photo came from a user, provider, generator or legacy data.
- Provider logins may refresh provider/generated fields and fill a missing legacy avatar. They cannot overwrite a user-edited or user-cleared field. Optional enrichment failure cannot prevent session issuance.

Deployment prerequisite: run the profile-source migration before starting backend code that writes the new source columns.

### IADME-020 — playback-start diagnostics

- Events identify physical/simulator/emulator, debug/profile/release, configured backend environment, platform, network class and validation.
- Timings separate controller queue, previous-controller disposal, initialization, configuration, play request, first positive playback position and first presented Flutter frame.
- Provisional physical release thresholds are 4 seconds on validated Wi-Fi/Ethernet, 5 seconds on validated cellular/VPN and 6 seconds for unvalidated/other networks. Nonphysical or nonrelease observations use 10 seconds and remain information-only.
- Only repeated eligible slow starts can request a GitHub incident; reports are capped at three per app session. Initialization/playback failures retain their independent incident behavior.

### IADME-023 — Android network callback ordering

- Android serializes the `NetworkCapabilities` supplied to `onCapabilitiesChanged` instead of re-querying active capabilities inside the callback.
- A synchronized epoch/revision tracker prevents old-network and post-teardown callbacks from replacing current state.
- Explicit Flutter snapshots remain available and share transport, status, validation and metering fields with callbacks. iOS now supplies the same validation/metering keys.

## Verification

- Flutter full suite: 238 passed, 2 existing intentional skips.
- Flutter changed-file analysis: no issues.
- Android native unit suite: passed, including Wi-Fi/cellular/VPN transitions, validation, rapid loss/recovery, stale network callbacks and engine teardown.
- iOS debug simulator compile: passed (`Runner.app`).
- Backend non-integration suite: 40 passed.
- Backend TypeScript typecheck and production build: passed.
- Git whitespace/error checks: passed in mobile, backend and docs repositories.

## Still requires external state or measured experiments

- **IADME-014 Meta mediation:** requires the Meta property/placement, AdMob mediation mapping, privacy/consent setup, app-ads.txt entries and compatible platform adapter versions before code integration can be verified.
- **IADME-015 Facebook login:** requires the Meta app, Android signing identifiers, iOS URL/redirect configuration, provider permissions and a decision on Limited Login credential verification.
- **IADME-019 HLS caching:** the read-only AWS check found distribution `E34MK64B9IXGL` (`dh3ddpx7cp1qj.cloudfront.net`) has no ordered cache behaviors and uses managed policy `658327ea-f89d-4fab-a63d-7e88639e58f6` for every object. An exact update must preserve the live origin/OAC/WAF config, add distinct versioned HLS manifest/segment behaviors and define reprocessing invalidation before applying long TTLs. No infrastructure code exists in `iadme-infra` yet, so a safe change should first import the live distribution into maintained IaC.
- **IADME-009, IADME-021 and IADME-022:** adaptive ad-cache size, guarded next-video preparation and HLS bitrate/segment changes require physical-device baselines and controlled experiments. Increasing decoder count, inventory requests or lowering quality without those measurements can worsen older Android stability, request throttling, bandwidth or retention.

## AdMob no-fill

The prepared [AdMob support case](ADMOB_SUPPORT_TICKET_2026-09-14.md) now includes the official publisher-support path and the immediate console checklist. Screenshots confirm that both apps are Ready with ad serving enabled and that app-ads.txt is found and verified. The remaining account-side checks are Policy Center and an Ads Activity report that separates requests, matches and impressions.
