# iAdMe Build, Migration, Deployment, and Mobile Commands

This file is stored in `/Users/saisrikrishnakumaradavikolanu/Projects/iAdMe`,
outside the `iadme-backend`, `iadme-mobile`, `iadme-website`,
`iadme-infra`, and `docs` Git repositories.
A tracked copy is maintained at `docs/runbooks/IADME_BUILD_DEPLOY_COMMANDS.md`;
keep both copies synchronized.

Backend tags, digests and migration filenames below are historical release
examples: replace them for the intended backend release. Mobile-only builds
start at section 9 and do not require backend redeployment. Mobile configuration
is saved in the app's `config/` directory; no Dart-define exports are required.

## Latest backend release — September 18, 2026

Image: `ghcr.io/forestpondtechnologiesllp/iadme-api:release-2026.09.18-ef63cf5`.
Digest: `sha256:3736cffbba8af6c9725fee4986c05629b8a2e4fa4aa76de56fa3aaad8881ff20`.
Backend source remains `ef63cf5`; the mobile follow-up is committed as `8ee8dd8`.
No schema migration is part of this rollout. Deployment results and rollback
references are recorded in `docs/test-results/RELEASE_BACKEND_2026-09-18.md`.
Use the production simulator command in section 12 to test the mobile follow-up
before requesting new store artifacts. The existing TestFlight build 43 remains
unchanged; these mobile changes require a fresh build/relaunch.

## 1. Release parameters — Mac

```bash
IADME_IMAGE_TAG="release-2026.09.14-718eb9d"
IADME_IMAGE_DIGEST="sha256:06c8266ada2cc5e4598a7400eda49aa5186a3e76f3330344fb10137dece17c8b"
IADME_IMAGE_PIN="${IADME_IMAGE_TAG}@${IADME_IMAGE_DIGEST}"
MIGRATION_FILE="database/migrations/20260914_add_profile_enrichment_sources.sql"
```

Use a new immutable `IADME_IMAGE_TAG` for every backend release. Mobile version
and build number come from `apps/iadme_app/pubspec.yaml` (section 16).

## 2. Authenticate Docker with GHCR — Mac, when needed

This syntax is for the default macOS `zsh` shell.

```bash
read -s "IADME_GHCR_PAT?GHCR token: "
echo

print -rn -- "$IADME_GHCR_PAT" | \
  docker login ghcr.io \
    --username forestpondtechnologiesllp \
    --password-stdin

unset IADME_GHCR_PAT
```

## 3. Apply the migration to the development database — Mac

The backend currently uses SQL files rather than an automatic migration
runner. Select and execute each new migration once. `ON_ERROR_STOP` prevents
`psql` from continuing after an error.

```bash
cd /Users/saisrikrishnakumaradavikolanu/Projects/iAdMe/iadme-backend

MIGRATION_FILE="database/migrations/20260914_add_profile_enrichment_sources.sql"

docker compose -f infra/docker/docker-compose.yml up -d postgres

docker exec -i iadme-postgres \
  psql -v ON_ERROR_STOP=1 -U iadme -d iadme \
  < "$MIGRATION_FILE"
```

## 4. Build and push the backend Docker image — Mac

```bash
cd /Users/saisrikrishnakumaradavikolanu/Projects/iAdMe/iadme-backend

IADME_IMAGE_TAG="release-2026.09.14-718eb9d"

docker buildx build \
  --platform linux/amd64 \
  --file services/api/Dockerfile \
  --tag "ghcr.io/forestpondtechnologiesllp/iadme-api:${IADME_IMAGE_TAG}" \
  --push \
  services/api
```

## 5. Apply the migration to staging — run from Mac

The command obtains `DATABASE_URL` from the running staging API container
without printing it, then streams the local SQL file directly to PostgreSQL.

```bash
cd /Users/saisrikrishnakumaradavikolanu/Projects/iAdMe/iadme-backend

MIGRATION_FILE="database/migrations/20260914_add_profile_enrichment_sources.sql"

ssh iadme-prod '
DB_URL=$(docker inspect iadme-staging-api \
  --format "{{range .Config.Env}}{{println .}}{{end}}" | \
  sed -n "s/^DATABASE_URL=//p" | head -n 1)
export DB_URL
python3 -c '"'"'import os, subprocess, urllib.parse; u=urllib.parse.urlsplit(os.environ["DB_URL"]); e=os.environ.copy(); e.update({"PGHOST":"127.0.0.1","PGPORT":str(u.port or 5432),"PGUSER":urllib.parse.unquote(u.username or ""),"PGPASSWORD":urllib.parse.unquote(u.password or ""),"PGDATABASE":urllib.parse.unquote((u.path or "/postgres").lstrip("/"))}); subprocess.run(["psql","-v","ON_ERROR_STOP=1"],env=e,check=True)'"'"'
' < "$MIGRATION_FILE"
```

## 6. Deploy the backend image to staging — EC2

Connect from Mac:

```bash
ssh iadme-prod
```

Run on EC2:

```bash
cd /opt/iadme-staging

IADME_API_TAG="release-2026.09.14-718eb9d@sha256:06c8266ada2cc5e4598a7400eda49aa5186a3e76f3330344fb10137dece17c8b"
sed -i "s#^IADME_API_TAG=.*#IADME_API_TAG=${IADME_API_TAG}#" .env

IADME_API_TAG="$IADME_API_TAG" \
  docker compose -f docker-compose.staging.yml pull api worker

IADME_API_TAG="$IADME_API_TAG" \
  docker compose -f docker-compose.staging.yml up -d \
    --force-recreate api worker

docker ps --format "table {{.Image}}\t{{.Names}}\t{{.Status}}"

curl -fsS http://127.0.0.1:3001/health
```

Return to the Mac shell when finished:

```bash
exit
```

## 7. Apply the migration to production — run from Mac

Do this only after the migration and backend have passed staging checks.

```bash
cd /Users/saisrikrishnakumaradavikolanu/Projects/iAdMe/iadme-backend

MIGRATION_FILE="database/migrations/20260914_add_profile_enrichment_sources.sql"

ssh iadme-prod '
DB_URL=$(docker inspect iadme-prod-api \
  --format "{{range .Config.Env}}{{println .}}{{end}}" | \
  sed -n "s/^DATABASE_URL=//p" | head -n 1)
export DB_URL
python3 -c '"'"'import os, subprocess, urllib.parse; u=urllib.parse.urlsplit(os.environ["DB_URL"]); e=os.environ.copy(); e.update({"PGHOST":"127.0.0.1","PGPORT":str(u.port or 5432),"PGUSER":urllib.parse.unquote(u.username or ""),"PGPASSWORD":urllib.parse.unquote(u.password or ""),"PGDATABASE":urllib.parse.unquote((u.path or "/postgres").lstrip("/"))}); subprocess.run(["psql","-v","ON_ERROR_STOP=1"],env=e,check=True)'"'"'
' < "$MIGRATION_FILE"
```

## 8. Deploy the backend image to production — EC2

Connect from Mac:

```bash
ssh iadme-prod
```

Run on EC2:

```bash
cd /opt/iadme

IADME_API_TAG="release-2026.09.14-718eb9d@sha256:06c8266ada2cc5e4598a7400eda49aa5186a3e76f3330344fb10137dece17c8b"
sed -i "s#^IADME_API_TAG=.*#IADME_API_TAG=${IADME_API_TAG}#" .env

IADME_API_TAG="$IADME_API_TAG" \
  docker compose -f docker-compose.prod.yml pull api worker

IADME_API_TAG="$IADME_API_TAG" \
  docker compose -f docker-compose.prod.yml up -d \
    --force-recreate api worker

docker ps --format "table {{.Image}}\t{{.Names}}\t{{.Status}}"

curl -fsS http://127.0.0.1:3000/health
```

Return to the Mac shell:

```bash
exit
```

## 9. Mobile configuration — saved once, no exports

All mobile commands below run from:

```bash
cd /Users/saisrikrishnakumaradavikolanu/Projects/iAdMe/iadme-mobile/apps/iadme_app
```

Use `dart tool/mobile.dart <environment> <action>`. The launcher loads the
saved JSON, validates required settings, adds `APP_RELEASE` from `pubspec.yaml`,
and starts Flutter. No OAuth, ads, feature, version, or billing exports are needed.

| Environment | Saved profile | API |
| --- | --- | --- |
| `dev` | `config/dev.json` | `http://127.0.0.1:3000` |
| `staging` | `config/staging.json` | `https://staging-api.iadme.app` |
| `prod` | `config/prod-release.json` | `https://api.iadme.app` |

The five boolean video feature switches are `true` everywhere. URLs, client IDs,
numbers, and billing selectors keep their correct types and values.

### Complete Dart-define inventory

| Define | Saved value / behavior |
| --- | --- |
| `APP_ENV` | `dev`, `staging`, or `prod`, matching the selected profile |
| `API_BASE_URL` | Dev only: localhost by default; device overlays below. Staging/prod endpoints are fixed in `AppConfig`. |
| `APP_RELEASE` | Generated from app version/build: `iadme-mobile@<version>+<build>` for prod, with `dev-` or `staging-` prefix for the others. Do not manually maintain it in JSON. |
| `GOOGLE_IOS_CLIENT_ID` | Existing iOS OAuth client ID saved in every profile |
| `GOOGLE_WEB_CLIENT_ID` | Existing web/server OAuth client ID saved in every profile |
| `BILLING_PROVIDER` | `auto`; iOS uses Apple StoreKit. Android uses Google Play in prod release, Razorpay otherwise. |
| `ADMOB_ANDROID_NATIVE_AD_UNIT_ID` | `ca-app-pub-2924641977385769/3473122948` |
| `ADMOB_IOS_NATIVE_AD_UNIT_ID` | `ca-app-pub-2924641977385769/9941107182` |
| `ADMOB_NATIVE_FEED_INTERVAL` | `2` content videos between ad slots |
| `REEL_PRELOAD_ENABLED` | `true`: prepare the next two eligible reels' opening segments |
| `REEL_REPLAY_CACHE_ENABLED` | `true`: retain eligible watched HLS segments in the bounded disk cache |
| `REEL_FAST_START_ENABLED` | `true`: reduce native startup waiting only for completed, locally prepared openings; normal recovery remains enabled |
| `REEL_BACKGROUND_PRELOAD_ENABLED` | `true`: post-build-43 durable public openings, Wi-Fi background Feed/Trending refresh, bounded extra queue; user control in Profile → Settings → Video storage |
| `VIDEO_STARTUP_DIAGNOSTICS_ENABLED` | `true`: sample fast and slow first frames, at most 12 samples in 4 small batches per app session |
| `REEL_PRELOAD_BYTES_PER_MINUTE` | `16777216` (16 MiB/minute speculative-download budget; metered-network policy reduces it) |
| `PREMIUM_VIDEO_PREVIEW_SECONDS` | `5` fallback; per-item backend configuration can override it |
| `PUBLIC_WEB_BASE_URL` | `https://iadme.app` |
| `SENTRY_DSN` | Empty: Sentry remains inactive until a real DSN is configured |
| `UMP_DEBUG_GEOGRAPHY` | Empty: no consent-geography simulation. Optional debug values: `eea`, `us`, `other`. |
| `UMP_TEST_DEVICE_ID` | Empty: no consent-test device override |

Google sign-in is configured in every profile. Ads are enabled in every profile;
the app substitutes Google's test units unless **both prod and release mode**
are active. A prod TestFlight build therefore uses live units; a staging
TestFlight build uses test units. Consent and ad availability still control delivery.

The next two upcoming openings retain priority and the 8 MiB foreground opening
cache. The post-build-43 background feature adds a disk queue of up to eight
public openings: 32 MB for durable openings plus 160 MB watched segments and
bounded metadata, within about 200 MB of video-cache storage. One extra downloader
uses at most a 4 MiB opening buffer; this is not 200 MB of RAM or a total-app-memory
limit. Extra preparation is Wi-Fi/unmetered only and limited to 50 MB/day, separate
from normal playback and next-two preparation. Native battery/storage/Data Saver
gates and OS scheduling apply. See `iadme-mobile/docs/background-reels-2026-09-18.md`.
`REEL_BACKGROUND_PRELOAD_ENABLED=false` is the build-time override; the app also
has a persisted user switch and Clear saved videos in Profile → Settings → Video storage.

The September 17 startup changes start preparation after 150 ms of settled,
healthy playback, preserve a useful in-flight opening across a swipe/ad, and
reuse CDN connections within the active player's cache proxy. They do not add
extra players or increase the download/cache limits. `REEL_FAST_START_ENABLED=false`
restores the previous native startup wait; `VIDEO_STARTUP_DIAGNOSTICS_ENABLED=false`
disables the new sampled reports. Both overrides require a rebuild. See
`iadme-mobile/docs/reel-fast-start-2026-09-17.md` for verification and release gates.

Upload progress does not need a Dart define. S3 upload acceleration is controlled
by the backend and bucket settings, with the agreed prod-only configuration;
these mobile profiles do not enable it for dev/staging.

### Prepare tools

```bash
flutter pub get
flutter devices
```

Use the device IDs printed by `flutter devices` if these saved IDs change. The
launcher selects Android Studio's bundled JDK when available on this Mac.
Android signing, Xcode signing/provisioning and installed SDKs are still required.

**Keep `GeneratedPluginRegistrant` generated by Flutter. Never delete/move it
or use `--no-pub` to bypass a build error.** The launcher rejects `--no-pub`.
The old removal workaround caused the Android plugin-registration failure.

## 10. Run iOS for development

Simulator (dev API and worker must be running on the Mac):

```bash
xcrun simctl bootstatus 923F0814-1A02-4E4C-9DAE-C0F64E8A551F -b
open -a Simulator
dart tool/mobile.dart dev run -d 923F0814-1A02-4E4C-9DAE-C0F64E8A551F
```

Physical iPhone on the same network as the Mac:

```bash
dart tool/mobile.dart dev run -d 00008120-0004786C3E38C01E \
  --dart-define-from-file=config/dev-lan.json
```

`config/dev-lan.json` contains only the dev API address
`http://sais-macbook-air.local:3000`. Update that file if the Mac hostname changes.
The launcher merges it with the full dev profile.

## 11. Run iOS against staging

```bash
dart tool/mobile.dart staging run -d 923F0814-1A02-4E4C-9DAE-C0F64E8A551F
```

## 12. Run iOS against production

Production API and test-ad layout on the simulator (debug mode):

```bash
dart tool/mobile.dart prod run -d 923F0814-1A02-4E4C-9DAE-C0F64E8A551F
```

Production release on the physical iPhone:

```bash
xcrun devicectl device info ddiServices --device 00008120-0004786C3E38C01E
dart tool/mobile.dart prod run --release \
  -d 00008120-0004786C3E38C01E --device-timeout=60
```

Unlock and trust the iPhone and enable Developer Mode when prompted. The iOS
simulator requires debug mode. Use TestFlight for purchase acceptance testing.

## 13. Run Android for development

Connected Android phone over USB, forwarding port 3000 to the Mac:

```bash
adb -s 701d5e0e reverse tcp:3000 tcp:3000
dart tool/mobile.dart dev run -d 701d5e0e
```

Physical phone over the same Wi-Fi instead:

```bash
dart tool/mobile.dart dev run -d 701d5e0e \
  --dart-define-from-file=config/dev-lan.json
```

Standard Android emulator (replace `emulator-5554` with its actual ID):

```bash
dart tool/mobile.dart dev run -d emulator-5554 \
  --dart-define-from-file=config/dev-android-emulator.json
```

The emulator overlay uses `http://10.0.2.2:3000`.

## 14. Run Android against staging

```bash
dart tool/mobile.dart staging run -d 701d5e0e
```

Android staging defaults to Razorpay through `auto`. To specifically exercise
Google Play billing, add `--dart-define=BILLING_PROVIDER=google_play`; complete
licensed-purchase testing requires a suitable Google Play testing-track install.

## 15. Run Android against production

```bash
dart tool/mobile.dart prod run --release -d 701d5e0e
```

This uses Google Play billing and the live native ad unit. Do not tap live ads
during tests. Use a Google Play testing-track install for purchase acceptance.

## 16. Prepare a mobile release

Edit `version:` in `pubspec.yaml` once before a new store upload. The format is
`<app-version>+<build-number>`; the current source is `1.0.4+43`. Increase the build
number for each new App Store Connect / Google Play upload. Both platforms read
that same version, and the launcher generates the matching `APP_RELEASE`.

Validate the saved settings without starting a build:

```bash
dart tool/mobile.dart dev run --check
dart tool/mobile.dart staging apk --check
dart tool/mobile.dart prod aab --check
dart tool/mobile.dart prod ipa --check
```

`--check` validates configuration and prints the Flutter arguments. It does not
check signing, reachability, runtime playback, or store acceptance.

For an occasional artifact version override, pass `--build-name=1.0.5
--build-number=44` to an `apk`, `aab`, or `ipa` command; its release label follows
those values automatically. `run` uses `pubspec.yaml` and accepts no version overrides.

## 17. Build the staging APK

```bash
dart tool/mobile.dart staging apk
```

Output: `build/app/outputs/flutter-apk/app-release.apk`.

## 18. Build the staging IPA

```bash
dart tool/mobile.dart staging ipa
```

Output: `build/ios/ipa/`. IPA export defaults to App Store distribution. A direct
registered-device build can use `--export-method=development` when needed.

## 19. Build the production Android AAB

```bash
dart tool/mobile.dart prod aab
```

Output: `build/app/outputs/bundle/release/app-release.aab`. Upload this to Google Play.

## 20. Build the production Android APK

```bash
dart tool/mobile.dart prod apk
```

Output: `build/app/outputs/flutter-apk/app-release.apk`, for direct installation.

## 21. Build the production IPA

```bash
dart tool/mobile.dart prod ipa
```

Output: `build/ios/ipa/`, for App Store Connect / TestFlight.

Build 43 is an **iOS-only artifact request**, version `1.0.4+43`. Its production
release label is `iadme-mobile@1.0.4+43`. Archive the IPA, signing/version checks,
source/config evidence, and release notes in `artifacts/releases/1.0.4-build43/`.
No Android build or store upload is part of this build-43 request. Android's
physical-device verification is still pending.

The prepared-start and sampled-diagnostic switches are saved in every profile
and validated by `tool/mobile.dart`; do not reintroduce shell exports or omit them
in future builds. Background video downloading and a larger disk allowance are
**not included in archived build 43**. The September 18 working-tree follow-up
implements them for iOS and Android, along with fresh Feed/Trending refresh and
the ready-video snackbar fix. All profiles now save `REEL_BACKGROUND_PRELOAD_ENABLED=true`.
It requires a new build number for release. Read
`iadme-mobile/docs/background-reels-2026-09-18.md` for actual behavior, limits,
controls and tests; the older proposal is historical.

Select the exact versioned IPA from the release folder when uploading. Older IPAs
can remain in `build/ios/ipa/`; do not use a `*.ipa` wildcard or mistake Flutter's
aggregate directory-size message for the size of the new IPA.

Every artifact command uses release mode. Build outputs share the same paths
across environments, so copy/rename a staging artifact before building prod if
you need to retain both. These commands build locally; store upload is separate.
The existing `bash tool/build_store_release.sh android` / `ios` entry points also
use the prod profile now and no longer require exported ad-unit IDs.

## Configuration changes and troubleshooting

- Edit the appropriate saved JSON for lasting changes. Client build settings
  are compiled into the app: stop the current run and launch again, or build
  and distribute a new artifact. Hot reload does not apply Dart-define changes.
- Extra JSON overlays are supported; inline `--dart-define=KEY=value` overrides
  all files. For example, append `--dart-define=REEL_PRELOAD_ENABLED=false` to
  turn opening preparation off, or `--dart-define=REEL_REPLAY_CACHE_ENABLED=false`
  to disable watched-segment retention. A change to one does not disable the other.
- To deliberately test without ad slots, append
  `--dart-define=ADMOB_NATIVE_FEED_INTERVAL=0`.
- `config/dev-reel-quality.json` is a compatibility copy of `config/dev.json`;
  prefer the launcher for new commands. It now uses the standard two-video
  ad interval. Update both files together if keeping that old entry point.
- AdMob app IDs containing `~` remain in native Android/iOS configuration.
  OAuth client IDs and ad-unit IDs are public identifiers. Never put passwords,
  signing keys, GHCR tokens, service-account keys, or database URLs in Dart defines.
- If CocoaPods reports missing `GoogleMobileAds.xcframework/Info.plist` or
  `UserMessagingPlatform.xcframework/Info.plist`, repair those cached dependencies:

  ```bash
  cd /Users/saisrikrishnakumaradavikolanu/Projects/iAdMe/iadme-mobile/apps/iadme_app/ios
  pod cache clean Google-Mobile-Ads-SDK --all
  pod cache clean GoogleUserMessagingPlatform --all
  pod install
  cd ..
  ```

For mobile testing, leave the dev API and worker running. A local API connection
failure is separate from Google sign-in configuration or reel preloading.
