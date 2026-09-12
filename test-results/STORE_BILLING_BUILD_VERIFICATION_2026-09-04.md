# Store billing build verification — 4 September 2026

## Result

The Android and iOS store-billing code paths compile for release. Automated
checks pass. Store-side licensed purchase tests remain required before
production promotion.

## Android

- Candidate: `1.0.3+29`
- Output: `build/app/outputs/bundle/release/app-release.aab`
- Size: 72,179,803 bytes
- SHA-256:
  `3feefa754a37feaa8bd4297519b845e27db05aa3ac89a82066d88f82401d1259`
- Release signature: present (self-signed upload certificate, as expected for
  an Android upload key)
- Release dependency graph:
  `com.android.billingclient:billing:8.0.0`
- Packaged `base/root/billing.properties`:

  ```properties
  version=8.0.0
  client=billing
  billing_client=8.0.0
  ```

An installable local smoke-test APK was also produced at
`build/app/outputs/flutter-apk/app-release.apk` (71.4 MB), SHA-256
`9dec81ecadf4070196eee5647225ba53534ff382f06ba2c15da8cf6ce7ea837c`.
Its packaged `billing.properties` also reports Billing Client `8.0.0`.
This APK is not a substitute for a Play-installed licensed purchase test.

The final AAB was rebuilt with production API/billing selection, the Android
production Native Advanced ad-unit ID, and the production Google Web OAuth
client ID. It was built without a production Sentry DSN because one was not
available in the workspace environment; Sentry is therefore disabled in this
artifact.

The build host defaults to Java 26, which Gradle 9.1 cannot use. Android
Studio's bundled Java 17.0.6 then hit a known-class JVM JIT crash during a
clean release optimisation. The successful retry used Java 17 with
`JAVA_TOOL_OPTIONS=-XX:-TieredCompilation`. Prefer a current JDK 17 or 21 for
repeatable release builds.

## iOS

- Candidate: `1.0.3+29`
- Unsigned release compile: passed
- Output: `build/ios/iphoneos/Runner.app`
- Size: 41.8 MB

This verifies compilation only. A distribution-signed IPA, App Store Connect
product availability, server IAP key, and Sandbox/TestFlight purchase tests
are still required.

## Automated checks

- `flutter test`: 66 tests passed
- Backend `npm run typecheck`: passed
- `flutter analyze`: one pre-existing info-level lint in
  `lib/shared/sync/silent_sync_coordinator.dart`; no billing error

## Required store-side tests

Use `docs/runbooks/store-billing-closeout.md` for the Internal testing,
licensed tester, Sandbox/TestFlight, payment-profile, key, and production
promotion steps.

No AAB was uploaded or submitted to Play Console. A local Android run was not
possible on this host because no Android device was connected and both
configured emulators are missing their Android 36 system image. Connect an
Android device (with USB debugging enabled) or install the matching emulator
system image before running the APK smoke test.
