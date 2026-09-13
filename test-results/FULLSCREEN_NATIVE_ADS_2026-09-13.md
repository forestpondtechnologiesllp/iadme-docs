# Full-screen native ads and autoplay — 13 September 2026

## Scope

The owner authorized implementing full-screen ad presentation and video autoplay on Android and iOS. This extends the earlier local [reliability/media-binding fixes](RELIABILITY_MONITORING_FIXES_2026-09-13.md); it does not modify installed build 38. Source remains local and uncommitted. No deployment, store build/release or AdMob account setting change was performed.

## Behavior

- Both Feed and Trending use the shared viewport-filling native ad presentation. A black media canvas replaces the small white card. Portrait creatives have more space, with advertiser identity, supporting copy and action in a compact footer. Landscape screens use media and details side by side.
- Native MediaView explicitly receives the SDK MediaContent; static images fit without cropping and video playback/controls remain SDK owned. The request explicitly accepts any media aspect ratio and starts video muted, with custom controls disabled. This does not request a publisher-controlled or forced playback mode.
- Only the active placement in a visible app mounts its AdWidget. Offscreen pages and hidden tabs detach; returning reuses the cached creative. A temporary `inactive` focus loss (for example an SDK popup) preserves the view. Hidden/background presentation is removed when Flutter can render a frame; releases wait until child disposal. The OS/SDK own background playback suspension, and Flutter may stop scheduling frames while paused.
- A never-visible inactive placement does not acquire native-view ownership or become used inventory. Pool bounds, distinct placement ownership, three upcoming requests and the two-video cadence are preserved.
- Sponsored attribution and SDK AdChoices remain visible, with click handling owned by the SDK. The iOS action button uses a parent UIView for native video click handling.
- Native diagnostics include full-screen layout style, current viewport/media sizes, orientation, attach/detach and SDK playback observations.

Accepting all ratios retains inventory opportunities. A full-screen view cannot turn a landscape asset into a portrait recording, turn an image into video, or guarantee AdMob fill. We did not restrict requests to portrait or video only.

## Validation

- Android 11 Google Play emulator: official video sample loaded four ads in four requests, reported SDK autoplay and an impression, and displayed a real video with muted controls. Forward/backward swipes, hidden-tab return and the presentation lifecycle checks passed. Static-image and compact-footer native checks also passed on the final application source.
- iOS 26.5 simulator: official video sample reported SDK autoplay and showed changing video frames. Static-image and compact native checks passed; four ads loaded in four requests with one SDK impression callback. The final shared-lifecycle source also passed a fresh video rerun, including SDK playback and all presentation checks.
- Portrait media size: Android **1080 × 1334 px** inside a 1080 × 1931 px native view; iOS **402 × 524 pt** inside 402 × 745 pt. These are media containers, not a claim that a landscape video becomes portrait.
- Smallest native layout: Android media height **388 px**, CTA bottom **366 px** within a **388 px** detail panel. iOS media height **139 pt**, CTA bottom **131 pt** within **139 pt**. The compressed footer uses one-line supporting copy and a smaller icon so the action stays above the swipe hint.
- Flutter widget coverage includes 320 × 568, 430 × 932, 844 × 390 and 1024 × 1366 viewports, both themes and 200% text scaling for the Flutter wrapper. Native default-font presentation was checked separately above. The final full mobile suite passed **231 tests**, with **two existing skips**. Final analysis reported only the pre-existing informational `prefer_initializing_formals` lint at `silent_sync_coordinator.dart:26`; no new warnings/errors. Both repository whitespace checks passed.
- The native validator reported no implementation issues. Screenshots use official test inventory; no real-ad clicks or production impression loops were used.

Two development defects were corrected during validation: a short footer could extend beyond the native view, and treating temporary focus loss as hidden could remove the ad behind an SDK popup. The smallest native footer now has an explicit action-bounds assertion, and the lifecycle test preserves the ad on `inactive`. The lifecycle regression forces one final paused test frame because Flutter normally suspends frame scheduling in that state; it does not replace physical OS background/resume acceptance.

An initial Android emulator attempt timed out loading test inventory while basic emulator commands were also slow. A clean isolated emulator run subsequently passed. This is not evidence of production AdMob fill or a measurement of production ad latency.

Saved test screens: [Android video](assets/fullscreen-ads-2026-09-13/android-video.png), [Android validator](assets/fullscreen-ads-2026-09-13/android-video-validator.png), [Android compact](assets/fullscreen-ads-2026-09-13/android-compact.png), [iOS video frame 1](assets/fullscreen-ads-2026-09-13/ios-video-1.png), [iOS video frame 2](assets/fullscreen-ads-2026-09-13/ios-video-2.png), [iOS compact](assets/fullscreen-ads-2026-09-13/ios-compact.png).

## Release / physical acceptance

A new Android/iOS mobile build is required. Before rollout, manually verify actual Feed/Trending with the app's bottom navigation, short and tall screens, portrait/landscape creatives, SDK mute/play controls, backward scrolling, tab return, real OS background/resume and larger native accessibility fonts on physical phones. Production serving should be assessed through ordinary tester use and diagnostics after release. This work does not guarantee every creative is a video or that every eligible break receives a paid ad.

The connected physical Android installation was not replaced, and its account/device settings were not changed by these tests. Both task-owned test emulators were shut down after final verification.

## Official implementation references

- [Android full-screen native ads](https://developers.google.com/admob/android/native/full-screen)
- [iOS full-screen native ads](https://developers.google.com/admob/ios/native/full-screen)
- [Android native video options](https://developers.google.com/admob/android/native/options)

Google recommends consistent media sizing and accepting all aspect ratios for inventory availability. A dedicated ad unit is recommended when introducing a separate placement; this change replaces the existing placement and keeps its current configured units. No separate AdMob API is required to display full-screen native ads.

## Local test logs

Final reproducible results from this run:

- `/private/tmp/iadme-fullscreen-verified-suite.log`
- `/private/tmp/iadme-fullscreen-verified-analysis.log`
- `/private/tmp/iadme-fullscreen-android-focus-video.log`
- `/private/tmp/iadme-fullscreen-android-compact-final.log`
- `/private/tmp/iadme-fullscreen-ios-video-final.log`
- `/private/tmp/iadme-fullscreen-ios-image-final.log`
