# IADME-035 development verification — 29 September 2026

## Delivery

Implemented in `iadme-mobile`, `iadme-backend` and `iadme-website` working trees. The existing control layout is retained. Photos follow the selected theme and the same public post/location/engagement model. Upload permits either one reel or 1–5 photos; a photo post costs 5 Stars total, with no premium photo viewing and no internal ad.

The normal organic feed sequence is V/V/P/V with existing commercial breaks after two posts. Two content lanes preserve stable normal-feed pagination and nearby origin across changing page sizes. Trending retains per-type ranking and mixes the same media cadence. Missing eligible media/ad inventory does not create blank pages. Legacy clients remain video-only.

## Automated evidence

| Check | Result |
| --- | --- |
| Backend `npm run typecheck` | Passed |
| Backend `node --import tsx --test tests/*.test.ts` | 110 passed; 6 existing opt-in tests skipped |
| Backend `tests/photo-posts.integration.ts` | 7 passed against a dedicated empty local database and isolated Redis |
| Flutter `flutter analyze` | No issues |
| Flutter full `flutter test` | 444 passed; 4 existing tests skipped |
| Final focused mobile regression run | 49 passed, including late-initialization/details suspension and mixed-page background-refresh regressions |
| Website `node --test test/share-link.test.mjs` | 3 passed |
| Website `node --check admin.js` | Passed |
| API/worker Docker build | Passed; Sharp WebP encoding verified inside the actual container |
| `dart tool/mobile.dart dev simulator` | Passed: unsigned debug `Runner.app` |
| Local API `/health` | HTTP 200; Postgres, Redis and BullMQ connected |

Backend integration uses real Postgres transactions and Sharp image decoding with an in-memory S3 transport. Coverage includes concurrent reservations and other wallet debits, duplicate retry protection, atomic five-image publication, invalid/substituted images, ownership, one-time cancellation/failure/expiry refunds, metadata removal, filters, nearby cursor stability, old clients, sharing, reaction/target lists, blocking/reporting and delete/restore cleanup. No real AWS upload or MediaConvert job was created by those tests.

Mobile tests cover model/engagement preservation, ad cadence, next-two video selection, horizontal boundary versus vertical navigation, retained carousel position, light/dark small-screen composition, scrollable details, late reel initialization suspension, image-cache size/expiry/coalescing and clearing during an in-flight download. Background refresh retains public photo records in saved Feed/Trending pages while preparing only video openings; premium/ad pages still use the existing normal-refresh fallback. The full suite passed before final theme/status-bar refinements; the final focused suite and simulator rebuild passed afterward.

## Local environment

Applied `iadme-backend/database/migrations/20260929_photo_posts.sql` to local development. Built and restarted local `iadme-api` and `iadme-worker`; Postgres and normal Redis were kept running. Temporary integration resources were separate from development data. The dev simulator build uses the checked saved profile and existing generated plugin registration. Installation and launch succeeded on the running iPhone 17 simulator; UI verification stopped at its existing biometric app-lock screen. The lock was not bypassed. The temporary test database and test Redis were removed afterward; development services remain running.

## Release boundaries

No staging/production deployment, cloud bucket-policy/lifecycle change, App Store/TestFlight/Play upload or physical-device acceptance was performed. Before rollout, protect `photo-sources/*` from public/CloudFront reads, add the scoped source lifecycle backstop, verify real signed PUT/CDN delivery, and exercise interrupted uploads and native image selection on physical iOS and Android. Existing manual moderation is reused; the publishing fee is not an automated image moderation system.

Detailed behavior and deployment order:

- `iadme-backend/docs/photo-posts-2026-09-29.md`
- `iadme-mobile/docs/photo-posts-2026-09-29.md`

## Upload and typography refinement follow-up

Implemented the screenshot feedback in the same working tree:

- + opens the media picker directly. iOS 17+ uses Apple's embedded system picker with the custom one-video/up-to-five-photos heading and live rejection of incompatible selections. Older iOS and Android use their native picker and validate the result before composing; native headings on those platforms remain OS-controlled. No full-library access is added.
- Photo thumbnails use a top-left X, a 44-point removal target, hold-and-drag reordering and equivalent screen-reader reorder actions. The immutable paid-upload retry path remains protected.
- Location helper copy is shared by photo/video composers; upload prices remain in the publish action.
- Compact native-system typography applies to shared theme roles, Feed/Trending creator/caption text, details, filters, report sheets, comments/replies and upload. Accessibility text scaling and control geometry are retained.

Validation: final `flutter analyze` reported no issues; 53 focused checks passed, then the full Flutter suite passed **451 tests with 4 existing skips**. The focused suite exercises automatic picker opening/cancellation, account-scoped saved-draft detection, one-video/five-photo exclusivity, actual long-press dragging and X removal in both themes, and existing small-screen/accessibility sheet coverage. The checked `dart tool/mobile.dart dev simulator` build passed after correcting the native registrar API reference. Installed and launched on the existing iPhone 17 / iOS 26.5 simulator. Final visual picker/composer verification awaits unlocking the existing app lock; no lock settings were changed. The dev API remained HTTP 200 and API/worker/Postgres/Redis were kept running.

The custom iOS integration uses the public containment/continuous-selection APIs documented in [Apple's embedded Photos picker session](https://developer.apple.com/videos/play/wwdc2023/10107/). Physical-device and Android acceptance remain release checks.
