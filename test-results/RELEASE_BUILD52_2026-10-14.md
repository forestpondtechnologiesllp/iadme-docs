# Release 1.0.4 (52) — 14 October 2026

Backend deployment is complete on staging and production. Signed AAB and App Store IPA for **1.0.4 (52)** are built and verified. Mobile packages are for owner-managed uploads. Nothing has been submitted to Google Play or App Store Connect by this release task.

## Source and scope

- Mobile source: `54c1d5994959d216c72784619505ae28147a40e0`, version `1.0.4+52`, normal production target `lib/main.dart`.
- Backend image source: `9736dde5f709d3197a038f4e52ea35421d8d22d9`. Documentation-only follow-up: `ffcf106`.
- Image: `ghcr.io/forestpondtechnologiesllp/iadme-api:release-2026.10.14-build52-9736dde`.
- Immutable digest: `sha256:8a8dad3b14a37be0020dd9c219e396b327ecefb75f878e0473c2f1a194784a3d`.
- Backend release label: `iadme-api@2026.10.14-build52-9736dde`.

Mobile changes since build 51 fix listing field correction highlights and keyboard dismissal, cache visited private listing photos across swipes/signed-link renewal, recover compact Inbox/Comments ads after temporary failures, improve cold HLS startup and bound duplicate-request waiting, prepare both Feed and Trending queues, and add Refresh above Report a bug with coalescing/cooldown. Active next-two preparation retains priority, with one player, one speculative downloader and the existing memory/disk/data guards. OS background execution and instant switching are not guaranteed.

The backend adds shared Redis per-user/action/item limits, public IP and OTP destination limits, bounded comments pagination, `429` retry guidance and production fail-closed behavior when Redis is unavailable. The legacy unauthenticated payment-credit endpoint is removed; payment-provider-verified fulfillment remains. Detailed budgets are in `iadme-backend/docs/api-abuse-controls-2026-10-14.md`.

## Deployment and database

The owner explicitly approved production after staging passes, with Meta verification left pending. Staging passed before production rollout. Both environments run the same pinned image for API, worker and affiliate worker. Production remains **5 Stars per photo post**, staging 10; no encoder, Lambda, retention, public post or customer wallet change was made by this release.

There are **no new database migrations**. All 13 existing release migration filenames/checksums matched before deployment, and the count remains 13 afterward. Private database dumps and configuration copies are retained on the server:

- Staging: `/opt/iadme-staging/backups/release-20261014-build52-v1/`.
- Production: `/opt/iadme/backups/release-20261014-build52-v1/`.

Both dump catalogs passed `pg_restore --list`. Local evidence contains backup receipts and hashes, not database contents or credentials. The rollout helper restores the previous application/configuration on readiness failure. For a later application rollback, use `python3 /tmp/iadme-release52.py staging rollback` or `prod rollback` on the server, then rerun health and role checks. The previous pinned image is `release-2026.10.12-layout-photo-d1bddc6@sha256:bc26f0c33c965b8caec2df0d88c5302c0a6780a6bd63fdabadd78f0e1f192856`. Database restoration is not part of application rollback.

## Verification

- Backend: 43 tests and TypeScript checks passed; CI typecheck/build passed for `9736dde`.
- Mobile: 148 focused tests and analysis passed. The 120 responsive layout cases passed across ten small/large iOS/Android viewports, four text scales through 3×, notch/island/navigation and side-safe-area fixtures. The matrix was part of an early mixed run whose six unrelated failures were fixed and passed in the final focused suite; it is not represented as an overall passing initial run.
- Native simulator: iPhone 17 / iOS 26.5 software-keyboard input showed no correction patch; tapping outside dismissed the keyboard. A real Google compact sample creative rendered. Tests used a separate diagnostic identity and synthetic form data, with no live-ad clicks.
- Earlier authorized physical profile evidence: two cold public-video first frames at 1,106 ms and 344 ms after the bounded-duplicate fix. This is two samples, not a store-release benchmark or proof for every device/network. A later screenshot rerun lost its VM connection and remains recorded as incomplete. Current follow-up used only the simulator, as instructed.
- Staging and production: public HTTPS `/health` and `/ready` passed; all three roles run the expected image/release/price with zero restarts, including follow-up stability checks.
- Smoke tests: one signed random nonexistent subject received four authentication failures, then two `429` responses with valid retry guidance; another subject had an independent quota. Removed credit route returned `404`; store verification remained authenticated; public Star packs remained available. Only expiring test quota counters were created, with no customer writes.

## Ads: confirmed cause and remaining check

Build-51 production logs show three Google native SDK impressions, followed by repeated SDK code 1, `No ad to show`, with consent granted. Separately, Inbox/Comments stopped after a failed first request; build 52 adds bounded, paced retries. The two-video Feed/Trending cadence remains, but an available creative is required. A client fix cannot guarantee provider inventory or that both networks win auctions.

Meta app approval/verification and placement mappings were checked, but account diagnostics conflict: waiting-for-bids cards and an invalid-bundle debugger result do not establish current live fill. Ad Inspector cannot complete on the current connection: its Google hostname resolves to Airtel's `restrictedspam.rpz.airtelspam.com`, causing a TLS hostname mismatch. The ordinary Google ad-delivery endpoint validates. Airtel's exact classification reason is unavailable. This explains the blocked Inspector check; it does not establish why build 51 had no-fill responses. No DNS, proxy or trust settings were changed. The owner accepted builds/deployment with **Meta live serving unverified**.

## Artifacts and resource limits

Release folder: `artifacts/releases/1.0.4-build52/`. Both packages passed signing, version/build, production configuration and compiled release-marker checks. Android bundle validation, expected signing certificate and all 12 native 64-bit libraries’ 16 KB alignment passed. iOS App Store distribution profile, strict signature, production push entitlements and private Facebook native configuration passed. Live production ad IDs, interval 2 and all five video feature switches were verified; diagnostic/test overrides are off. Select the exact versioned artifacts from this folder for manual upload, not a wildcard from Flutter's build directories.

Heavy local tasks ran sequentially: Android with one Gradle worker, Docker with a one-CPU cap, Xcode with one compile worker, all low priority where supported. The simulator and release builder were stopped before the next heavy job. Dev API, worker, PostgreSQL and Redis were kept running. Temporary build settings are restored by the wrappers. macOS reported no thermal/performance warning during sampled checks; no temperature measurement or continuous monitoring is claimed.

Evidence: `artifacts/releases/1.0.4-build52/evidence/`, with detailed investigation and native screenshots in `artifacts/diagnostics/ios-build51-20261014/`. See [investigation record](BUILD51_FOLLOWUP_2026-10-14.md) for the original findings and incomplete attempts.

## Verified artifact hashes

| Artifact | Bytes | SHA-256 |
| --- | ---: | --- |
| `iadme-android-1.0.4+52.aab` | 80983269 | `d7533f368c5e32ca75bc93c57602d730843d512d6d5afc1a2a7e6c4ebf373eff` |
| `iadme-ios-1.0.4+52.ipa` | 39473666 | `88ae252cd27900c7143d2db76f882ea1b71c58b567b03e05fc3f6c8bd83131ee` |

The exact files, `SHA256SUMS`, release notes and verification report are in the versioned release folder. Temporary Gradle/Xcode settings were restored, and the mobile source tree remains clean at the artifact commit.
