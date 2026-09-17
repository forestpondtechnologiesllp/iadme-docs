# iAdMe 1.0.4 (42) — Android startup hotfix and iOS investigation

## Release notes

- Fix Android's immediate launch crash caused by missing Flutter plugin
  registration in build 41.
- Make missing Android plugin registration a build-time error, and reject store
  builds that skip Flutter's plugin-tooling regeneration.
- Add preparation state, cache consumption and playback-route details to the
  existing limited slow-start reports, so iPhone testing can be diagnosed
  without a USB data connection.

**The repeated iOS TestFlight preparing indicator is still under investigation.**
Build 42 does not change preload scheduling, quality selection or cache limits.
The automatic simulator regression prepared both upcoming openings and reused
them successfully, but that is not physical iOS 27 acceptance.

## Android acceptance

1. Update the existing Play internal-test installation to build 42. Preserve app
   data so this verifies the real update path and the Play signing identity.
2. Cold-launch the app three times. It must remain open and show the normal
   signed-in feed or login screen.
3. Verify existing-session behavior and Google sign-in using the Play build.
4. Browse forward/backward through reels and ads; check one active audio stream,
   no crash, and no persistent loading after stopping a rapid swipe.

The separately installed **iAdMe Release Check** package is a startup smoke-test
copy. It uses a different application ID to preserve the Play installation and
is not the production Google sign-in acceptance target. Its optimized release
build passed visible warm and cold launches on the connected OnePlus LE2121.

## iPhone TestFlight investigation

1. Update to build 42, confirm the build number, then close and reopen the app.
2. On Wi-Fi, watch the first reel for 5 seconds, then watch each next reel for
   3–5 seconds before swiping. Include transitions after ads and backward swipes.
3. Do a separate rapid-swipe pass. Distinguish this from the settled-playback
   pass because fast scrolling can outrun bounded preparation.
4. If a preparing indicator repeats, record the screen and note the approximate
   time, timezone, network, and whether it followed an ad. Normal app use sends
   the existing capped slow-start reports; no cable is required.
5. Compare prepared height, playback route, opening bytes consumed, completed
   openings, failure/cancellation counts, budget and eligibility gates before
   claiming the iOS issue is fixed.

No backend deployment, CDN setting change, upload-acceleration change, larger
cache, additional player or extra report request is required by this hotfix.
