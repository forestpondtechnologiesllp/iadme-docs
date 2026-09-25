# Backend rollout and mobile 1.0.4 (45) — 21 September 2026

## Outcome

The owner requested a new production Docker image and, because the fixes require
mobile changes, production AAB and IPA files. Staging and production API/worker
were deployed successfully. Both signed mobile artifacts were created and
verified locally; neither was uploaded to Google Play or App Store Connect.
No Git source commit/push, historical issue cleanup, or firewall change occurred.

## Source and image

Backend base commit: `6724775`; mobile base commit: `2cea5b2`. Both include the
uncommitted follow-up changes, not merely those base revisions. Mobile pubspec
is now `1.0.4+45`. Source manifests, exact diffs and changed-source archives are
saved in `artifacts/releases/1.0.4-build45/evidence/`.

Image: `ghcr.io/forestpondtechnologiesllp/iadme-api:release-2026.09.21-build45-followup`

Digest: `sha256:f831e0b065fed67d836afdd796b34aad7a750cb50309f0ef834cd0f2414da2c7`

Platform: `linux/amd64`. The owner explicitly approved the registry upload.
Both API and worker use the digest-pinned image in each environment.

The release adds meaningful method/route API failure titles and consolidates
recognized AdMob availability events across platforms/releases/placements.
Configuration, unknown SDK and media/rendering problems remain separate.
Historical tickets/pending legacy incidents are not bulk closed or reconciled.
Build 45 separates API endpoints before mobile batching; legacy mixed batches
cannot be accurately split retroactively.

## Migration and deployment

`database/migrations/20260921_monitoring_signal_counts.sql` was applied before
the new API/worker in staging, then production. It adds a non-null JSONB column
with an empty-object default. Old images remain compatible with this schema.

- Staging API started: `2026-09-21T16:38:19.166842789Z`.
- Production API started: `2026-09-21T16:40:00.51772111Z` (22:10 IST).
- Runtime settings were preserved except the backend `APP_RELEASE` identifier.
- Redis containers were not recreated. Development API/worker stayed running.
- No synthetic monitoring events or test GitHub issues were created on the server.

Public `/health` and `/ready`, protected-endpoint unauthenticated boundaries,
worker initialization, schema and compiled monitoring logic passed in both
environments. At `2026-09-21T16:42:59Z`, all four containers were running with
zero restarts and zero structured error/fatal logs since their rollout.

The Tailscale SSH route was intermittently unavailable/slow. The public fallback
was restricted to an older client IP. No firewall rules were changed: the
existing Tailscale route recovered, and a small checksum-verified change bundle
was transferred instead of waiting on a full source archive. The full source
archive remains local under `artifacts/deployments/2026-09-21-followup/`.

Local-development note: its database lacks the older `monitoring_incidents`
table. An attempt to apply only this additive migration failed and rolled back
without schema/data changes. No older local migrations were automatically run.
The migration was separately integration-tested on a disposable local database
during the preceding fix work; staging/production have the prerequisite table.

## Backups and rollback

Restricted server backup directories contain the previous project/runtime
environment, Compose configuration, container metadata, database dump, migration,
source-change bundle, rollback image override, private logs and completion receipt:

- staging: `/opt/iadme-staging/backups/releases/release-2026.09.21-build45-followup/`
- prod: `/opt/iadme/backups/releases/release-2026.09.21-build45-followup/`

Database backup SHA-256 values:

- staging: `842c85955fa8fbbdfe6c7849c4e31f24ef5284ce4daac3d1e5bee423c159b35b`
- prod: `3e19aa0e38bc7efbd400c6d57d80e5b1ae6f482d56bd7fbcf45855f95fc5dd5d`

Previous image pin for both environments:

`release-2026.09.21-6724775@sha256:017ed4910b35fedffd072677ac908148292275ed758eaf2b175516910af8ea97`

Image rollback restores the saved project/runtime environment and recreates only
API/worker with `rollback-images.json`. Leave the additive column in place; do
not restore a full database dump over subsequent user writes. The rollout script
provided automatic image rollback on failed post-deployment checks; neither
environment required it. Private backups were not downloaded.

## Mobile artifacts and configuration

Directory: `artifacts/releases/1.0.4-build45/`

- `iadme-android-1.0.4+45.aab` — Android package `app.iadme.mobile`.
- `iadme-ios-1.0.4+45.ipa` — iOS bundle `app.iadme.mobile`, App Store distribution,
  deployment target iOS 15.0, signing team `KU58Q677M3`.

Both used the checked production launcher and explicit private Facebook overlay.
Facebook login, live AdMob units, every-two-reel placement, next-two fast path,
replay cache and background preparation remain enabled. Affiliate video overlays
and affiliate comment ads remain disabled. No new Meta mediation adapter was added.

Android SHA-256: `ef7725a739f146aa91d4999e6d2f1db35454074c6031f72ad7c217eb8aafaf6d`

iOS SHA-256: `c3db5f1cde9eb602d99f453fab586fb5c485e61372b413265d3f83cfce74cfdb`

## Verification and remaining acceptance

- Backend TypeScript typecheck/build and 14 monitoring unit tests passed.
- Exact Docker image passed compiled-policy checks with networking disabled.
- 78 focused final mobile regressions passed; preceding full suite passed 387
  tests with four existing skips. Six native Android tests and iOS Simulator
  compilation also passed during the fix work.
- Android bundletool validation, build/package checks and JAR signature passed;
  upload-key certificate matches build 44. Self-signed upload-certificate warnings
  are retained in the signature evidence.
- IPA passed strict/deep signature validation, version/package, App Store
  provisioning, production push and native Facebook configuration checks.
- Both compiled binaries contain the build-45 release identifier. Actual iOS
  compiled defines were checked and recorded with the Client Token redacted.

Retest location permission/Settings recovery and Feed/Trending scrolling across
ad boundaries on physical devices. The reported whole-app iOS freeze remains
unreproduced/root-cause unconfirmed; these builds do not prove it is resolved.
Facebook Limited Login is still intentional. Genuine ad no-fill remains an
inventory/serving condition, not something this diagnostic consolidation repairs.

Local operational evidence: `artifacts/deployments/2026-09-21-followup/`.
Mobile build, signature and source evidence: `artifacts/releases/1.0.4-build45/evidence/`.
