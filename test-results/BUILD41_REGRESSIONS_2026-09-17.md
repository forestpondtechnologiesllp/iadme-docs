# Build 41 regression investigation — 17 September 2026

## Android launch failure — confirmed

The connected OnePlus LE2121 had version 1.0.4, build 41 installed through
Google Play. Its recorded crashes and one fresh launch reproduce:

```text
GeneratedPluginsRegister: Could not find or invoke the GeneratedPluginRegistrant.
java.lang.ClassNotFoundException: io.flutter.plugins.GeneratedPluginRegistrant
java.lang.IllegalStateException: Could not find a ... instance.
The plugin may have not been registered.
```

The failure occurs in `MainActivity.configureFlutterEngine`, when registering
the native ad factory after Flutter's reflective plugin registration fails.
The build 41 release procedure removed the ignored generated Java registrant to
bypass a stale integration-test registration error, then built with `--no-pub`.
Flutter 3.44 skips platform plugin-tooling regeneration when that option is used.
The missing class could therefore compile successfully and fail at startup.

The previous release checks verified signatures, versions, embedded configuration
and tests, but did not launch the final release on a physical Android phone.
That was an insufficient release gate. The build 41 Android AAB must not be
distributed further.

Corrections under validation:

- Reference `GeneratedPluginRegistrant.registerWith` directly from the app-owned
  engine configuration so a missing registry is a compilation failure.
- Reject `--no-pub` in the store build wrapper and allow Flutter to regenerate
  the release registry, excluding development-only plugins correctly.
- Build an optimized `app.iadme.mobile.releasecheck` test app to verify launch
  on the OnePlus while preserving the differently signed Play app and its data.
- Six release-preflight tests passed, including the new skipped-generation guard.

The optimized ARM64 release-check APK completed successfully with build number
42. Its R8 mapping contains `io.flutter.plugins.GeneratedPluginRegistrant`, and
the generated release registry excludes the development-only integration-test
plugin. It installed beside the Play app on the OnePlus; `am start -W` returned
`Status: ok` and its process remained alive for several minutes. After the user
unlocked the phone, the app visibly reached the welcome/login screen with
Google, phone and email options. This is a diagnostic package, not a replacement
store AAB, and it does not change the existing Play app's data.

A second launch after force-stopping only the diagnostic package also reached
the login screen (`LaunchState: COLD`, `Status: ok`). Its new process had no
`FATAL EXCEPTION`, `ClassNotFoundException`, `MissingPluginException` or Flutter
error in the captured startup log. Google sign-in itself was not exercised:
the diagnostic package has a separate application ID for safe side-by-side
installation, so this evidence covers release startup, not production OAuth.

All 34 focused Flutter tests passed: store preflight, single-player lifecycle,
automatic preload policy, ad transitions, budget and memory gates, and watched
segment caching. Static analysis of the changed Dart files and native probe
found no issues.

## iOS startup delay — confirmed; attribution still under investigation

The supplied 91-second TestFlight recording shows preparing indicators on
multiple content transitions, including after native ads. This is not explained
by a stale Android build or by the upload-acceleration setting.

Production client events confirm **physical iOS, release build 41, validated
Wi-Fi**. Two reported startup durations were 5,907 ms and 4,720 ms. Their native
controller initialization consumed 5,865 ms and 4,703 ms respectively; controller
disposal and queueing were negligible. The native first-frame callback was used.

Read-only catalog checks of the latest 25 public ready reels found:

- All 25 selected 720p openings met the implemented format, duration and 4 MiB
  per-opening limits.
- 21 have legacy first segments longer than two seconds, generally 4–10 seconds.
- The actual shipped Dart preloader successfully prepared three affected public
  clips from the Mac in 353, 209 and 223 ms, selecting 720p/1080p as its measured
  network capacity changed. These are Mac download timings, not iPhone results.

Earlier native playback tests prepared the cache manually. A new opt-in native
integration test checks automatic scheduling from real network and playback
state, preparation of the next two clips, and whether the native player consumes
the prepared segment bytes. This distinction is required before attributing
the remaining delay to CDN, scheduling or native player buffering.

The affected iPhone cannot currently connect by data cable. Simulator evidence
must be reported separately from physical-device acceptance.

Xcode subsequently discovered an existing Wi-Fi pairing to that iPhone. A
wireless tunnel connected and confirmed physical iOS 27.0 with Developer Mode
enabled. Launching the installed TestFlight app for console capture was blocked
by Apple's `kAMDMobileImageMounterDeviceLocked` error; the phone must be unlocked
to mount its developer support image. This is a device-lock prerequisite, not
evidence of an incompatible SDK or an app defect. The TestFlight app was not
replaced or uninstalled.

Slow-start telemetry now includes the preparation gates, byte budgets, failures,
chosen playback route, prepared height, and opening bytes consumed during native
initialization. These fields use the existing rate-limited slow-start reports;
they do not add report requests or change playback behavior.

### Automatic native test result

On the iOS 26.5 simulator, the new automatic test passed with all three affected
production URLs. The first cold reel initialized in 18,461 ms and displayed its
first frame in 18,874 ms; this happened immediately after a resource-heavy cold
native build and must not be treated as a physical-device benchmark.

While that reel played, the real network/playback scheduler prepared both
upcoming openings with no failures or cancellations. The next two native
controllers consumed 1,385,936 and 330,880 bytes from those prepared openings.
Their first frames arrived in 596 and 541 ms (initialization 544 and 515 ms).
The test did not manually call opening preparation or fake network readiness.

This proves automatic preparation and native reuse work on the tested simulator
path. It does not reproduce or explain the iOS 27 physical TestFlight failure.
Build 42 adds diagnostics for that gap; it must not be described as a verified
iOS startup fix. Another physical TestFlight run is required to distinguish
unprepared openings, budget/cancellation gates, cache bypass and native delays.

## Replacement artifacts — build 42

Mobile source commit: `87430f28b90c2e4e06f3d03eba19143cd6308010`, pushed to
`iadme-mobile/main`.

The production AAB and App Store-signed IPA were built and locally verified:

| Artifact | Size | SHA-256 |
| --- | ---: | --- |
| `iadme-1.0.4-build42-prod.aab` | 74,087,651 bytes | `967ba57a3262458459454187a0e4737085d8841ce7821014a770aadf4973d6b2` |
| `iadme-1.0.4-build42-prod.ipa` | 33,069,882 bytes | `7bbcda6ea6175da604c947713dd7ffc9592a3b32deda36b3edd65704acc8e4c3` |

Both use the production app identity, production API/sign-in/ad configuration,
platform store billing and the new startup diagnostic fields. The final AAB
contains the generated plugin registry and excludes the development integration
plugin. Android upload signing and iOS App Store distribution signatures passed;
the IPA's App.framework UUID matches its preserved archive and dSYM.

Artifacts and verification evidence are saved under
`/Users/saisrikrishnakumaradavikolanu/Projects/iAdMe/artifacts/releases/1.0.4-build42/`.
Neither artifact was uploaded to a store by this investigation. Production Play
update/authentication acceptance and physical iOS preload diagnosis remain open.

## Evidence and scope

Local evidence directory: `/private/tmp/iadme-build41-investigation/`.
This includes filtered crash evidence, a copy of the installed APK for signature
inspection, recording contact sheets, public HLS manifests, catalog checks,
sanitized production startup context and test/build logs.

No production configuration, databases, CDN settings or media files were changed
as part of the investigation. Upload acceleration remains a separate feature.
