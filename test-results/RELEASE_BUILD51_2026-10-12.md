# Build 51: responsive layout and photo-price rollout — 12 October 2026

## Scope and current gate

This follow-up addresses IADME-046 after the owner requested small/large iOS and
Android testing, large accessibility text, notches and use of the connected phone.
The owner authorized backend production deployment after successful checks.
The owner subsequently requested signed AAB/IPA generation for manual store uploads; no store upload is authorized or performed by this release task.

**Layout verification and staging/production backend deployment are complete.**
Mobile source is committed and pushed as `f5439a7`, version `1.0.4+51`. Signed
AAB/IPA generation and artifact verification are complete; store upload is manual. The archived build-50 AAB/IPA do not contain
these follow-up changes.

## Layout changes

- The normal-size age, mute and menu controls remain inline when the measured
  content fits. The header measures scaled text and wraps when necessary,
  retaining 44dp controls.
- The action rail is 6dp inside the safe content edge, 8dp farther right than
  build 50 on the reported phone. At large text sizes, its bounded viewport can
  scroll so every action remains reachable without covering the header.
- Bottom navigation grows with accessibility text instead of clipping its
  icon/label column. Long labels retain their full semantics but can ellipsize.
- Caption details stack at large text sizes. The caption and rail use the
  measured timer height, keeping both clear of the progress controls. The caption
  can scroll within the remaining space on extremely small, large-text screens.
- Rail counts scale down only as needed to fit their fixed width, preserving the
  complete number and its full screen-reader value. This prevents a clipped
  `15` from visually becoming `1`; it does not abbreviate or round counts.
- The previous outside-tap/Done keyboard fix and server-quoted photo-price UI
  remain included. Playback, next-two preparation and background-cache policy
  have not changed.

The initial expanded matrix found navigation clipping, a long-score header
overflow and a large-text caption overflow. Native Android screenshots then
revealed timer overlap with the caption/report count. These were fixed and
explicit separation assertions added before the final passing matrix.
Reviewing the large-iPhone 3× screenshot caught a further count-clipping issue.
The matrix and native harness now also assert that every rail paragraph renders
all its text and stays within the rail horizontally.

## Automated layout coverage

`release_layout_matrix_test.dart` passes **120 cases**: ten logical viewport and
safe-area profiles × four text scales (1, 1.3, 2 and 3) × Feed, Trending and photo
composer. The application supports portrait orientation.

| Platform | Logical viewport | Safe-area stress case |
| --- | --- | --- |
| iOS | 320×568 | Small phone, 20dp status bar |
| iOS | 375×667 | SE-sized screen |
| iOS | 375×812 | Mini/notch, 50dp top and 34dp bottom |
| iOS | 393×852 | Notch, 59dp top and 34dp bottom |
| iOS | 440×956 | Large Dynamic Island, 62dp top and 34dp bottom |
| Android | 320×568 | Small phone, 48dp navigation buttons |
| Android | 360×640 | Compact screen with 40dp cutout |
| Android | 360×804 | Connected-phone viewport profile |
| Android | 412×915 | Large screen with 48dp cutout |
| Android | 480×960 | Wide phone, side-safe insets and top cutout |

Assertions cover render exceptions, control hit targets, safe navigation,
header/rail separation, caption/rail/timer separation, reachable report action,
and photo publication controls before and after keyboard appearance. Long names,
captions, scores and counts are included. Light theme is exercised at 1.3×;
remaining cases use dark theme. These are logical layout profiles, not claims of
certification on every device model or OS version.

The final 120-case log is `artifacts/diagnostics/layout-release-20261012/matrix-counts-final.log`.
Analysis of the layout files and the subsequent count fix passed with no issues.
The final six focused suites passed another **63 tests** for headers, navigation,
captions, composer behavior, photo posts and quoted prices; see
`mobile-focused-final.log`. This gives 183 distinct mobile automated cases. A separate final release gate
passed **36 tests** for action-rail accessibility, saved production profiles,
launcher validation, billing selection and video upload cost configuration; see
`artifacts/releases/1.0.4-build51/evidence/release-preflight-tests.log`.

## Native checks

**Connected Android:** OnePlus 9 Pro LE2121, 1080×2412 physical
screen at density 480. The owner-approved `app.iadme.mobile.diagnostics` copy ran
the production Feed/Trending/composer widgets with synthetic local state.
All nine screen/text combinations (three screens × 1×/2×/3×) passed, including
native keyboard focus and outside-tap dismissal. No sign-in, publication or
wallet charge is performed by this harness. The Play-installed build 50 and its
data remain separate.

A further attempt to override the phone's display dimensions was rejected by the
device's `WRITE_SECURE_SETTINGS` restriction. Read-back confirmed the original
size and density, without overrides. Small/large Android coverage therefore comes
from the automated matrix; the native Android result is the connected model.

Android evidence: `android-native-final.log`, `android/*.png`, and matching
`android-display-before.json` / `android-display-after.json` in the evidence folder.

**iOS native checks:** iPhone SE (3rd generation, 375×667 logical pixels) and
iPhone 17 Pro Max (440×956 with Dynamic Island/home-indicator insets), both on
iOS 26.5, passed the same nine screen/font combinations each. The large simulator
reused the small simulator's compiled app. Screenshots were visually reviewed,
including normal and 3× Feed spacing, and all result files were retained before
the two task-created simulators were deleted. The previously booted simulator was
restored. This gives **27 native screen/font checks** across the physical Android
phone and two iOS simulators; it is not a physical-iPhone release certification.

Evidence: `ios-small-native-final.log`, `ios-large-native.log`,
`ios-small/*.png`, `ios-large/*.png` and each device folder's
`release-followup-result.txt`. The standalone large simulator's native console
does not mirror every Dart test message; its result file is written only after
all assertions and screenshots complete successfully.

After the count-clipping fix, the complete 120-case matrix passed again. The
connected Android phone and large iPhone simulator each repeated all nine native
screen/font cases successfully with the new full-number assertions. The final
3× iPhone screenshot visibly retains `15`. The small iPhone native pass predates
this isolated counter fix; its sizes are covered by the final automated matrix.
Final rerun evidence: `android-counts-standalone.log`, `android-counts/`,
`ios-counts-final.log` and `ios-counts/`. The earlier Android Flutter-attached
attempt stalled after the phone slept; a fresh standalone diagnostic process
completed all assertions in 17 seconds. No global font, display or lock settings
were changed. Temporary iOS test devices were removed after exporting evidence.

## Backend compatibility and deployment

Backend source `d1bddc6` adds an explicit rollout setting:

- `PHOTO_POST_COST_STARS=5`: accepts unquoted build-50 requests at their displayed
  price and supports the new quoted client.
- `PHOTO_POST_COST_STARS=10`: requires a matching 10-Star quote before reserving
  funds. Missing/stale quotes fail before creation or debit.
- Only 5 or 10 are accepted. Existing draft retries and refunds always retain
  their original ledger debit, including 5-Star reservations.

The owner explicitly chose to keep production at 5 Stars until the app update.
Staging tests the 10-Star activation while production preserves build-50
compatibility. This is the same code
image with different explicit rollout settings, consistently applied to API and
workers. Production 10-Star activation remains a separate rollout step.

The isolated backend suites passed **48 test executions**: 36 at the 10-Star
configuration (unit, photo integration and refund tests), plus the 12 photo
integration cases at 5 Stars. Typecheck passed; GitHub CI for `d1bddc6` also
[passed](https://github.com/forestpondtechnologiesllp/iadme-backend/actions/runs/38246976039).
There are no new database migrations in this follow-up.

**Deployed image:**
`ghcr.io/forestpondtechnologiesllp/iadme-api:release-2026.10.12-layout-photo-d1bddc6@sha256:bc26f0c33c965b8caec2df0d88c5302c0a6780a6bd63fdabadd78f0e1f192856`.

The identical immutable image runs on API, main worker and affiliate worker in
both environments. Live checks confirmed all six containers running with zero
restarts, healthy PostgreSQL/Redis/BullMQ, and consistent prices (staging 10,
production 5). Production continues to use Lambda; staging uses local Sharp.

Before rollout, each environment received a private `pg_dump -Fc` backup,
validated with `pg_restore --list`, plus its previous runtime/Compose files in
`backups/release-20261012-layout-photo-v1/` under `/opt/iadme-staging` or
`/opt/iadme`. Migration count remained 13; no migration was reapplied. The helper
verifies the backed-up baseline before deployment and restores the previous image
and runtime settings if the initial health gate fails. Application rollback can
use these saved files without a database restore because this change has no DDL.

Staging live smoke passed a real synthetic S3 upload, worker processing and CDN
read, exactly one 10-Star debit despite a repeated reservation, and exactly one
10-Star refund despite repeated cancellation. Missing/stale quotes failed before
creation/debit. Synthetic users were disabled, posts hidden and exact fixture S3
objects removed; the audit ledger was retained. Production smoke confirmed
config 5, unauthenticated config 401, stale quote 409 with zero posts or wallet
entries, and successful Feed/Trending/health routes. No production publication or
wallet charge was performed. Both public HTTPS health endpoints passed.

Receipts in `artifacts/diagnostics/layout-release-20261012/`:
`staging-backup.json`, `prod-backup.json`, `staging-deploy.log`, `prod-deploy.log`,
`staging-smoke.log`, `prod-smoke.log`, `release-health.json`,
`staging-public-health.json` and `prod-public-health.json`. The subsequent
`release-health-soak.json` also passed with zero restarts;
`release-log-counts.json` found no JSON error/fatal, HTTP 5xx or plain fatal
records across the six services in its ten-minute window. Raw request logs were
not exported.

## Resource limits and remaining acceptance

Heavy builds run sequentially with reduced scheduling priority. Android uses two
processors/workers and a 2GB Gradle heap; `android-resource-caps.json` confirms
the running release JVM has `ActiveProcessorCount=2` and a 2GiB maximum heap. The old Xcode wrapper missed implicit
build actions; that attempt was interrupted and the corrected wrapper's actual
two-job compile was verified in `xcode-job-limit.json`. The final build-51
archive also records its actual two-job process in `ios-resource-caps.json`
under the release evidence folder. The existing Docker
builder is capped at two CPUs, 3GiB and one build step at a time. The dev API and
worker remain running. The macOS thermal check reported no recorded warning;
this is not a temperature measurement.

These checks validate layout and photo-price contracts. Physical iOS,
release-mode playback/provider/billing acceptance and public store rollout
remain separate. Signed artifacts are verified as recorded below.
The approved source-photo retention and existing image encoders
are unchanged. Evaluating 2560px/WebP 90 still needs original camera images and
matching delivered variants, as described in the
[photo-quality assessment](ANDROID_BUILD50_FOLLOWUP_2026-10-12.md).


## Mobile artifacts

Version `1.0.4 (51)` is built from committed source `f5439a7` with the saved
production profile and automatic private Facebook overlay. Both launcher
preflights passed. Native builds run sequentially, first AAB then IPA. Artifacts,
release notes, checksums and signing evidence are collected in
`artifacts/releases/1.0.4-build51/`. **Both builds and final verification passed.**
The owner will upload both stores manually. No store upload was performed.

| Artifact | Exact file | Bytes |
| --- | --- | --- |
| Android | `iadme-android-1.0.4+51.aab` | 80931080 |
| iOS | `iadme-ios-1.0.4+51.ipa` | 39462510 |

Android bundletool validation, non-debuggable package `app.iadme.mobile`, version
1.0.4/code 51, the expected upload-certificate fingerprint, API 24 minimum/36
target, bundle 16KB alignment and all 12 included 64-bit native library layouts passed.
iOS version/build/bundle ID, strict deep signature, Apple Distribution authority,
production push/team/application entitlements and an unexpired App Store profile
passed. Deployment target is iOS 15.0. Compiled release markers, saved production
settings, enabled next-two/background switches and disabled diagnostic/test
ad overrides were checked. These checks establish artifact integrity and
configuration, not store approval or physical-device release performance.

`SHA256SUMS`:

```text
0773c2a1d338b04b30397ae464117f80d148b058c03d3c3c97097438fdbc0535  iadme-android-1.0.4+51.aab
d1322a1d45c2d9253f978e4e8291180bce5fe1908478fea3c49236e95ae58f3d  iadme-ios-1.0.4+51.ipa
```

See `evidence/artifact-verification.json`, `evidence/build-inputs.json`,
`evidence/android-build.log` and `evidence/ios-build.log`. Android compiled in
358.3 seconds; iOS archive took 151.1 seconds and App Store export 123.2 seconds.
The exact new IPA was selected by bundle version and build timestamp; the
aggregate Flutter output-directory size was not used as the artifact size.
The native build tools emitted dependency/Kotlin migration advisories; both
builds completed successfully without changing dependencies in this follow-up.
Release copy is in `RELEASE_NOTES.md`, `GOOGLE_PLAY_RELEASE_NOTES.txt` and
`APP_STORE_RELEASE_NOTES.txt` in the release folder.


## Photo comparison inputs

The suggested 2560px/WebP 90 setting is an evaluation proposal, not an image
already generated by this release. Use the original camera file from Photos or
the phone Gallery, including the pencil drawing and a detailed outdoor image;
transfer it as an original file (for example by USB or AirDrop). The WhatsApp
screenshots show the UI but are not suitable source-quality references. Compare
the same original through the current and proposed pipelines, recording output
pixels, bytes and visual detail. No new quality preset was silently enabled.
