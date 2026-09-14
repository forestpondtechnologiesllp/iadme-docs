# Backend release and mobile build 40 — 14 September 2026

## Authorization and source

The owner authorized committing and pushing all three repositories, publishing the immutable backend image, deploying it to staging and production, and producing Android/iOS build 40 store artifacts.

| Repository | Commit | Contents |
| --- | --- | --- |
| `iadme-backend` | `718eb9d8473c468108b15a70ad24d842f5fac2dd` | Profile-source enrichment, video-orientation handling and backend support for the current reliability work |
| `iadme-mobile` | `c807298a7552599dd882b9512a195a1a71705f15` | Version 1.0.4+40, including functional source `25516cd1d46ab901ab8d68d61d7739fe81356662` |
| `docs` | `cc8553e9295f5b14ed30743d86ca83080d20c64b` plus this release record | Backlog implementation, investigation and release evidence |

## Backend artifact and rollout

- Image: `ghcr.io/forestpondtechnologiesllp/iadme-api:release-2026.09.14-718eb9d`.
- Pinned digest: `sha256:06c8266ada2cc5e4598a7400eda49aa5186a3e76f3330344fb10137dece17c8b`.
- Platform: `linux/amd64`; the same image runs the API and worker in both environments.
- Migration: `database/migrations/20260914_add_profile_enrichment_sources.sql`, applied before the corresponding backend code. Staging required no legacy-row updates; production classified three existing profiles.
- Production backup: `/opt/iadme/backups/releases/release-2026.09.14-718eb9d/database-before.dump` (504,524 bytes), with the previous API/worker inspections and deployment configuration in the same restricted directory.
- Both environment `.env` files persist the tag plus digest. Only API and worker containers were recreated; Redis and PostgreSQL were not restarted.

| Environment | API started (UTC) | Result |
| --- | --- | --- |
| Staging | `2026-09-14T12:24:10Z` | Public `/health` and `/ready` passed; API healthy, worker running, zero restarts |
| Production | `2026-09-14T12:18:57Z` | Public `/health` and `/ready` passed; API healthy, worker running, zero restarts |

Both health responses reported PostgreSQL, Redis and BullMQ connected. A count-only inspection found zero error/fatal application log entries across the four new containers in their first ten minutes.

Rollback uses the pre-release configuration snapshots and previous pinned image in each environment. The new source columns are additive and can remain present during an application rollback; the database dump is retained for disaster recovery.

## Mobile production artifacts

Both artifacts target `https://api.iadme.app`, identify release `iadme-mobile@1.0.4+40`, and request a native-ad opportunity after every two content videos. Neither compiled binary contains the staging API URL.

### Android AAB

- Path: `iadme-mobile/apps/iadme_app/build/app/outputs/bundle/release/iadme-1.0.4-build40-prod.aab`.
- Size: **70,004,049 bytes**.
- SHA-256: `f15c76115a8da81f358414e6b44c703042a52f17290829909f5fd13dbf886d28`.
- Manifest: package `app.iadme.mobile`, version `1.0.4`, build `40`, minimum API 24 and target API 36.
- Architectures: `arm64-v8a`, `armeabi-v7a` and `x86_64`.
- Embedded production configuration: Google Play billing and AdMob native unit `ca-app-pub-2924641977385769/3473122948`.
- Archive integrity and JAR signature verification passed. The upload-certificate SHA-256 is `B9:D1:45:E2:9F:18:B7:EA:3D:19:21:80:E8:01:B6:C4:5F:62:7F:B5:08:43:CD:70:B4:6C:E4:CB:62:A9:61:2F`.

### iOS IPA

- Path: `iadme-mobile/apps/iadme_app/build/ios/ipa/iadme-1.0.4-build40-prod.ipa`.
- Size: **32,916,192 bytes**.
- SHA-256: `b43aef2d9b161739af9f50ac0d6a692e9c07dba3b0d3ed2128cfc65b2e2ca9ed`.
- Bundle: `app.iadme.mobile`, version `1.0.4`, build `40`, minimum iOS 15.0 and arm64.
- Embedded production configuration: StoreKit and AdMob native unit `ca-app-pub-2924641977385769/9941107182`.
- IPA archive integrity and `codesign --verify --deep --strict` passed. Xcode exported it for App Store Connect with the Cloud Managed Apple Distribution identity for team `KU58Q677M3`.
- App.framework UUID `0C7143A3-5943-5FE5-485E-50E6BFCA919E` matches the exported archive.

## Verification and remaining acceptance

- Flutter full suite: 238 passed with two intentional skips.
- Android native ordered-network tests passed on JDK 17.
- Backend: 40 tests passed; TypeScript typecheck and production build passed.
- Flutter analysis found no error or warning; one unchanged style-only `prefer_initializing_formals` info remains outside this release's changes.
- The release build initially exposed a stale generated Android dev-test plugin registrant. Removing that ignored generated file restored the normal release plugin graph; the final AAB contains no test-only registration.
- Final binary inspection confirmed the intended API, billing provider and platform-specific live AdMob units. Candidate artifacts with stale build parameters were overwritten and are not the named release artifacts above.

Store upload, Google Play/App Store processing, live AdMob fill and creative variety, production purchase flows, upload rotation, keyboard behavior, social-profile enrichment and physical Android/iOS acceptance remain owner internal-testing steps.
