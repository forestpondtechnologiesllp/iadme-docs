# Reliability and incident-reporting fixes — 13 September 2026

Follow-up: [full-screen native ads and autoplay validation](FULLSCREEN_NATIVE_ADS_2026-09-13.md) records the subsequent presentation changes.

## Status and scope

The owner authorized development of the critical/high follow-up from the build 38 Android/iOS investigation, plus GitHub incidents for AdMob failures and other observed inconsistencies. The development checks below preceded deployment. Source was subsequently committed and pushed as backend `7eee307` and mobile `697fac5`; **the additive migration and backend API/worker were deployed to staging and production on 2026-09-13**. See the [deployment record](RELEASE_MONITORING_2026-09-13.md). No new mobile store release or production advertising configuration change was made.

The earlier backend `37edc10` deployment and mobile build 38 remain separate release records. Backend `7eee307` now replaces that server image. Installed build 38 is unchanged; the new mobile queue, SDK/media/network fields and recovery behavior still require a new Android/iOS release and physical acceptance.

## Changes

| Area | Implemented behavior | Limit / remaining acceptance |
| --- | --- | --- |
| Wi-Fi / cellular recovery | Native network-change signals on Android/iOS replace idle Dart HTTP connection pools. Active writes are allowed to complete. Temporary failures retain sessions; the existing single safe auth retry remains bounded. | Does not establish or repair the underlying router/ISP/device TCP/TLS cause. Same-network physical reproduction is still required. |
| Connection diagnostics | Final API connection/server errors include request ID, route/method, transport/error stage, network snapshots and a bounded independent native HTTPS health probe. | No OTP, credential or raw request/response export. A native health success can distinguish a stack/path difference but is not proof that a past auth request reached the API. |
| Feed and Trending refill | A failed placement backs off independently while other upcoming placements continue. Fresh unused ads are counted separately from already-shown cache. Forward demand avoids downloading a new ad for a skipped break behind the user. | The opportunity remains after every two content videos; paid fill depends on available inventory and network/consent. No fabricated fallback impression. |
| Cache / request bounds | Current ad plus three upcoming placements and eligible previous cache remain within six native objects and two concurrent loads. Persistent failures across changing placement IDs are request-limited. Only never-mounted, unimpressed unused inventory can transfer to future slots. | A larger adaptive cache and Meta mediation remain deferred. |
| Native ad media | Explicit Android/iOS MediaView content binding, SDK-managed attribution/AdChoices/action handling, finite media sizing, asset/layout and playback observations. Missing native asset contracts are excluded with bounded retry. Disposed callbacks cannot revive inventory or increment impressions. | `onAdLoaded` or an impression alone does not prove visible media. Production Blinkit reproduction still requires the affected physical device. |
| Automatic issues | Encrypted local diagnostic queue, backend PostgreSQL incident/receipt persistence, worker-based GitHub delivery, counted repetitions, distinct platform/build/cause groups and preserved human notes. | Bounded queues/samples, seven-day mobile retention and eventual delivery after connectivity returns; not a guarantee of capture for every uninstrumented defect or native crash. |

The native iOS test exposed an overgrown advertiser row that squeezed video media to 120 pt. Bounding that row gave the media 227 pt on the tested screen. The subsequent official-video test passed SDK playback callbacks and showed visibly changing video frames. This is local implementation evidence, not proof that every production ad failure had that cause.

## Verification

- Mobile unit/widget coverage includes Feed/Trending two-video placement, pagination, forward/backward scrolling, rapid demand changes, bounded no-fill retries, fresh-versus-used cache, auth/session races, real local HTTP socket replacement without replaying an active auth write, secure-storage failures, offline retry IDs, capture during upload, sensitive-field redaction and payload limits.
- A dedicated platform-channel test sends an asset-missing observation before `onAdLoaded`, verifies the unusable ad stays out of the pager, observes the retry delay, replays early video playback through a later layout event and rejects a disposed ad's impression callback. Passed after correcting the test's fake-clock setup.
- Backend: **31 unit tests passed**, TypeScript typecheck passed. Real disposable PostgreSQL integration passed migration rerun, 20 concurrent duplicate submissions, counted new batches, retained GitHub 429 failure, concurrent worker delivery, issue update cooldown and info-event exclusion. GitHub HTTP was mocked for these automated tests; real production/customer events were not generated by the test suite.
- iOS simulator: official static-image ad test passed; official video ad test passed playback callbacks, three forward/backward swipe cycles, hidden-tab unmount/return, four prepared objects and one SDK impression callback. Screenshots showed actual changing video frames and AdMob's validator reported no implementation issues. A separate read-only production `/health` check returned HTTP 200 through both Dart and native URLSession.
- Android 11 Google Play emulator: official video-ad test passed native playback callbacks, bidirectional swipes, tab detach/return and four prepared creatives. The cold run needed seven requests to obtain four ready ads; initial SDK load errors/timeouts recovered. Its installed Google Play Services version was old and emitted update-required warnings. That environment is not evidence of current production fill rates.

The final full mobile suite passed **225 tests with two existing skips**. `flutter analyze --no-pub` reports only the pre-existing `prefer_initializing_formals` informational lint in `silent_sync_coordinator.dart:26`; no new analyzer warning/error was introduced. All three repository diffs passed whitespace checks.

Android's stationary visual rerun passed with four requests, four ready creatives and one SDK impression callback. Its initially blank screenshot was a test-harness artifact: a single long `tester.pump` slept without painting Android's texture-composed platform view. Replacing that hold with continuous 100 ms pumps showed the full native view and changing video frames. No production renderer setting was changed to force that result. The native validator reported no implementation issues.

The final iOS static-image rerun also passed against the final source: four requests, four ready creatives, one SDK impression callback and a 376 × 240 pt media view. No implementation failure remains in the completed automated/native test set. Physical and production-serving acceptance remains open as described below.

The normal Android application entry point also compiled successfully with `flutter build apk --debug --dart-define=APP_ENV=dev --no-pub`, without the diagnostic application-ID override. This is a development APK, not a new production/store artifact, and it was not installed over the user's Play-signed app.

Saved official-test-inventory frames (no customer/account content): [Android frame 1](assets/reliability-2026-09-13/android-video-1.png), [Android frame 2](assets/reliability-2026-09-13/android-video-2.png), [iOS frame 1](assets/reliability-2026-09-13/ios-video-1.png), [iOS frame 2](assets/reliability-2026-09-13/ios-video-2.png).

## Physical acceptance still required

The connected OnePlus was locked with the screen off when checked; a new test cannot bypass its keyguard. The iPhone was offline. Updated Google/phone/Apple login, same-Wi-Fi versus cellular handover, physical native-media display and a long Feed/Trending traversal are therefore **not yet accepted** for this code.

1. Unlock the Android and connect/unlock the iPhone. Record the installed test build and signing source.
2. Compare both phones on the same Wi-Fi with cellular fallback disabled, then cellular and a second Wi-Fi network. Capture the failure with the new request/native-probe diagnostics. If the failure recurs below both HTTP stacks, capture a narrowly scoped TCP/TLS trace before deciding on infrastructure changes.
3. Verify Google and phone login on Android; Apple/Google/phone on iOS as applicable; cold launch, background/resume and Wi-Fi/cellular handover without duplicate OTP requests or unnecessary logout.
4. Traverse at least 20 content videos in each surface with official test inventory; inspect every ready slot, pagination, fast/backward scrolling, tab return, consent denial/recovery and memory pressure. Confirm motion and static media visually, not only from SDK impressions.
5. After an explicitly authorized deployment/release, validate real production delivery using diagnostic/AdMob reports and ordinary tester use. Do not use automated clicks or repeated paid-impression loops to measure fill.

The prior evidence established an intermittent Android Wi-Fi TCP/TLS failure outside Flutter. iOS success did not measure an identical route or exclude cellular fallback. This implementation improves recovery and diagnosis; it does not claim that Android Wi-Fi versus iOS has a proven permanent network fix.

Cleanup: removed the task's separate `app.iadme.mobile.diagnostics` harness from the physical Android. The Play-signed `app.iadme.mobile` remains build 38, with Wi-Fi enabled and the original stay-awake setting of 0. Removed the disposable local monitoring-test database after validation. Production/staging data and credentials were not changed.

## GitHub reports and operational guide

Created with the owner's authorization; no issue is closed as fixed:

- [#19 — Android production no-fill / inventory gaps](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/19)
- [#20 — iOS production no-fill / missing ads](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/20)
- [#21 — Android loaded ad with black media](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/21)
- [#22 — Android Wi-Fi login fails before the API](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/22)

The tickets preserve the confirmed build 38 observations and state missing SDK response identifiers explicitly. Private login recordings, credentials and personal contact information were not uploaded. See the [incident-reporting runbook](../runbooks/mobile-incident-reporting.md) for payload fields, retention, retry behavior and backend-first rollout order, and [IADME-017](../roadmap/BACKLOG.md#iadme-017--automatic-diagnostic-github-incidents) for tracked acceptance.
