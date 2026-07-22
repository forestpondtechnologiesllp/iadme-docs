# iAdMe staging corrective mobile E2E retest — 22 July 2026

## Executive result

The earlier test gaps have been corrected: this retest used the already-installed staging app, made real native permission choices where a prompt was available, selected every Feed filter option, and exercised actual loaded Feed and Trending videos with native swipe gestures.

The player lifecycle fix is behaving correctly in this run. I did not reproduce a crash or multiple simultaneous active players during the verified loaded-video swipe and tab-switch runs.

There is, however, one cross-cutting functional defect that prevents sign-off on infinite scrolling:

> When location permission is granted but the device cannot return a GPS position, the app awaits the position indefinitely. Initial Feed/Trending loading and their pagination can remain in a loading state, with no fallback request and no user-visible recovery path.

The current staging inventory also has only 11 eligible Feed/Trending videos. That permits a real rapid-scroll test through the available content, but it cannot honestly satisfy the requested 25–30 distinct-video or non-empty second-page test without approved staging seed content.

## Environment and method

| Item | Value |
| --- | --- |
| API | Staging: https://staging-api.iadme.app |
| App | iAdMe 1.0.0+14 |
| Device | iPhone 17 Pro Simulator, iOS 26.5 |
| App state | Existing install retained throughout; no uninstall or app-data reset |
| Account | Supplied staging test account, authenticated in the installed app |
| Location | Native location permission granted; a static Coimbatore simulator location was then used to obtain a valid device position |
| Automation | Real iOS/Flutter UI interactions and native vertical swipe gestures, with Flutter device logs captured |

### Permission correction

- Notification permission was already **authorized** on the retained app install, so iOS correctly did not show its prompt again. Keeping the app installed preserves that choice; it does not make an already-granted system prompt reappear.
- Location permission was exercised with the real iOS prompt and **Allow While Using App** was selected.
- Permission alone did not give the simulator a GPS fix. Before a static location was supplied, Feed and Trending could wait indefinitely for location. This is the underlying resilience defect described below, not a reason to bypass the permission flow.

## Retest matrix

| Area | Result | What was actually checked |
| --- | --- | --- |
| Launch and retained sign-in | Pass | The installed staging build launched and remained signed in. |
| Notification permission | Pass / retained | The existing install reported iOS authorization as granted. No duplicate prompt was expected or forced. |
| Location permission | Pass | The native iOS location dialog was shown and **Allow While Using App** was selected. |
| Feed filters | Pass | Opened and selected **All, City, Nearby, Local, State, National, and International** in the live app. The All scope was restored afterward. |
| Feed filter empty state | Pass | International was selectable and showed its valid no-eligible-content state rather than leaving the filter sheet open. |
| Feed rapid video scrolling | Pass, bounded by data | An initial 11-card forward pass and an additional **32 native forward/reverse transitions** over loaded Feed cards passed. The player remained stable and no crash occurred. |
| Feed infinite scrolling | Fail / blocked | At the end of the available page, the app started load more but waited on the location lookup and never issued the next Feed request. |
| Trending initial load | Pass | Trending loaded 11 real playable videos after a valid device location was available. |
| Trending rapid video scrolling | Pass, bounded by data | 12 native forward swipes were made through loaded Trending cards. No crash occurred. |
| Trending infinite scrolling | Fail / blocked | At the end of the available page, load more started but the location lookup remained pending before a pagination request was made. |
| Feed ↔ Trending while playing | Pass | Four loaded-content tab handoffs were executed. Each stopped/disposed the old player before the next player attached and played. |
| Multiple active players | Pass in exercised scenarios | Logs showed one controller lifecycle at a time; no multiple-player diagnostic or unhandled player exception appeared. |
| Premium unlock | Not executable | All 11 active staging videos were public. No locked non-owned premium video was available to unlock. |
| Long root comment | Pass, visual check | A 981-character tagged root comment was posted, rendered with readable wrapping, then removed and verified absent. |
| Long reply | Pass, visual check | An 807-character tagged reply was posted, rendered under its parent with correct indentation/wrapping, then removed and verified absent. |
| Long title / description | Not executable | Current staging media did not provide a suitable long title/description, and no live record was modified. |
| 50+ comments across videos | Not executed | No bulk synthetic comment data was created in this retest. A meaningful test needs authorized tagged seed data and cleanup approval. |
| 1,000 comments / 10,000 likes | Audited, not load-tested | Current source/API behavior was reviewed; it is not sufficient to sign off on those volumes. Details below. |

## What the video tests prove

The verified loaded-video logs consistently show the intended single-player sequence:

1. stop/dispose the prior controller;
2. attach the next controller;
3. start that controller.

Relevant diagnostics included SINGLE_PLAYER_FORCE_STOP, SINGLE_PLAYER_DISPOSE_CURRENT, SINGLE_PLAYER_CONTROLLER_DISPOSE_SAFE, SINGLE_PLAYER_MAIN_ATTACHED, and SINGLE_PLAYER_PLAY. There were no MULTIPLE_ACTIVE_PLAYERS, unmounted-Riverpod access errors, unhandled exceptions, or app termination during the completed loaded-content scenarios.

This is meaningful regression coverage for the earlier rapid-swipe/tab-switch lifecycle issue. It is not a claim that the current 11-video staging dataset proves 25–30 distinct cards or a populated second page.

The additional Feed cycle used 32 actual 180 ms native gestures (16 forward and 16 reverse) over the available cards. It passed in 7m 6s because the UI test runner waits for each card transition to settle; it is therefore valid lifecycle coverage, not a claim of a continuous 32-card human flick.

## Important findings

### 1. Location lookup can block Feed and Trending indefinitely — fix before sign-off

UploadLocationService.getCurrentUploadLocation() awaits Geolocator.getCurrentPosition(...) without a timeout. Its fallback is used only if an error is thrown; a location future that never completes does not throw and therefore prevents:

- the first Feed/Trending request;
- Feed load more;
- Trending load more.

This was reproduced after granting location permission but before providing a simulator GPS coordinate. With a static location set, the first-page Feed and Trending requests completed normally. At the end of each real list, the log recorded the load-more request and provider start, but no subsequent API request, success, or failure result before the test completed.

Recommended implementation:

1. Cache the last valid location for the session.
2. Apply a short timeout (for example, 8 seconds) to the GPS lookup.
3. On timeout or error, use the existing fallback/last-known location and continue the request.
4. Surface a non-blocking location hint only where it improves the result; do not leave the feed spinner indefinitely.

### 2. Current staging content cannot prove a full second page

The live inventory exposed 11 eligible videos in Feed and 11 in Trending. The direct API sequence was effectively:

| Surface | Available sequence |
| --- | --- |
| Feed | 10 items → 1 item → 0 items |
| Trending | 11 items → 0 items |

To complete the requested 25–30-card fast-scroll and true infinite-scroll test, staging needs at least 30 ready, location-eligible public short videos, plus one locked premium video owned by a different user. These should be tagged for the test and removed/reverted afterward.

### 3. Trending cursor implementation will not scale correctly beyond the small dataset

The Trending v2 assembler currently decodes cursor information but fetches pool candidates with an offset of zero and encodes next offsets as zero. At higher volume this can repeat records/loop once the client’s short seen-ID list is exhausted.

Required change: apply each decoded pool offset to candidate retrieval, advance it by the records consumed, and return a null end cursor when no additional records exist.

### 4. High-volume comments, reactions, and activity need pagination work

| Surface | Current behavior | Readiness conclusion |
| --- | --- | --- |
| Root comments | Client asks for 20; server accepts/clamps a 1–50 offset page and the app appends near the end. | Reasonable small-scale pattern, but a stable cursor is preferable before large concurrent comment volumes. |
| Replies | All replies for each returned root comment are included in the response. | Not suitable for a thread with 1,000 replies. Add reply-level cursor pagination and a “View replies” continuation. |
| Reaction/engagement sheet | A single request materializes all users for reaction/share/target/achievement lists. | Not suitable for 10,000 likes. Use per-type cursor pages and lazy-load only the active tab. |
| My Activity | One request loads several tab lists; some repository queries have a limit and others do not. | Give every tab its own capped cursor endpoint; do not fetch uncapped history in one request. |

### 5. Long-comment visual behavior is readable, but the keyboard remains open after posting

Both the long comment and long reply wrapped cleanly without horizontal overflow. After a successful post, however, the composer clears without unfocusing its text input. The iOS keyboard stays on screen and reduces the usable comments area.

Recommended implementation: retain a composer FocusNode and call unfocus() after a successful submit (and ensure tapping the sheet body behaves naturally).

## Capacity position

Do not publish a numeric DAU guarantee without a measured load test. A well-indexed monolith with healthy connection-pool limits, Redis/cache, background media workers, and CDN delivery can be an appropriate architecture for an initial 1K–10K DAU launch; microservices are not automatically required.

Before making even that measured claim, complete the following:

1. Fix the location timeout and pagination defects above.
2. Review database query plans and indexes for Feed, Trending, comments, engagements, and activity.
3. Cap every unbounded list query and add cursor pagination where noted.
4. Add metrics/alerts for API latency, database pool saturation, Redis, media-processing queues, error rate, and player failures.
5. Run an authenticated k6/Artillery workload that models scrolling, comments, reactions, uploads, and concurrent media processing.

For 100K DAU, scale from measurements and identified bottlenecks rather than starting with a speculative microservice split.

## Evidence retained from this retest

- Live filter options and applied City/All selections: /private/tmp/iadme-maestro-open-filter-artifacts/2026-07-22_105936/iadme-maestro-open-filter/takeScreenshot/live-filter-options.png, /private/tmp/iadme-maestro-select-city-artifacts/2026-07-22_110422/iadme-maestro-select-city/takeScreenshot/city-filter-applied.png, and /private/tmp/iadme-maestro-select-all-artifacts/2026-07-22_110704/iadme-maestro-select-all/takeScreenshot/all-filter-reapplied.png.
- Feed native-swipe run: /private/tmp/iadme-maestro-feed-rapid-pagination-artifacts/2026-07-22_104426/iadme-maestro-feed-rapid-pagination/commands.json.
- 32-transition loaded-Feed cycle (passed; 32 recorded swipes, 31 player attachments, zero multiple-player/unmounted errors): /private/tmp/iadme-maestro-32-transition-cycle-artifacts/2026-07-22_120353/iadme-maestro-32-transition-cycle/ and /private/tmp/iadme-maestro-32-transition-cycle-junit.xml.
- Loaded Trending rapid-swipe run and device logs: /private/tmp/iadme-maestro-trending-rapid-pagination-artifacts/.maestro/tests/2026-07-22_112946/iadme-maestro-trending-rapid-pagination/.
- Loaded Feed/Trending tab-switch run: /private/tmp/iadme-maestro-real-tab-switch-artifacts/.maestro/tests/2026-07-22_113617/iadme-maestro-real-tab-switch/.
- Long-comment rendering: /private/tmp/iadme-maestro-long-comment-artifacts/.maestro/tests/2026-07-22_114403/iadme-maestro-long-comment/screenshots/.
- Long-reply rendering: /private/tmp/iadme-maestro-long-reply-clean-artifacts/.maestro/tests/2026-07-22_115511/iadme-maestro-long-reply-clean/takeScreenshot/long-reply-rendered.png.

These are local test artifacts. Tagged comments and replies created during the check were deleted and a subsequent API read verified that neither tagged record remained.

## Next retest gate

After the location and Trending cursor fixes are deployed, provide or authorize:

1. at least 30 tagged, ready, location-eligible staging videos;
2. one tagged locked premium video owned by a different user;
3. a safely isolated high-volume comment/reaction dataset.

Then rerun: 25–30 true card swipes per surface, non-empty second-page checks, premium unlock, 50+ comment-page checks, and paginated reaction/activity tab checks.
