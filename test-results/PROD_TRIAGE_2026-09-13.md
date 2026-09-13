# Production investigation — 13 September 2026

## Scope and evidence

Reviewed production API, worker and nginx logs for **10:20–10:50 IST** (04:50–05:20 UTC), with an additional five minutes to cover the recording's 10:19–10:20 phone clock. Reviewed the owner's 76-second Android recording and current mobile/backend code. Production checks were read-only. No application code, deployment, database records, ad settings or store builds changed during this investigation.

Versions: backend `440e51435adf4b7f57db7b1c79e05ee497384df6`; mobile source `3a905376ef0d7a6a8b615a51ef1b628099af0698`. Successful Android requests and iOS diagnostic events identify mobile `1.0.4+37`. The failed recording itself does not display the app version or installation source.

Raw logs and extracted recording frames remain outside the repositories; no credentials, user contact details, IP addresses or exact locations are included here.

## Android login

- The recording shows Google's consent screen waiting roughly 30 seconds before a generic Google sign-in failure. The subsequent phone send-code attempt ends with “Unable to reach the server.” It never reaches the OTP entry screen.
- Neither a Google authentication request nor a phone send-OTP request corresponding to the recorded failures appears in the collected API/nginx logs. There is no SMS-provider rejection to attribute this attempt to.
- At **10:15:55 IST**, a **OnePlus LE2121 / Android 14 / build 37** Google login reached production and succeeded: HTTP 200 in **123 ms**, followed by successful Feed requests. This does not establish that the phone in the recording was the same device.
- An authenticated refresh succeeded at **10:21:19 IST**, HTTP 200 in **154 ms**.
- From this workstation, one public API check timed out while connecting to port 443 after ten seconds. Subsequent production/staging public API checks succeeded, as did the server-local readiness check. Website connectivity succeeded separately. This establishes an intermittent connection failure on the workstation, not the precise cause of the Android/Google failure.
- API and worker containers remained running. Nginx listens on 443, the host firewall allows 443, and the only fail2ban jail is SSH. These checks do not identify a persistent API outage or an HTTP ban of the tester.

**Still needed:** recording phone model/build/install source; Wi-Fi versus mobile-data comparison; the native Google error code and phone request's underlying Dio/socket/TLS error. Current Google native errors are printed locally and the login screen replaces most of them with a generic message, so server logs alone cannot identify the native-provider failure.

## Ads after tab switches and pagination

Two real-ad failures arrived from **iPhone 14 Pro Max / iOS 26.6.2 / build 37**:

| Time (IST) | SDK result | Requests | Loaded | Impressions | Ready at failure |
| --- | --- | ---: | ---: | ---: | ---: |
| 10:23:16 | `com.google.admob`, code 1, “Request Error: No ad to show.” | 28 | 27 | 17 | 5 |
| 10:25:19 | Same result | 2 | 0 | 0 | 0 |

Both report production units, interval 2, content market IN and no ad-load timeout. These are separate counter snapshots; they must not be added together or treated as a complete session trace. A later snapshot with zero loaded ads confirms an empty inventory at that moment. The earlier snapshot confirms that real ads did load and receive impressions.

**Unresolved:** these events do not identify the active tab, stable placement, or exact tab-switch/pagination sequence. They do not prove that no-fill explains all missing ads, nor that the placement lifecycle works on the affected devices. Test shared-cache demand across two retained tabs, loading during pagination, returning to previous placements and native view reattachment. Add bounded per-surface diagnostics before declaring this resolved. IADME-009's proposed larger cache remains deferred; it was not enabled as a speculative fix.

## Feed filters — confirmed implementation gaps

In `feed-v2-assembler.service.ts`, `getPoolReadsForQuery`:

- `all` and `nearby` return the same pool list; this path does not apply a geographic radius.
- `local` expands to city, state and national pools; `city` expands to state and national; `state` expands to national. The API does not explicitly identify this widening in its response.
- `international` falls through to an empty pool list even though the route and mobile UI expose it.

The current UI's short labels do not explain those semantics. A redesign must align actual filtering and visible labels, explain any explicit fallback, and keep the selected scope visible. Changing appearance alone will not correct these backend gaps.

## Missing uploads and Feed pagination

Production contains **30 non-deleted videos, all public and ready** at inspection. The two video records created in the preceding 36 hours completed processing in approximately **15 seconds** and **8 seconds**. Both have HLS and location keys, and both are present in the national Feed pool and global Trending pool. There was no newer video record at this snapshot.

### Confirmed pagination defect

`getFeedV2` personalizes/reorders pool candidates, but advances each Redis offset by the number of selected candidates. Selected candidates need not be the leading rows of that pool. Advancing the offset can therefore discard unreturned videos.

A local replay executed the **current assembler source**, transpiled with TypeScript, with isolated in-memory pool, hydration and personalization dependencies. It made no production calls or writes. With 30 eligible candidates, the first nine already watched and a page size of ten, the actual assembler returned:

| Page | Videos returned |
| --- | ---: |
| 1 | 10 |
| 2 | 10 |
| 3 | 1 |
| 4 | 0 |

Only **21 of 30** eligible videos were returned. Nine were skipped because the cursor moved past them; watched videos should be deprioritized, not lost this way.

The production Feed request cursors at **10:21:58, 10:22:31 and 10:22:32 IST** contain **10 → 20 → 21** seen video IDs. The same progression repeats around 10:23. Neither of the two recent uploads is in those observed seen-ID sets. This strongly connects the reproducible backend defect to the missing-Feed report. It does not by itself prove why a specific video is absent from Trending or Profile.

The upload screen also refreshes Feed approximately one second after requesting asynchronous processing. That refresh can occur before the video is ready; processing readiness must be distinguished from upload completion in acceptance tests.

**Fix acceptance:** no eligible video skipped or duplicated across pages with watched/unwatched reordering, overlapping geographic pools, blocked/reported content, late readiness and ranking changes; exhaustion must be explicit. Validate the actual reported uploads across Feed, Trending and My Videos. Do not conflate this with the separate earlier backward-video-player defect.

## Other observations and limits

- Within the 30-minute window, all **125 completed non-health application requests** in the captured API logs returned 2xx. This says nothing about attempts that never reached the server or failures parsing/displaying successful responses on the phone.
- Feed: 14 successful requests in that window. Trending: 3 successful requests. Across the expanded capture, Feed handler time was 9–42 ms and Trending 7–19 ms.
- Three iOS build 37 video-startup warnings reported approximately **1.56, 2.17 and 1.51 seconds**. No `NO_ACTIVE_PLAYER` event appeared in this capture.
- Existing mocked ad/social tests do not establish native Google sign-in, native ad rendering, or real inventory availability on a physical phone. These issues remain open for development and physical acceptance.

## Required work by component

1. **Backend:** fix Feed cursor coverage and define/enforce filter semantics. A backward-compatible pagination repair can benefit existing builds.
2. **Mobile:** improve actionable native-auth/network diagnostics and recovery once the failure is identified; validate/fix shared ad lifecycle across tabs/pagination; redesign the filter UI around the corrected semantics; check refresh after processing readiness.
3. **Serving/device checks:** verify real-ad availability separately from placement behavior; compare the failing Android installation and network paths. No deployment or build has been produced by this investigation.
