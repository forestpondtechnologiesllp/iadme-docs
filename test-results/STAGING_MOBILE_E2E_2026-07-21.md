# iAdMe Mobile Staging E2E Test Report — 21 July 2026

## Test scope

- App: iAdMe Flutter mobile app, version 1.0.0 (build 14)
- Device: iPhone 17 Pro Simulator
- Backend: `https://staging-api.iadme.app`
- Method: user-level exploratory testing in the deployed app, followed by source-level fixes and deterministic regression tests

## Results

| Area | Result | Evidence / notes |
| --- | --- | --- |
| Reviewer login | Pass | Reviewer Account B signed in successfully. |
| Feed | Pass | Feed loaded, multiple videos were reached by vertical scrolling, and playback controls remained responsive. |
| Trending | Pass | Trending loaded and supported repeated vertical scrolling. |
| Reactions | Pass | Like, dislike and Super Like/Target interactions updated as expected. |
| Comments and replies | Pass | Comments and replies could be created and read; disposable test content was removed. |
| Sharing | Pass | The native share flow opened with the video link. |
| Profiles and history | Pass with fixes below | Profile, Wallet history, Purchase history, settings and related rows opened. A bottom-navigation overlap risk was found and fixed in source. |
| Messaging and i³ | Pass | Conversation loading and message exchange worked. |
| Wallet recharge | Pass | Razorpay test checkout completed; the wallet increased from 85 to 185 Stars, Purchase history updated and an invoice was available. Cancellation returned to the app safely. |
| Upload | Pass | `uploads.enabled` was initially false from an earlier configuration test. The app correctly displayed the maintenance message and did not charge. After the flag was restored to `true`, the user confirmed upload success; a 17 MB MP4 produced processing, success and 5-Star fee notifications. This was a configuration state, not an upload defect. |
| Notifications | Pass | Upload-started, upload-success, fee and wallet notifications appeared. |

## Findings corrected after the session

- Profile, Messages and Upload now reserve layout space for the bottom navigation bar; Feed and Trending retain the immersive floating layout.
- The floating Feed/Trending navigation is now a compact responsive 68dp bar (64dp on compact phones), and the playback progress control sits 8dp above it on iOS and Android safe-area layouts.
- Icon-only navigation, feed actions, profile controls, upload preview, messaging controls, comment menus and authentication visibility controls now expose accessibility names.
- A Razorpay order left at `created` now displays **Not completed** with an explanation instead of the technical provider state.
- Upload's **Add Stars** button opens the existing Wallet recharge screen and refreshes the displayed balance on return.
- After the file reaches storage, the upload confirmation now makes the pending processing state explicit and tells the user they will receive a notification when the video is ready.
- Reviewer Account B's backend repair source now enforces the documented display name **USER B** and the reviewer-health endpoint now verifies that value. The staging account must be repaired once after deployment before store review; this source change was not manually re-run in the simulator session.
- Apple review notes now reference build 14.
- Help now points invoice and payment-support actions to **Purchase history**.
- Unit, widget and device integration regression tests were added. See the app's `TESTING.md`.

These source changes were not manually re-run in the simulator at the user's request. They are covered by static analysis and automated regression tests before handoff.

## Automated verification after the fixes

- `flutter analyze` — passed with no issues.
- `flutter test` — 15 unit/widget tests passed.
- `flutter test integration_test/navigation_layout_test.dart -d F5DC1BF5-A000-46B9-9911-61EB36C9F72D` — passed on the iPhone 17 Pro Simulator.
- `npm run typecheck` in `iadme-backend/services/api` — passed.
- Flutter reports Razorpay's known future Swift Package Manager compatibility warning; no new application error was introduced.

## Accepted external warning

Razorpay's future Swift Package Manager compatibility warning remains unchanged. It has been reported to Razorpay and does not block the tested CocoaPods build.

## Release follow-up

1. Deploy the backend change, then run the existing **store reviewer seed/repair** action once so Account B's current staging profile becomes `USER B`.
2. Build the next mobile binary from these changes and run the automated commands in `TESTING.md`.
3. Before store submission, confirm uploads are enabled in Platform Configuration and perform a short login → Feed → Profile → Upload smoke check on the exact submitted build.
