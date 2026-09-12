# Build 36 release — 12 September 2026

## Scope and authorization

The owner authorized committing the docs, mobile and backend repositories, building production AAB/IPA artifacts, creating a versioned backend Docker image, and applying migrations and deploying the API/worker to staging followed by production. Mobile upload and internal testing remain with the owner. This authorization supersedes the earlier restriction against backend deployment.

The larger ad-preload proposal remains deferred as **IADME-009**. This release retains the verified cache and cadence from the ad-development pass.

## Source revisions

| Repository | Commit | Contents |
| --- | --- | --- |
| iadme-mobile | `24694b1` | Mobile 1.0.4+36: session restoration, social login, registration details, biometrics, player and ad fixes |
| iadme-backend | `d2190d3118fc08cd81b84733be42cb9895cacdfb` | 90-day sessions, recovery, social continuation, registration notifications and migrations |
| docs | `f606fe5` | Backlog, development verification and existing store-billing records |

These are local commits. No Git repository was pushed during this release operation. This report and the updated backlog are recorded in a subsequent docs commit.

## Production mobile artifacts

Both platforms use `APP_ENV=prod`, `APP_RELEASE=iadme-mobile@1.0.4+36`, ad interval **2**, and the production native-ad/OAuth identifiers verified against the previous build 34 binaries. API endpoint: `https://api.iadme.app`. No dependency versions were changed for the release build.

### Android

- Artifact: `iadme-mobile/apps/iadme_app/build/app/outputs/bundle/release/iadme-1.0.4-build36-prod.aab` (from the iAdMe workspace root).
- Embedded package: `app.iadme.mobile`; version name **1.0.4**; version code **36**.
- Size: **73,182,188 bytes**.
- SHA-256: `8def9f5216edb468bd02e6260cccecf2c18d2b10203aebcce27f24a4043713f1`.
- Signature verification passed. The upload certificate matches build 34; certificate SHA-256: `B9:D1:45:E2:9F:18:B7:EA:3D:19:21:80:E8:01:B6:C4:5F:62:7F:B5:08:43:CD:70:B4:6C:E4:CB:62:A9:61:2F`.
- Compiled app contains the production native-ad unit, Google Web OAuth client, production API URL and new release identifier.
- Built using the existing Android Studio Java 17 runtime and the previously validated JVM workaround (`-XX:-TieredCompilation`).

### iOS

- Artifact: `iadme-mobile/apps/iadme_app/build/ios/ipa/iadme-1.0.4-build36-prod.ipa` (from the iAdMe workspace root).
- Embedded bundle: `app.iadme.mobile`; version **1.0.4**; build **36**; arm64; minimum iOS **15.0**.
- Size: **32,812,474 bytes**.
- SHA-256: `185e548de42c805df9b1279f18f1f2191653618ea353c76905cb04e44e9b5977`.
- `codesign --verify --deep --strict` passed using macOS trust services. Distribution identity: **Apple Distribution: FORESTPOND TECHNOLOGIES LLP (KU58Q677M3)**.
- The embedded App Store profile matches the team/application identifier, allows no debugging and has no development-device or enterprise-distribution list. Profile expiry: **2027-06-09 15:47:27 UTC**.
- Export method: `app-store-connect`, destination `export`, with automatic version/build renumbering disabled so the output remains build 36. No upload was performed.
- Compiled app contains the production native-ad unit, both Google OAuth client IDs, production API URL and new release identifier.

No production Sentry DSN was available or embedded in the previous build 34 binaries. These builds preserve that configuration; app-to-backend diagnostic reporting remains enabled. Store processing, physical-device acceptance, real ad fill/relevance, native social sign-in, biometrics and licensed store-purchase tests remain separate from local artifact verification.

## Backend image

- Registry/tag: `ghcr.io/forestpondtechnologiesllp/iadme-api:release-2026.09.12-d2190d3`.
- Immutable digest: `sha256:ec3e648379e953b5255d40c4a27410cc158e6500fd22788c298839e25c191113`.
- Platform: **linux/amd64**, matching the deployment host.
- OCI revision label: `d2190d3118fc08cd81b84733be42cb9895cacdfb`.
- Runtime checked in both deployments: **Node v22.23.2**.
- The same image serves API and worker in both environments. The floating `latest` tag was not changed.
- Both stacks persist `IADME_API_TAG=release-2026.09.12-d2190d3@sha256:ec3e648379e953b5255d40c4a27410cc158e6500fd22788c298839e25c191113` in their project `.env`, pinning the tag and digest.

## Migrations and deployment

Both SQL files were applied with stop-on-error enabled, a five-second lock timeout and a sixty-second statement timeout, in one transaction per database. DDL executed under the corresponding application role; the resulting registration table is accessible to the application.

| Migration | SHA-256 |
| --- | --- |
| `20260912_registration_details.sql` | `51bbd4c9a19039c8c2236b488895a54727632032dc8dc8e667c4338a999e648b` |
| `20260912_refresh_recovery.sql` | `c931869df13bfc1bf2bd88539297bff595181da355acc82d76bbc90c1991cf8a` |

| Environment | Database | Migrations committed (UTC) | New API/worker started (UTC) | Initial checks passed (UTC) |
| --- | --- | --- | --- | --- |
| Staging | `iadme_staging` | 15:01:11 | 15:02:16 | 15:02:22 |
| Production | `iadme_prod` | 15:04:24 | 15:04:59 | 15:05:05 |

Production started at **20:34:59 IST** on 12 September 2026.

- `AUTH_SESSION_DAYS=90` is set explicitly for each API and worker.
- `APP_RELEASE=iadme-api@release-2026.09.12-d2190d3` now identifies the actual backend release.
- Existing JWT signing secrets were compared before/after and preserved. Access-token expiry remains **15 minutes**. The older `JWT_REFRESH_EXPIRY=30d` setting remains for configuration compatibility; the new code uses `AUTH_SESSION_DAYS` for refresh issuance.
- Only API and worker containers were recreated. Redis and the database were not restarted.

## Verification evidence

- Backend TypeScript build passed immediately before committing.
- All **9** backend unit tests passed locally and again against compiled code inside the exact release image, with networking disabled for the container test.
- Each deployed environment passed **11 HTTP checks**: health/readiness 200; validation errors 400 for empty login/Google/Apple/social-registration requests; invalid refresh 401; authenticated registration-details/feed/trending routes 401 without credentials.
- Both public HTTPS `/ready` endpoints returned `{"status":"ready","database":"ok"}`.
- Each deployed application role can access all three new recovery columns and select/insert/update/delete registration details.
- The actual deployed token-generation function produced a refresh token with an exact **90-day** lifetime in each environment. The session INSERT was intercepted for this check: **no user or session was created, no token was printed or used, and no test email was sent**.
- Both Redis instances returned `PONG`; both registration-notification workers recorded a fresh heartbeat after their new container started.
- Both workers logged successful initialization. Post-startup checks found **zero API/worker restarts and no error/fatal log entries** in either environment.
- Earlier Flutter regression/native-ad evidence remains in [ADS_FIXES_2026-09-12.md](ADS_FIXES_2026-09-12.md) and [TESTER_FIXES_2026-09-12.md](TESTER_FIXES_2026-09-12.md).

The deployment checks validate routing, configuration and service/database health. They do not claim a completed real-account Google/Apple login or physical-device acceptance. The owner will perform those through the store internal-testing builds.

## Backups and recovery

Verified PostgreSQL custom-format dumps, original runtime/Compose configuration, previous image references, migration receipts and deployment receipts are retained on the server:

- Staging: `/opt/iadme-staging/backups/releases/release-2026.09.12-d2190d3/`.
- Production: `/opt/iadme/backups/releases/release-2026.09.12-d2190d3/`.

Dump checksums:

- Staging `database.dump`: `aa6098eea035b06aad49fc91b88b71dfe0f3a25d66cfc3572eea427a59bcab13`.
- Production `database.dump`: `b05f9d3e355402f58a6cca23a833eed1fa73dabc40a80477bf478b0c3f6d17c7`.

`pg_restore --list` successfully read each dump, and its checksum was rechecked before migration. Database/configuration backups remain on the server with restricted permissions; secrets and customer data were not copied into Git.

Previous API/worker image in both environments: `ghcr.io/forestpondtechnologiesllp/iadme-api:prod-2026.09.09-registration-growth-v1`, image ID `sha256:846f70754e91c371db853114f816483583866629723be35f0ff676bb6aaa0306`.

If an application rollback is needed, restore the matching runtime configuration and explicitly select that retained image for both API and worker. Leave the additive tables/columns intact. Once build 36 is installed, any backend recovery must continue supporting its new endpoints; the old image alone does not provide them. Database restoration is a separate recovery decision because it would replace data written after the backup. No rollback was required.

## Internal-testing handoff

- [ ] Owner uploads the signed AAB to Google Play internal testing and the signed IPA to TestFlight.
- [ ] Confirm upgrades from the installed build preserve login; test new Google/Apple registration and legal-consent completion.
- [ ] Test feed/trending forward/backward swipes, rapid scrolling, unavailable ads, tab switches, poor network and background/resume on affected older Androids and current Android/iOS devices.
- [ ] Confirm production native-ad delivery, India relevance, one skippable opportunity every two content videos and actual filled impressions.
- [ ] Confirm registration email device details, permission-dependent area details, biometrics, upload privacy copy and month labels.
- [ ] Confirm licensed Google Play/StoreKit purchases and premium playback remain functional.
- [ ] Authorize public mobile rollout only after internal acceptance.

Local build/deployment logs are under `/private/tmp/iadme-release-20260912-build36/`. No mobile artifact was uploaded or publicly released by Codex.
