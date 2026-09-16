# Backend release and mobile build 41 — 16 September 2026

## Source and scope

The owner authorized committing and pushing the pending changes, publishing a
Docker image, deploying staging and production, and creating signed AAB/IPA
artifacts. Mobile store submission is a separate step.

| Repository | Commit | Contents |
| --- | --- | --- |
| `iadme-backend` | `ef63cf53d78d192386e473e70cae7873abbc55b5` | Configurable upload acceleration and guarded, versioned dev reel re-encoding tool |
| `iadme-mobile` | `5af9c201103a83051f3cd2503ba931b85c4f8b6f` | Reel preparation/quality, first-frame handoff, bounded replay cache, upload progress and production build 1.0.4+41 |
| `docs` | `8cefd02bb23ecef33bd3f7b1df183f78fd4c62ab` plus this record | Release notes, tester checklist and completed release evidence |

All three source commits were pushed to `main`. Infrastructure and website
repositories had no pending changes.

## Backend image and deployment

- Image: `ghcr.io/forestpondtechnologiesllp/iadme-api:release-2026.09.16-ef63cf5`.
- Digest: `sha256:0add7f9225965790a8b3179284b6aff79f023286243983da9cab3bec0d7909bf`.
- Platform: `linux/amd64`; Node `v22.23.2`.
- API and worker in both environments run this same pinned image.
- No database migration or catalog re-encoding was performed.
- Runtime configuration was preserved except for the release identifier.
- Redis and PostgreSQL were not restarted. Local dev API and worker were left running.

| Environment | API started (UTC) | Worker started (UTC) | Upload acceleration |
| --- | --- | --- | --- |
| Staging | `2026-09-16T10:45:57Z` | `2026-09-16T10:45:57Z` | Off; regional S3 upload URL verified |
| Production | `2026-09-16T10:47:12Z` | `2026-09-16T10:47:12Z` | On; S3 accelerated upload URL verified |

Each deployment passed 11 HTTP smoke checks, covering readiness, health,
authentication boundaries and invalid authentication input. Real SDK signing
with the running service credentials confirmed upload routing; download URLs
remain regional. Public `/health` and `/ready` returned HTTP 200 in both
environments.

At `2026-09-16T10:53:25Z`, both APIs were healthy, both workers were running,
and all four containers had zero restarts and zero structured error/fatal log
entries since startup. Each API had nine expected warning entries from the
negative smoke probes: six HTTP 401 and three HTTP 400 responses.

### Backups and rollback

Restricted release backup directories on the server:

- Production: `/opt/iadme/backups/releases/release-2026.09.16-ef63cf5/`.
- Staging: `/opt/iadme-staging/backups/releases/release-2026.09.16-ef63cf5/`.

Each contains the pre-release database dump, runtime/project environment files,
Compose configuration, previous container metadata, per-service rollback images,
source archive, smoke evidence and deployment receipt.

| Database dump | SHA-256 |
| --- | --- |
| Production | `bdddd6568953453f5f36618ef2381ce375e0260a9f14ac444d3cd6341f9be2ea` |
| Staging | `9133699b1aacfd84b4dc6861aad4976908af023f4b955f463c51e23abada1093` |

Application rollback restores the saved environment files and recreates only
API/worker using the saved Compose file plus `rollback-images.json`. Production
previously used different API and worker images, so retain the per-service
override. No schema rollback is required for this release.

The independent production upload-acceleration switch can be turned off without
a mobile update:

```sh
ssh iadme-prod 'docker exec iadme-prod-api node dist/main/scripts/set-upload-acceleration.js off --environment=production'
```

It affects newly issued upload URLs within 30 seconds; outstanding uploads can
finish. Keep the bucket acceleration capability enabled while those uploads finish.

## Mobile artifacts

Both signed artifacts are **1.0.4 (41)**, bundle/package `app.iadme.mobile`,
and target `https://api.iadme.app`. Compiled binaries contain the release
identifier `iadme-mobile@1.0.4+41`, production Google sign-in configuration and
the correct platform-specific live native-ad unit. Neither contains the staging
API URL. Ad opportunity interval is two content videos; preparation/replay
caching is enabled through `config/prod-release.json`.

Files are retained under the workspace directory
`artifacts/releases/1.0.4-build41/`:

| Artifact | File | Bytes | SHA-256 |
| --- | --- | --- | --- |
| Android | `iadme-1.0.4-build41-prod.aab` | 70,616,265 | `06e19f5fb3714caef32c2bb4f341642f0b1121b79dd652ce109dfd0f5766b07a` |
| iOS | `iadme-1.0.4-build41-prod.ipa` | 33,067,364 | `c5df45e64ff5ff101a552bf6f8a63b7ad7c241ac660c240bffe92e38284014ff` |

### Android verification

- Manifest package/version/build, ZIP integrity and JAR signature verified.
- Upload certificate SHA-256:
  `B9:D1:45:E2:9F:18:B7:EA:3D:19:21:80:E8:01:B6:C4:5F:62:7F:B5:08:43:CD:70:B4:6C:E4:CB:62:A9:61:2F`.
- Includes `arm64-v8a`, `armeabi-v7a` and `x86_64`.
- Google Play billing and Android AdMob native unit
  `ca-app-pub-2924641977385769/3473122948` verified in the binary.
- Built with Java 17 and a bounded 3 GiB Gradle heap.

### iOS verification

- Exported for App Store Connect with the Apple Distribution identity for team
  `KU58Q677M3`; provisioning profile is distribution-only, with debugging disabled.
- Minimum iOS 15.0, arm64; bundle/version/build and ZIP integrity verified.
- `codesign --verify --deep --strict` passed for the exported app.
- StoreKit billing and iOS AdMob native unit
  `ca-app-pub-2924641977385769/9941107182` verified in the binary.
- App.framework UUID `0C7143A3-9714-88F0-485E-50E6ACBF4F47` matches the archive
  and its dSYM. `Runner.xcarchive`, including debug symbols, is preserved beside
  the artifacts.
- Automatic export build-number management was disabled to preserve build 41.

Both artifacts were created locally; neither was submitted to a store.
Build 40 artifacts/archive were preserved separately. Release builds used Flutter
3.44.0 and Xcode 26.6. The Android build directory contained an older named AAB,
so verification explicitly selected the fresh `app-release.aab` and checked its
manifest before naming the build 41 delivery.

## Validation

- Full Flutter suite: **282 passed, two existing skips**.
- Flutter analysis: no errors or warnings; one unchanged style-only
  `prefer_initializing_formals` info in `silent_sync_coordinator.dart:26`.
- Android native video-plugin tests: **89 passed** earlier in the same session,
  with no subsequent plugin source changes.
- Backend: **47 passed**, TypeScript typecheck and production build passed.
- Exact release Docker image: **47 passed** with networking disabled and dummy
  test configuration.
- [Backend CI for the released commit](https://github.com/forestpondtechnologiesllp/iadme-backend/actions/runs/35086282568): passed.
- Initial accelerated routing was also exercised with an 8,192-byte PUT and
  byte-identical regional GET before this full rollout. The temporary object was
  removed. This confirms correctness, not an upload-speed benchmark.

Build logs and machine-readable verification receipts are retained locally in
`/private/tmp/iadme-release-20260916-build41/`. The ignored Android dev-test plugin
registrant was regenerated before the release build; no application change was
needed. Generated Android intermediates were removed after preserving and
verifying the AAB to free space for the iOS build.

## Release notes and acceptance

See [build 41 release notes and tester checklist](../test-plans/BUILD41_ACCEPTANCE_2026-09-16.md).

The main improvements are preparation of the next two eligible reels across ads,
network-informed HD startup, display of the real first frame, bounded reuse of
watched segments, visible upload percentages and production-only upload acceleration.

Physical Android/iOS acceptance, store processing, live ad serving and store test
purchases remain to be checked through internal testing/TestFlight. Cache misses
and fast swipes can still need network time. Uploads remain single PUT transfers;
this release does not add resumable/background uploads or automatic compression.
