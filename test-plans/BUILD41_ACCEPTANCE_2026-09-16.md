# iAdMe 1.0.4 (41) — release notes and tester checklist

## Release notes

- Smoother reel transitions with advance preparation of the next two eligible
  videos, including while an ad is visible.
- Improved opening quality: choose a suitable HD rendition from recent network
  measurements, then let the player adapt as playback continues.
- Cleaner thumbnail-to-video transitions when the first video frame is ready.
- Faster revisits when previously watched segments are still in the local cache.
- Visible video-upload percentages, followed by separate confirmation and
  processing states.
- Production uploads use S3 Transfer Acceleration, with a server switch to
  return to normal uploads if needed.

## Build configuration

Both store artifacts are **production builds**, using `https://api.iadme.app`,
the existing Google sign-in configuration and live AdMob native units. Google
Play billing is selected on Android and StoreKit on iOS. The native-ad opportunity
interval remains two content videos. Ad availability depends on live serving.

The next two eligible content reels can be prepared. Unknown network quality
starts with an available rendition up to 720p; measured capacity can select up
to 1080p. Preparation is bounded to 8 MiB of retained compressed media and
16 MiB/min on Wi-Fi, halved on cellular/metered connections. These are limits,
not guaranteed traffic usage. Watched segments use up to 128 MiB of disk cache;
these limits do not describe total app RAM.

Upload acceleration is **on in production and off in staging/dev**. The same
backend release can be deployed in each environment without changing that
setting. No store upload or public mobile release is part of creating these
artifacts; install through the normal internal-testing/TestFlight process.

## What to test

Use a physical Android phone and iPhone. Start with the current installed version
and update to build 41 so session and cache behavior are exercised realistically.
Run the main playback checks on both Wi-Fi and mobile data.

| Check | Steps | Expected result |
| --- | --- | --- |
| Update and login | Update, open the app, then check Google and phone sign-in where needed. | Existing valid sessions remain usable; both sign-in options work. |
| First launch | Close the app, reopen, and play the first reel. | Video starts and the poster disappears when a real frame is ready. A cold start can still need network time. |
| Normal forward swipes | Watch each reel for 3–5 seconds, then swipe through at least 15. | Upcoming reels generally start promptly; no consistent forced-blurry opening, black flash or overlapping audio. |
| Ads between reels | Watch through multiple native ads and any in-stream ads. Swipe to the following content. | Content resumes correctly and eligible upcoming reels can prepare during ads. Do not repeatedly click ads as a test. |
| Rapid scrolling | Swipe rapidly across 10–15 items, then stop. Repeat in reverse. | Only the settled content reel plays; no competing audio, persistent loading or crash. Fast scrolling may outrun preparation. |
| Backward replay | Watch three reels, revisit them, then restart the app and revisit where possible. | Cached segments can be reused. Different renditions, expiry/eviction and missing segments can still require CDN downloads. |
| Network changes | Switch Wi-Fi/mobile data during playback. Enable Android Data Saver or iOS Low Data Mode. | Active playback can adapt/recover; speculative preparation pauses on constrained networks. A brief network handover stall is possible. |
| Pause and resume | Pause manually, switch tabs, background for 30 seconds, then return. | No hidden audio; manual pause is respected and playback resumes according to the normal app controls. |
| Upload progress | Upload a normal clip, then a larger clip around 100 MB. Watch the percentage and post-processing status. | Percentage reflects bytes sent; 100% sent waits for confirmation, then the app saves the post and reports processing separately. The video becomes viewable after processing succeeds. |
| Upload interruption | Interrupt connectivity during transfer, restore it and retry once after a failure. | A useful error and retry option appear; the percentage resets for a new attempt. Uploads still use a single PUT, so retry can resend the whole file. |
| Quality and orientation | View portrait and landscape clips, including a new upload. | Correct orientation, aspect ratio, audio sync and appropriate quality. Low-resolution originals cannot become true HD. |
| Longer session | Browse for 20 minutes with forward/back swipes, ads and tab changes. | No progressive lag, audio overlap, repeated crashes or sustained excessive heat. Report the phone model if these occur. |
| Billing and links | Check Stars/premium screens and a shared video link. | Correct store billing is presented; existing premium-preview and share-link behavior remains intact. Use the configured store test-purchase workflow. |

Uploading and video processing are separate waits. For a useful upload comparison,
record the same file size, network and phone; compare transfer completion
separately from the time until the video becomes ready. Acceleration does not
compress the file or speed up video encoding.

## How to report an issue

Include build **41**, phone model/OS, Wi-Fi or mobile carrier, approximate time
and timezone, video/reel identifier if available, and a short screen recording.
For upload issues, include file size/duration and whether the problem occurred
during the percentage, confirmation, saving or processing stage. Distinguish an
intentional pause from a playback freeze.

## Known limits

- Instant playback and 1080p on every connection are not guaranteed. Real cache
  misses, bandwidth limits and rapid swipes can still show a preparing indicator.
- Cached replay is partial reuse, not full offline video downloading. Private,
  premium, signed and unsupported media bypass these public-media caches.
- Upload progress does not add resumable multipart transfers, background upload
  persistence or automatic compression. Keep the upload screen open to finish.
- This release does not bulk re-encode the existing catalog. The separate dev
  re-encoding tool remains guarded and its earlier AWS permission issue is
  documented in the backend repository.

## Production acceleration switch

Turn off new accelerated upload URLs (takes effect within 30 seconds):

```sh
ssh iadme-prod 'docker exec iadme-prod-api node dist/main/scripts/set-upload-acceleration.js off --environment=production'
```

Use `status` to inspect the setting or `on` to re-enable it. Existing uploads can
finish; no mobile update is needed to change this server setting.
