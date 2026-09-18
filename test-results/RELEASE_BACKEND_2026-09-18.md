# Backend deployment and background-reel handoff — September 18, 2026

## Scope and source

The owner requested commit/push and Docker deployment to staging and production,
with production simulator acceptance before any new AAB/IPA build.

- Mobile: `8ee8dd8d167b29101e40aa1795ebf9d49ae32df1`, pushed to `iadme-mobile/main`.
  Includes the saved build profiles, prepared-first-frame playback improvements,
  native diagnostics, bounded background video preparation, fresh Feed/Trending
  refresh and upload-ready snackbar dismissal.
- Backend: unchanged `ef63cf53d78d192386e473e70cae7873abbc55b5`, already on
  `iadme-backend/main`. A fresh image was built from this clean revision.
- No database migration or catalog re-encoding was performed. Runtime settings
  were preserved except for `APP_RELEASE`. Redis was not recreated and the local
  development services were left running.
- No new AAB/IPA was created or uploaded. Pubspec remains `1.0.4+43`; archived
  TestFlight build 43 predates the background feature and remains unchanged.

## Image and rollout

Image: `ghcr.io/forestpondtechnologiesllp/iadme-api:release-2026.09.18-ef63cf5`.

Digest: `sha256:3736cffbba8af6c9725fee4986c05629b8a2e4fa4aa76de56fa3aaad8881ff20`.

Platform: `linux/amd64`; Node `v22.23.2`. Both API and worker use this same
immutable digest in each environment. Staging passed before production started.

| Environment | API started (UTC) | Worker started (UTC) | Upload acceleration |
| --- | --- | --- | --- |
| staging | `2026-09-18T07:40:32.69163166Z` | `2026-09-18T07:40:32.706447881Z` | Off |
| prod | `2026-09-18T07:41:40.088616159Z` | `2026-09-18T07:41:40.083778964Z` | On |

Each deployment passed 11 HTTP checks (health, readiness, authentication
boundaries and invalid auth input), worker initialization and public HTTPS
health/readiness. Real SDK signing confirmed that new production upload URLs
use S3 acceleration and staging uses regional S3. No object was uploaded by
these checks; download signing remains regional.

The exact Docker image passed all **47 backend tests**, with networking disabled
and dummy credentials. The production simulator launcher preflight passed.
At `2026-09-18T07:42:33Z`, all four containers had zero restarts and zero
structured error/fatal entries since deployment. Each API had nine expected
400/401 warnings from the negative smoke probes. See the
[final health evidence](assets/background-reels-2026-09-18/deployment-health.json).

## Mobile validation already completed

- Flutter app suite: **329 passed, two existing skips**.
- Final focused cache/pagination regressions: **19 passed**.
- iOS simulator and Android API 30 emulator native checks passed: scheduler
  callback, resource guards, independent SQLite connections/competing writes,
  and saved 720p opening restoration with **zero CDN bytes**.
- Analyzer: no errors or warnings; one pre-existing style-only info.
- User reported good Build 43 playback, upload speed and progress behavior.
  This is separate from acceptance of the new background follow-up.

Background preparation fetches fresh Feed and Trending selections when allowed,
prepares up to eight public openings, and preserves the next-two priority path.
The approximately 200 MB allowance is disk storage. Extra preparation is Wi-Fi
only, bounded at 50 MB/day; normal viewing and next-two preparation are separate.
OS scheduling and access-token validity govern background refresh. Resume falls
back to normal fresh requests while keeping the current list visible.

See the [mobile implementation notes](https://github.com/forestpondtechnologiesllp/iadme-mobile/blob/8ee8dd8d167b29101e40aa1795ebf9d49ae32df1/docs/background-reels-2026-09-18.md).

## Backups and rollback

Restricted server backup directories contain the prior project/runtime environment,
Compose configuration, container metadata, per-service rollback image override,
database dump, source archive and verification receipts. Credentials remain on the
server and are not included in this repository.

- staging: `/opt/iadme-staging/backups/releases/release-2026.09.18-ef63cf5/`; database dump SHA-256 `96918f8e946669ce128cffb99edae7d4f166d31b6019ecb5f3cbd1e77aa945c4`.
- prod: `/opt/iadme/backups/releases/release-2026.09.18-ef63cf5/`; database dump SHA-256 `1e6de3e6de6227fee93e651e208dcc79b8d5cd42a77455228c0c56bd043aaf33`.

Previous image pin: `release-2026.09.16-ef63cf5@sha256:0add7f9225965790a8b3179284b6aff79f023286243983da9cab3bec0d7909bf`.
Source archive SHA-256: `5dcce2006d088b71c953150ca222777816d4a896a7506a869c3aab79eff10d37`.

Rollback restores the saved environment files and recreates only API/worker using
its saved Compose configuration plus `rollback-images.json`. A schema rollback
is unnecessary. The deployment script would automatically do this on failed
post-deployment checks; both environments passed.

## Production simulator acceptance

```sh
cd /Users/saisrikrishnakumaradavikolanu/Projects/iAdMe/iadme-mobile/apps/iadme_app
xcrun simctl bootstatus 923F0814-1A02-4E4C-9DAE-C0F64E8A551F -b
open -a Simulator
dart tool/mobile.dart prod run -d 923F0814-1A02-4E4C-9DAE-C0F64E8A551F
```

Uses production API/Google login, saved video switches and test ads in simulator
debug mode. Stop an existing Flutter run and fully relaunch so native/config
changes are included. No shell exports are needed.

Check new API results after minimizing/reopening both tabs, forward/back swipes,
ads, video storage controls and upload-ready notice dismissal (eight seconds,
close button and View). Use a second account's newly published eligible content
for freshness testing; the backend determines Feed/Trending ordering. Physical
release-device background scheduling and playback remain separate acceptance.
Create a new build number for the eventual store artifacts after this acceptance.

The [tracked build/deploy guide](../runbooks/IADME_BUILD_DEPLOY_COMMANDS.md) mirrors
the workspace root reference file for future tasks. Local detailed evidence is
in `artifacts/deployments/2026-09-18/` and
`artifacts/diagnostics/background-reels-20260918/`.
