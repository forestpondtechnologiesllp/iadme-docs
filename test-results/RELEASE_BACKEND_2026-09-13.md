# Backend release and production Android testing — 13 September 2026

## Authorization and revisions

The owner authorized adding AdMob mediation to the backlog, committing docs/mobile/backend, deploying the backend to staging and then production, and testing the latest mobile code on the connected Android against production. Mediation is tracked as IADME-014; it is not implemented or enabled by this release.

| Repository | Commit | Contents |
| --- | --- | --- |
| iadme-backend | `37edc10770f34edd31ef8979e183d0e10cf9fd21` | Stable Feed discovery/pagination, strict geographic scopes and owner-only processing status |
| iadme-mobile | `8fa2f02` | Auth recovery, ad refill/lookahead, pagination, filters, publication watcher, age/mute alignment and native-ad height |
| docs | `2e76bb1` | Investigation, development/device verification and mediation backlog |

These commits were created locally. Git pushes and store uploads were not part of this operation. This release record and subsequent acceptance updates use additional docs commits.

## Backend artifact and rollout

- Image: `ghcr.io/forestpondtechnologiesllp/iadme-api:release-2026.09.13-37edc10`.
- Pinned digest: `sha256:d82e0109bdfa05170fc1b1f7c3902f9721e9f915dc7d2875ea9136f7354c0907`.
- Platform: `linux/amd64`. The image provenance records the full backend commit; its OCI revision label is `37edc10`.
- The same image runs API and worker in both environments. Both project `.env` files persist the tag plus digest, and each runtime `APP_RELEASE` identifies this release. The floating `latest` tag was not changed.
- No database migration was needed or run. All runtime environment values other than `APP_RELEASE` were compared before/after and preserved, including signing secrets and the 90-day session setting.
- Only API/worker containers were recreated; Redis and database services were not restarted.

| Environment | API started (UTC) | API started (IST) | Authenticated Feed verification | HTTP checks |
| --- | --- | --- | --- | --- |
| Staging | 2026-09-13 07:54:40 | 2026-09-13 13:24:40 | 17 eligible videos returned once, pages 10 + 7 | 19 passed |
| Production | 2026-09-13 07:55:34 | 2026-09-13 13:25:34 | 22 eligible videos returned once, pages 10 + 10 + 2 | 20 passed |

Eligibility is specific to the verification account, including its blocked/reported content; these counts are not total database video counts. Each pass was checked against the database's initially eligible IDs. Both public HTTPS `/ready` endpoints returned ready/database OK. Post-startup API/worker inspection found zero restarts and no error/fatal JSON log messages in either environment.

## Verification and recovery

- TypeScript build passed. All 34 backend regressions passed, including 12 PostgreSQL cases using a disposable local database on loopback port 55439.
- Another 22 compatibility/status tests passed against compiled modules inside the exact published image with container networking disabled.
- Deployed checks covered health/readiness, unauthenticated route rejection, malformed auth requests, invalid refresh, legacy `/me` email-string compatibility, Feed coverage/deduplication/exhaustion, missing Nearby location, invalid coordinates, International exclusion, Trending response shape, owner processing status and rejection of another owner.
- Authenticated server smoke checks used short-lived access tokens kept inside the container process. No auth-session record, test account, SMS or registration email was created by these server checks. Feed metrics/cache work can occur as with normal reads. Tokens and customer details are excluded from this report.

Configuration/image snapshots and receipts remain in restricted server directories:

- `/opt/iadme-staging/backups/releases/release-2026.09.13-37edc10/`
- `/opt/iadme/backups/releases/release-2026.09.13-37edc10/`

Rollback target: `ghcr.io/forestpondtechnologiesllp/iadme-api:hotfix-2026.09.12-profile-440e514@sha256:dce10852eb341ef85e5e1d7f0be9c341298d800f18b86d83ba4290f1bcef8c5d`. The rollout restores the original project/runtime configuration and recreates API/worker if its checks fail. No rollback was needed. No database restoration is required for application rollback because this release has no schema change.

## Android production-backend acceptance

Prepared the committed mobile code for the same OnePlus 9 Pro / Android 14 using `APP_ENV=prod`, `APP_RELEASE=iadme-mobile@prod-device-8fa2f02` and Google Play billing configuration. The API is `https://api.iadme.app`; no loopback/LAN API override is supplied.

This is a debug APK for direct device testing, still version 1.0.4+37, not a new store artifact. It uses Google's sample native units under the existing debug-build guard. As in the dev test, a temporary debug-manifest flag suppresses native validator popups; the manifest source was restored after building. The prior production AAB/IPA artifacts are unchanged.

- APK relative to the mobile repository: `apps/iadme_app/build/app/outputs/flutter-apk/iadme-1.0.4-build37-prod-api-test-8fa2f02.apk`.
- SHA-256: `1a6538cf20b8959740498329b656f6f92c5563a39e86367b427e8f2a1a2821fa`.
- Device interaction and login/scrolling acceptance are pending screen unlock. Both previously approved test accounts exist in production.
- The optional choice to test the real ad units on a registered test device remains pending; no AdMob serving or mediation setting has changed.
- Sideloaded debug testing does not establish store-signing behavior, licensed purchases or production ad fill/creative variety.

Private build/deployment/device evidence is under `/private/tmp/iadme-release-20260913/` and is not committed. Earlier development-device evidence is in [the local Android test report](ANDROID_DEVICE_DEV_2026-09-13.md).
