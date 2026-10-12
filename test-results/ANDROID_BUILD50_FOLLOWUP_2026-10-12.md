# Android build-50 follow-up — 12 October 2026

## Status and scope

The owner reported a second-row age/mute/menu header, an overly inset action rail, a photo-caption keyboard that remained open, a requested photo price increase from 5 to 10 Stars, and soft-looking uploaded photos. The screenshots came from a OnePlus 9 Pro (LE2121). ADB confirmed that `app.iadme.mobile` was installed by Google Play with version `1.0.4`, code `50`. This establishes installation on one device, not public production rollout.

The mobile/backend follow-up is implemented and tested. It is **not deployed**, and the existing build-50 AAB/IPA do not include it. Production remains on the build-50 backend with a 5-Star photo price. Photo encoder settings and original retention have not changed.

Source commits: mobile `1c1d326`, backend `dcd880a`.

## Fixes

- The 360dp-wide phone leaves 332dp for the header. The old 340dp breakpoint, enabled when the three-dot menu is present, forced the controls down unnecessarily. The breakpoint is now 280dp; the filter label can truncate while all touch targets remain 44dp. Large accessibility text still gets a second row.
- The shared Feed/Trending action rail uses a 6dp right inset, an 8dp move on this phone. Captions, progress controls, playback, ads and reel preparation are unchanged.
- Photo descriptions now dismiss focus on outside taps and the keyboard's Done action. The selected description remains intact.
- Photo publishing obtains a server price and submits `expectedCostStars`. New backend reservations cost 10 Stars. Missing/stale quotes are rejected before a post or debit is created. The mobile client refreshes a changed quote and requires another explicit tap. Retry buttons also display the price.
- Existing 5-Star drafts retain their original selection identity and charge. Status, final ledger descriptions, cancellation/expiry/failure refunds and refund receipts read the original ledger debit. They do not refund 10 Stars for an older 5-Star reservation. Closed drafts return to the current price for a new attempt. No database migration is needed.

## Verification

Focused checks, performed sequentially:

- 48 distinct mobile tests passed across `feed_video_header_test.dart`, `upload_composer_test.dart`, `photo_posts_test.dart` and `photo_upload_price_test.dart`. The initial outside-tap test needed a pump after text entry so the test's TapRegion state matched focus; its corrected run passed. The composer plus four price-flow tests then passed together (29 tests). Price scenarios cover an existing 5-Star draft, a quote changing from 5 to 10, unavailable pricing and a closed legacy draft.
- Backend TypeScript typecheck passed.
- Final `flutter analyze` passed with no issues after removing one redundant test import.
- 34 backend tests passed: 5 photo unit tests, 11 photo integration tests and 18 refund integration tests. These used isolated local PostgreSQL 18/Redis fixtures and in-memory S3 with real Sharp, including reservation concurrency, missing/stale price rejection before debit, original-price retry/refunds and exactly-once notification receipts.
- One physical Android regression scenario passed, covering both Feed and Trending headers, the 6dp action-rail inset, outside-tap dismissal with the Android IME, preserved caption text and the visible 10-Star composer button. Flutter reported all tests passed, including teardown.

The owner explicitly approved a separate test copy. The diagnostic package is `app.iadme.mobile.diagnostics`, labeled **iAdMe Diagnostics**, built in dev/debug mode through the checked launcher. It uses production widgets with synthetic local data; it does not sign in, publish a post or charge a wallet. ADB confirmed the original Play package remained at code 50 after testing. Its saved data was not cleared.

Captured device evidence (app surface, not the Android system bars):

- [Feed header and rail](../../artifacts/diagnostics/android-build50-followup-20261012/feed-header-fixed.png)
- [Trending header and rail](../../artifacts/diagnostics/android-build50-followup-20261012/trending-header-fixed.png)
- [Photo composer after keyboard dismissal](../../artifacts/diagnostics/android-build50-followup-20261012/photo-composer-fixed.png)
- [Device result](../../artifacts/diagnostics/android-build50-followup-20261012/release-followup-result.txt)

The synthetic feed intentionally has no video URL; these captures establish layout, not release-mode playback or a real production upload. The later retry-price label change was covered by widget tests, not a second native build.

Native build: `ORG_GRADLE_PROJECT_iadmeDiagnostic=true`, Java active processors 2, Gradle workers 2, 2GB heap, no persistent Gradle daemon, in-process Kotlin, `nice -n 10`, and `dart tool/mobile.dart dev run -d 701d5e0e --target=integration_test/android_release_followup_test.dart`. Native builds and Flutter tests were kept sequential to preserve generated plugin registration. The completed diagnostic runner and isolated database/Redis test containers were stopped. Dev API/worker services stayed running. `pmset -g therm` reported no recorded thermal/performance warnings; this is not a temperature measurement.

## Photo quality assessment

The current source confirms these stages:

| Stage | Current behavior | Implication |
| --- | --- | --- |
| Android selection | Picker requests a 1920×1920 bounding box and quality 92 | Large camera images lose pixels before upload; exact codec handling depends on source format |
| iOS selection | ImageIO bounds to 1920px and writes JPEG at 0.92 | Includes HEIC normalization and a lossy JPEG encode |
| Published display | Sharp auto-orients, bounds to 1920×1920 without enlargement, then WebP quality 82 | Further lossy encoding after the mobile preparation |
| Thumbnail | Generated from display, bounded to 480×480, WebP quality 70 | Intended for small previews, not the full carousel |
| Feed display | Uses the full photo URL, with decode/cache width 1080px | Matches this phone's 1080px physical width; can reduce detail on wider physical viewports |

Example: a 4032×3024 camera photo (12.19MP) becomes 1920×1440 (2.76MP), retaining approximately 22.7% of its pixels. **77.3% fewer pixels is not 77.3% perceptual quality loss.** Quality 92 and 82 are encoder controls, not percentages of remaining quality, and cannot be multiplied to estimate visual loss. Fine pencil lines and text are plausible casualties of resizing and repeated lossy encoding, but the WhatsApp screenshots do not establish exact file-size or perceptual degradation. A comparison needs the original and the delivered variant of the same image.

Recommended options to evaluate:

| Option | Proposed policy | Tradeoff |
| --- | --- | --- |
| Balanced, recommended | 2560px longest edge, WebP quality 90; avoid the first lossy JPEG pass where supported | More detail and fewer compression artifacts, with larger transfers and decoded images |
| Extra detail | 3200px longest edge, WebP quality 92–95 | Better fine detail at higher memory, processing and download cost |
| Artwork/text | Lossless or near-lossless WebP from the best input; choose resolution separately | Preserves encoder detail better, potentially much larger; resizing and already-lossy input still limit quality |

For the balanced option, preserve eligible original JPEG/PNG inputs where limits allow and normalize HEIC once at high quality. Apply byte/pixel safeguards, orientation and metadata privacy consistently. Derive display and thumbnail variants from the best available input. Replace the fixed 1080px decode cap with a bounded physical-viewport-aware size if larger screens are supported. Keep the small thumbnail for previews.

Changing only the server cannot recover pixels already removed by the picker. A quality rollout must update the mobile preparation and both local Sharp/production Lambda implementations, then compare representative originals (including pencil drawings) with delivered variants and record bytes, memory and visual detail. The [Sharp WebP output documentation](https://sharp.pixelplumbing.com/api-output/#webp) describes quality, lossless and near-lossless modes; [resize documentation](https://sharp.pixelplumbing.com/api-resize/) describes bounding and enlargement controls.

Published photos already processed at the old settings cannot regain original detail merely by changing settings. Sources are normally cleaned by the worker after terminal processing and the URL-expiry grace period, with the approved two-day S3 lifecycle as a backstop; re-upload from the original may be necessary. No retention-policy change is proposed here.

## Release dependency

Build 50 shows 5 Stars and sends no `expectedCostStars`. Activating this backend immediately would reject new photo reservations from that app without charging; already-reserved drafts still work at their original price. The new mobile client requires `/photo-posts/config`, so it also needs the new API before it can publish.

Coordinate staging/backend validation and a new owner-uploaded AAB/IPA before activating the new price. A compatibility deployment with the config/quote support at the old price, followed by mobile rollout and then price activation, is an option if uninterrupted old-price uploading is required; that phased deployment is not implemented or performed by this follow-up. Complete real device upload/debit/refund and release-mode playback acceptance as part of the rollout. The original deployment, migrations, storage controls and artifacts remain documented in [build-50 release evidence](RELEASE_BUILD50_2026-10-12.md).
