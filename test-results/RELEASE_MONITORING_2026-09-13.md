# Reliability and incident-reporting backend release — 13 September 2026

## Authorization and source

The owner authorized committing and pushing all pending changes and deploying the backend Docker image to staging and production. The three repositories were committed and pushed to `origin/main`, including previously local commits.

| Repository | Implementation commit | Scope |
| --- | --- | --- |
| iadme-backend | `7eee3070f70fa42669b386f05226a6c40f0b8287` | Durable incident persistence, grouped GitHub delivery and additive migration |
| iadme-mobile | `697fac50882e154e89d15d9b5d7b90b649bd0129` | Full-screen native ads, SDK muted autoplay, bounded refill/recovery and durable diagnostics |
| docs | `49449205325c32bcfb4c188f52d737675a072982` | Investigation, backlog, reporting runbook and native test evidence |

This release record and updated rollout statuses are committed in a subsequent docs commit. Infra and website repositories had no pending file changes.

## Docker artifact

- Image: `ghcr.io/forestpondtechnologiesllp/iadme-api:release-2026.09.13-7eee307`.
- Pinned digest: `sha256:7ac44601e2854b473a213b441d51bb6975f76d408de89c87ad042cf481176d5e`.
- Platform: `linux/amd64`, matching the server. OCI revision label is `7eee307`; build provenance records the full backend commit.
- Both API and worker in both environments use this same tag plus digest. Project `.env` files persist the pin. `APP_RELEASE` is `iadme-api@release-2026.09.13-7eee307`. The floating `latest` tag was not changed.

## Migration and rollout

Before each migration, retained a full custom-format PostgreSQL database dump and copies of runtime/Compose configuration and previous container inspection. Verified each dump with `pg_restore --list` and its SHA-256 before applying SQL.

Applied `20260913_add_monitoring_incident_outbox.sql` to each database under its application role. The migration adds `monitoring_incidents`, `monitoring_event_receipts` and indexes; existing user/session tables are untouched. Stop-on-error, a five-second lock timeout and a sixty-second statement timeout were enabled. Migration SHA-256: `de9b0830fa1038025618f195a4ee3b28c9d2bcf2657d86e520b18773466fa33b`.

| Environment | API/worker rollout (UTC) | Rollout (IST) | HTTP checks | Readiness |
| --- | --- | --- | --- | --- |
| Staging | 2026-09-13 14:59:14 | 20:29:14 | 21 passed | Public HTTPS ready/database OK |
| Production | 2026-09-13 15:02:08 | 20:32:08 | 22 passed | Public HTTPS ready/database OK |

Staging completed its controlled incident-delivery and restart verification before production rollout. Staging worker was deliberately restarted at 15:00:36 UTC for that check. Only API/worker containers were recreated; PostgreSQL and Redis were not restarted. All runtime environment values were compared before and after and preserved except `APP_RELEASE`, including signing secrets, `AUTH_SESSION_DAYS=90` and incident-reporting configuration.

## Validation

- TypeScript typecheck and all **31 backend unit tests** passed before committing. The same **31 tests passed against compiled modules in the exact published image**, with container networking disabled and dummy credentials only.
- Earlier disposable PostgreSQL tests verified migration rerun, concurrent UUID deduplication, occurrence aggregation, GitHub failure backoff, concurrent worker delivery, update cooldown and information-only event exclusion. GitHub HTTP was mocked for those failure tests; no deliberate delivery outage was introduced into the deployed environments.
- HTTP smoke checks covered health/readiness, unauthenticated route rejection, malformed authentication requests, invalid refresh, legacy phone-profile response compatibility, complete Feed pagination, Nearby validation, International exclusion, Trending response shape, owner processing status and rejection of another owner. The verification account received all 17 eligible staging videos in pages 10 + 7 and all 22 eligible production videos in pages 10 + 10 + 2, without duplicates. These are account-specific eligible counts, not total videos or an ad-fill measurement.
- Monitoring checks rejected malformed events and server-source impersonation, verified both tables and application privileges, and confirmed server-side GitHub access to the configured mobile repository. Authentication smoke tokens stayed inside the container process; no account, session, OTP, registration email or new video was created.
- One **staging-only synthetic diagnostic** verified HTTP 202 after durable storage, repeated-UUID deduplication, server-owned environment, scheduled worker delivery, counted aggregation and restart recovery. Two acknowledged batches contributed counts 2 and 3; retries did not inflate the total. The worker created [verification issue #23](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/23), then updated the same issue to 5 after restart. Its pending state survived the restart. The test made only this synthetic group's update eligible early instead of waiting the full five-minute issue-update cooldown. An initial assertion ran before the normal 30-second sweep; waiting for the sweep passed. The issue was closed after successful verification and remains clearly labeled as synthetic.
- Final post-startup inspection found both APIs healthy, both workers initialized/running, zero automatic restarts, no error/fatal JSON events, no HTTP 5xx and no monitoring-delivery/retention failures. Redis returned `PONG` for both environments. Staging had one fully delivered synthetic group; production had no received groups at that check. Neither environment had pending groups or delivery retries. This is rollout-time evidence, not a claim about later traffic.

## Backup and rollback

Restricted server directories retain database/configuration backups, migration logs, previous image references and receipts:

- Staging: `/opt/iadme-staging/backups/releases/release-2026.09.13-7eee307/`.
- Production: `/opt/iadme/backups/releases/release-2026.09.13-7eee307/`.

Database dump SHA-256:

- Staging: `f457e05e527b0e8799be44fe8869bd780735354d8995e8f0c145cd9cc30bf9f9`.
- Production: `5f980ddd8a025eccf5adea6f1368d4808705d920c0222622ee0ad8f0a9c4a136`.

Previous image for application rollback: `ghcr.io/forestpondtechnologiesllp/iadme-api:release-2026.09.13-37edc10@sha256:d82e0109bdfa05170fc1b1f7c3902f9721e9f915dc7d2875ea9136f7354c0907`. Restore the saved project/runtime configuration and recreate API/worker with that pin if rollback becomes necessary. Leave the additive monitoring tables intact. Database restoration would replace newer data and is a separate recovery decision. No rollback was needed.

Deployment scripts and private local logs are under `/private/tmp/iadme-release-20260913-monitoring/`; secrets, database dumps and customer data are not committed.

## Mobile handoff

Mobile source is committed and pushed, but **no new AAB/APK/IPA was built or released in this operation**. Installed build 38 cannot acquire the new full-screen native presentation, network recovery or richer diagnostic queue through a backend deployment. Those require the next Android/iOS build and physical acceptance.

Existing clients can submit their supported diagnostic events to the deployed backend. New SDK/media/network fields require the new mobile code. The prior mobile suite passed 231 tests with two existing skips, and Android/iOS native sample-video/static-image checks passed; see [full-screen validation](FULLSCREEN_NATIVE_ADS_2026-09-13.md). Real-phone Wi-Fi/cellular recovery and production ad-serving acceptance remain pending. AdMob fill and portrait/video availability remain external inventory outcomes, and the original intermittent Wi-Fi root cause is not established as repaired by this deployment.
