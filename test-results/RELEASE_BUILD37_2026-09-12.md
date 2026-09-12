# Mobile build 37 — 12 September 2026

## Scope

The owner approved implementing the revised upload-screen concept and packaging production Android AAB and iOS IPA files. “IPK” in the request was interpreted as IPA, consistent with the existing iOS release workflow. Store submission and internal-testing distribution remain with the owner.

- A combined video-preview/caption card, compact location control and the shorter **Show location as** label.
- One small helper block below the location control: “Videos must be under 4 minutes. Only your selected area is shown. Your exact location stays private.”
- A compact wallet balance/Add Stars row and a primary **Upload · 5 ★** action.
- Theme-aware surfaces/text and responsive layout for small screens, enlarged text and the keyboard. Retry, upload errors, location loading/unavailability, and caption limits are retained.
- The defensive phone-profile parser fix from `c0d7e1f`: null or missing email no longer prevents a profile from opening.

The navigation shared with other tabs remains the existing app navigation. No upload API, payment, location-permission, ad cadence/cache or backend behavior was changed. The larger ad-preload proposal (IADME-009) remains deferred. The backend profile compatibility fix is already deployed and this mobile release adds defensive handling independently.

## Source and build configuration

- Mobile source: `3a905376ef0d7a6a8b615a51ef1b628099af0698`.
- Version: **1.0.4+37** on both platforms; bundle/package `app.iadme.mobile`.
- `APP_ENV=prod`, `APP_RELEASE=iadme-mobile@1.0.4+37`.
- Production API, native-ad unit IDs, Google OAuth identifiers and two-video ad interval retained from the verified build 36 configuration.
- Dependency lockfile retained. Existing build 36 AAB/IPA files are preserved.
- No new Sentry DSN was supplied; monitoring configuration remains as in build 36.

## Validation

- **171 tests passed** in the full Flutter suite. Two social-login tests requiring configured OAuth were skipped in this default run, then both passed in the production-configured run.
- **6 production-configured social-login/ad preflight tests passed**.
- **20 new upload-composer tests** cover the combined guidance and contrast, small/large displays, 1×/1.5×/2×/3× text, keyboard/landscape layout, retained caption focus during keyboard resize, caption limit, selected location, wallet action, loading/unavailable location, duplicate-action prevention, long locality names and retry versus a fresh upload.
- Profile regression cases cover missing/null/empty email and preservation of existing email/Apple relay addresses.
- Targeted static analysis and formatting passed. Both theme layouts were rendered in the Flutter widget harness with a placeholder video preview.
- The large-text tests caught a wallet-row overflow; the final implementation stacks that row when needed. A separate dynamic keyboard-resize check caught lost caption focus; the final stable scroll tree preserves it and passed the regression test before packaging.

The first Android packaging attempt with `--no-pub` encountered stale generated integration-test plugin registration left by the test run. Regenerating the release plugin metadata with the standard Flutter build workflow resolves that configuration mismatch; no dependency upgrade is required.

## Artifacts

### Android

- AAB: `iadme-mobile/apps/iadme_app/build/app/outputs/bundle/release/iadme-1.0.4-build37-prod.aab` (from the iAdMe workspace root).
- Manifest verified: `app.iadme.mobile`, version name `1.0.4`, version code `37`.
- Size: **73,194,101 bytes**.
- SHA-256: `009e25f5fb1c524a8217c67569ab64443ddc6e0bf8b7e671628a184b3e8ff7a4`.
- JAR signature verified. The upload certificate matches build 36: `B9:D1:45:E2:9F:18:B7:EA:3D:19:21:80:E8:01:B6:C4:5F:62:7F:B5:08:43:CD:70:B4:6C:E4:CB:62:A9:61:2F`.
- Production ad/OAuth/API/release identifiers and the new upload guidance were verified in the compiled app.

### iOS

- IPA: `iadme-mobile/apps/iadme_app/build/ios/ipa/iadme-1.0.4-build37-prod.ipa` (from the iAdMe workspace root).
- Bundle verified: `app.iadme.mobile`, version `1.0.4`, build `37`, minimum iOS `15.0`.
- Size: **32,815,395 bytes**.
- SHA-256: `762237966a6c6eaae7bf7048920bbec3f3e90e23f3fa296e6c4916b30e8cabf2`.
- `codesign --verify --deep --strict` passed. Distribution identity: **Apple Distribution: FORESTPOND TECHNOLOGIES LLP (KU58Q677M3)**.
- The App Store profile matches the application/team, expires `2027-06-09T15:47:27 UTC`, allows no debugging, and contains no development-device or enterprise distribution list.
- The IPA App.framework UUID matches the final archive: `0C7143A3-C72F-8758-485E-50E61FE28FEC`.
- Production native-ad/OAuth/API/release identifiers and the new upload guidance were verified in the compiled app.
- Export method: `app-store-connect`; automatic version/build renumbering disabled. The initial export lost its network connection; the final export succeeded with the existing automatic-signing configuration. The prepared manual-signing fallback was not needed.

Both named build 36 artifacts were rehashed after packaging and match the previous release report. Build 37 mobile source is committed and pushed. These files have not been uploaded to either store or distributed to testers by this task.

## Internal testing

Check the selected-video upload screen in both themes, change location level, enter a long caption, open Add Stars, upload a valid clip and exercise a network-failure retry. Confirm the combined helper note on smaller devices and with enlarged text. Open Profile on the reported phone-only account. Native video picking/playback and physical-device acceptance remain with the owner/testers.

## Release notes

- Refreshed the upload screen with a cleaner layout and concise video/location guidance.
- Improved upload-screen readability in light and dark themes and with larger text.
- Fixed profile loading for accounts registered with a phone number.
