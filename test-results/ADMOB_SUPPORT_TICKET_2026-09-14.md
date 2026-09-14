# AdMob support case — iAdMe native-ad no-fill

Prepared: 2026-09-14
Evidence attachment: [`assets/admob-support-evidence-2026-09-14.json`](assets/admob-support-evidence-2026-09-14.json)

## Copy/paste support request

**Subject:** Repeated Native Advanced no-fill on iAdMe Android production unit; intermittent iOS no-fill

Hello AdMob Support,

Please investigate repeated Native Advanced no-fill for iAdMe. The first controlled Android release-mode capture recorded 31 consecutive `ERROR_CODE_NO_FILL` results across two sessions, Feed and Trending, on a validated cellular connection. The grouped production incident subsequently reached 65 no-fill occurrences through `2026-09-14T09:13:32.171Z`. Historical Android and iOS production-unit tests sometimes loaded ads, but also produced no-fill runs and missing ad opportunities.

We understand that no-fill means the request reached the ad service but no suitable ad was returned. AdMob now shows both app records as **Ready** with **Ad serving enabled**, and app-ads.txt as **found and verified** for both platforms. We are asking you to confirm whether an app/account serving limit, policy status, regional Native Advanced demand, buyer targeting, consent eligibility, or ad-unit configuration explains the repeated results.

### Publisher and application

- Publisher ID: `pub-2924641977385769`
- App: iAdMe
- Android package: `app.iadme.mobile`
- iOS bundle: `app.iadme.mobile`
- Android AdMob app ID: `ca-app-pub-2924641977385769~7002018420`
- Android native ad unit: `ca-app-pub-2924641977385769/3473122948`
- iOS AdMob app ID: `ca-app-pub-2924641977385769~8882076308`
- iOS native ad unit: `ca-app-pub-2924641977385769/9941107182`
- Current test distribution: Google Play internal testing and TestFlight. AdMob links the Android record to Google Play and the iOS record to App Store ID `6778307078`.
- Public app-ads.txt: `google.com, pub-2924641977385769, DIRECT, f08c47fec0942fa0`
- URLs: <https://iadme.app/app-ads.txt> and <https://www.iadme.app/app-ads.txt>. Both return HTTP 200 over HTTPS; HTTP redirects to HTTPS; `robots.txt` does not block the crawler. AdMob confirms **app-ads.txt file found and verified** for both app records.

### AdMob console status confirmed on 2026-09-14

- Android iAdMe: **Ready**, **Ad serving enabled**, linked to Google Play, one active ad unit.
- iOS iAdMe: **Ready**, **Ad serving enabled**, linked to App Store ID `6778307078`, one active ad unit.
- app-ads.txt: **100% of queries authorised**; both Android and iOS rows say **app-ads.txt file found and verified**. The page showed 111 Android queries and 250 iOS queries in the previous seven days, last crawled three hours before capture.
- Payments: AdSense (India), no current identity-verification action shown. The verification page says identity details may be requested once earnings reach the verification threshold.
- Mediation: no custom mediation group is configured. The AdMob default group recorded 138 impressions in the previous seven days, confirming that the account has served ads during this period.
- These screenshots rule out app-readiness, disabled app serving, app-ads.txt verification, and a current identity-verification prompt as explanations for the captured no-fill runs.
- Policy Center and Ads Activity report results are still required to distinguish an account/app serving restriction from demand-only no-fill.

### Current Android evidence — build 39

- Release: `iadme-mobile@1.0.4+39`
- Release mode: true
- Test ad unit: false
- Google Mobile Ads Android SDK: `25.3.0`
- Flutter `google_mobile_ads`: `9.0.0`
- Device-reported environment: OnePlus / `OnePlus8Pro` / Android 11. The build did not capture an emulator-versus-physical flag, so device type is unavailable.
- Network: cellular, available, Android-validated and metered
- Consent state: eligible to request ads (`consent=true`)
- Request content market: `IN`
- Result for all 31 recorded requests: code `3`, domain `com.google.android.gms.ads`, message `No fill.`
- Response IDs returned: none
- Mediation adapter class returned: empty
- Adapter response waterfall returned: empty for every failure
- Loaded ads: 0
- Shown ads: 0
- Client timeouts: 0
- Surfaces: 18 failures in Feed; 13 in Trending; six stable placement IDs
- Load-result latency: 270–6,116 ms; average 1,385 ms; median 895 ms; approximate p90 2,339 ms
- Bounded retry backoff: 15, 30 and 60 seconds; maximum two concurrent loads

Session 1 produced 21 consecutive no-fill results from `2026-09-14T05:37:02.854085Z` through `2026-09-14T05:39:59.264190Z`. Session 2 produced 10 from `2026-09-14T06:27:41.863770Z` through `2026-09-14T06:29:31.021420Z`. The attached sanitized JSON includes each UTC timestamp, placement, latency, counters, consent, network state, SDK versions and device fields.

### Later build 39 physical-device retest

- The Google Play internal-test installation was independently verified on a physical OnePlus `LE2121`, Android 14, as version `1.0.4+39`, installed by Google Play.
- The device reported Wi-Fi available, validated and unmetered; consent remained eligible and the production Android native unit was used.
- In the observed session, six loads reached the SDK: five returned code 3 / `No fill` and one loaded. No ad reached a displayed/impression callback during the recorded Feed/Trending pass, so the app omitted the unfilled breaks and continued with videos.
- By `2026-09-14T09:13:47.157Z`, issue #30 had grouped 25 planned breaks with no fresh ad inventory. These are consequences of unavailable inventory and must not be added to the 65 SDK load-failure count.
- The SDK also returned three code 1 errors stating `Too many recently failed requests ... wait a few seconds`. Those three events are grouped separately in issue #31. The current client could start four requests inside a 15-second window after failures; a pending mobile fix reduces that to three and applies a global 60-second pause when this exact SDK throttle is returned.

The original JSON attachment remains a fixed record of the first 31 consecutive no-fill callbacks; it does not claim to contain the later occurrences.

### Historical production-unit evidence — build 38

Android, OnePlus 9 Pro `LE2121`, Android 14, cellular:

- `2026-09-13T12:23:05.526Z`: code 3 / `com.google.android.gms.ads` / `No fill`; session snapshot requested 6, loaded 5, shown/impression callbacks 3, timed out 0. This establishes that the same production unit could load ads but did not fill every request.
- `2026-09-13T12:24:47.918Z`: Trending had no fresh eligible creative at a planned break; requested 10, loaded 5, shown 5, skipped opportunity 1.
- `2026-09-13T12:30:24.359Z`: code 3 no-fill; requested 23, loaded 7, shown 7, timed out 0, skipped opportunities 14.
- `2026-09-13T12:33:41.900Z`: code 3 no-fill in a new request run; requested 2, loaded 0, timed out 0.
- `2026-09-13T15:26:52.331Z`: code 3 no-fill; requested 10, loaded 0, shown 0, timed out 0. Response ID and adapter waterfall were absent.

iOS, iPhone 14 Pro Max, iOS 26.6.2:

- `2026-09-13T11:48:11.726Z`: code 1 / `com.google.admob` / `Request Error: No ad to show.`; response ID `eo2mapq4LLWGjeYPt9XAwAg`; requested 5, loaded 4, shown 2, timed out 0.
- `2026-09-13T11:48:25.628Z`: Feed had no fresh eligible creative at a planned break; requested 5, loaded 4, shown/impression callbacks 4, skipped opportunity 1.
- The build 38 diagnostic did not capture the iOS native SDK version or complete adapter waterfall. The response ID above was recovered from the saved server event and should be used for AdMob lookup.
- A build 39 user test confirmed that the new full-reel native layout renders successfully, but at least one delivered creative used non-Indian text. No response ID or advertiser metadata was captured for that observation. The client supplied content-market context for India, but AdMob creative language and geography are selected by Google's serving system and are not guaranteed by request keywords. Please advise whether publisher blocking controls, app readiness, mediation or regional demand settings can improve relevance without materially worsening fill.

### Integration details already verified

- The app makes individual native requests through Flutter `google_mobile_ads` and platform `NativeAdView` factories.
- Request format is native, with `MediaAspectRatio.any`, SDK-owned controls and video starting muted. No portrait-only filter is applied.
- Request keywords are `India`, `local communities` and `short videos`. They describe app content and are not treated by us as guaranteed geographic targeting.
- The released build 39 pool allows six native objects and at most two concurrent loads. Unfilled requests retry at 15, 30 and 60 seconds, capped at 60 seconds. The request HTTP timeout is 18 seconds. A locally implemented, unreleased client fix additionally limits starts after failures and honors the SDK's own recent-failure throttle for 60 seconds.
- Feed and Trending plan one skippable ad opportunity after every two user videos. Unfilled opportunities are omitted, so users continue to the next video rather than seeing an application-created blank page.
- Official Google sample native ads load, render, autoplay video while muted and report impressions in the Android/iOS verification harnesses.
- Real production ads loaded during some build 38 sessions, proving the app ID/ad unit can return ads.
- Build 39 records fresh inventory separately, uses per-placement replenishment and does not reuse a displayed ad as fresh stock.
- The observed build 39 failures were explicit SDK no-fill callbacks, not client timeouts, API failures or missing ad-slot creation.

### Questions for AdMob Support

1. Although both apps show **Ready** and **Ad serving enabled**, is either app or the publisher account subject to a serving limit, invalid-traffic assessment, policy restriction or other restriction not visible on the Apps page?
2. Can you inspect Android unit `ca-app-pub-2924641977385769/3473122948` through `2026-09-14T09:13:32.171Z` and explain the 65 grouped code 3 callbacks without response IDs or adapter-response details? The attachment contains the first 31 callbacks in full.
3. Can you inspect iOS response ID `eo2mapq4LLWGjeYPt9XAwAg` and confirm the reason no suitable ad was returned?
4. Do you see low or unavailable Native Advanced demand for India, buyer targeting restrictions, format restrictions, or auction exclusions affecting either unit?
5. Why did the Android error callbacks contain neither a response ID nor an adapter-response waterfall, and what additional Ad Inspector output would let you trace those requests?
6. Can you confirm from your backend that app IDs, package/bundle association and app-ads.txt authorization are healthy, consistent with the attached console screenshots?
7. Are there account, ad-unit, blocking-control or privacy settings we should change to improve match rate and India relevance without violating policy?

There were no intentional clicks on live ads during these tests. Please tell us if you need an Ad Inspector export, AdMob console status screenshots, store URLs after publication, or another controlled UTC test window.

## Internal assessment

The screenshots eliminate the earlier unpublished/unreviewed-app hypothesis: both app records are **Ready**, both show **Ad serving enabled**, both are linked to their stores, and both app-ads.txt records are verified. The captured code 3 callbacks therefore most strongly indicate that no eligible Native Advanced creative matched or won those particular requests, unless Policy Center or an account-level serving-limit notice says otherwise.

This does not mean AdMob ran out of ads globally. Eligibility is evaluated per request using the app and ad unit, country, format, buyer targeting, consent/privacy state, policy and invalid-traffic controls, and auction demand. The later Android retest loaded one of six requests, and earlier Android/iOS sessions also loaded some production ads. That proves the units can serve while still having a poor match rate during specific sessions.

The next decisive evidence is the Ads Activity report. If requests are recorded but matched requests are near zero, the remaining problem is AdMob demand/eligibility or serving controls. If matched requests are healthy but impressions/show rate are low, the app still has a load-to-display problem. If AdMob records no requests, the client is not reaching the ad service as expected.

Build 39 removed the earlier shared-cooldown starvation, stale-inventory accounting, blank-placeholder insertion and incomplete native-media binding defects. The later retest exposed one remaining client weakness: after repeated no-fill callbacks it could still reach the SDK's recent-failure throttle. The pending rate-limit fix prevents that request pattern, but it cannot create AdMob inventory. The original 31-callback capture did not include a device-type flag; only the later OnePlus `LE2121` retest is independently confirmed as physical.

## GitHub issue review

| Issue | Finding | Use in support case |
| --- | --- | --- |
| [#19](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/19) | Android build 38 had intermittent code 3 no-fill and missed Trending opportunities. App replenishment contributed to missed placements; the SDK no-fill itself came from Google. | Historical evidence; separate delivery no-fill from the fixed app-pool behavior. |
| [#20](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/20) | iOS build 38 had one recorded `No ad to show` response followed by an inventory gap. | Include response ID `eo2mapq4LLWGjeYPt9XAwAg`; the issue body incorrectly says no response ID was captured. |
| [#21](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/21) | One loaded Blinkit creative rendered text and CTA but black media on Android; other creatives rendered normally. | Secondary rendering report only. It lacks response ID, exact request time, media type and native callbacks, so it cannot establish an AdMob creative defect. Build 39 changed the app's native media binding; physical recurrence is unverified. |
| [#24](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/24) | Android build 38 requested 10 ads in the recorded run and loaded none; final callback was code 3 no-fill with no response ID. | Strong historical zero-fill run, though build 38 had less-complete diagnostics. |
| [#27](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/27) | Android build 39 reached 65 grouped code 3 failures through `2026-09-14T09:13:32.171Z`. GitHub truncated the event sample. | Primary no-fill evidence; the attached JSON restores the first 31 sanitized samples. Device type is unknown for those first sessions; the later OnePlus retest is confirmed physical. |
| [#30](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/30) | Build 39 grouped 25 planned Feed/Trending breaks where no fresh loaded ad was available. | Demonstrates user impact from the load outcomes. Do not add these gaps to SDK failure totals. |
| [#31](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/31) | Google returned three explicit recent-failure throttle errors on the physical OnePlus retest after repeated no-fill callbacks. | Shows a separate client request-rate weakness; a pending mobile fix applies a 60-second global pause. It does not explain the preceding code 3 no-fill callbacks. |

Issue [#23](https://github.com/forestpondtechnologiesllp/iadme-mobile/issues/23) is excluded because it was a controlled staging monitoring-delivery smoke test, not an AdMob request. Video playback/network issues are also excluded unless they contain an AdMob SDK event.

## Evidence integrity and remaining console fields

The attached JSON contains no user IDs, credentials, phone numbers, emails, IP addresses or precise locations. Session and placement identifiers are random diagnostic correlation values.

The Apps, app-ads.txt and Payments screenshots now confirm:

- Android and iOS are **Ready** with **Ad serving enabled**.
- Both app-ads.txt records are **found and verified**.
- No current identity-verification action appears on the Payments verification page.

Before submitting, add one **Policy Center** screenshot and one Ads Activity report export for 2026-09-13 through 2026-09-14, broken down by app, ad unit, country and platform, with requests, matched requests, match rate, impressions and show rate. These are the remaining account-side facts needed to classify the problem.

## Immediate publisher actions

1. Open **Policy Center** and capture whether it reports no issues, restricted ad serving or limited ad serving. A notice there can explain low or zero demand even while the Apps page says Ready.
2. In **Reports → Ads Activity**, select 2026-09-13 through 2026-09-14; break down by app, ad unit, country and platform; include requests, matched requests, match rate, impressions and show rate.
3. Reproduce one controlled physical-device release-mode session and retain its UTC window, ad-unit ID, response ID, adapter responses, app version, SDK version, device model, consent state and network class. Use Ad Inspector to copy troubleshooting output where possible.
4. At the bottom of an AdMob Help Center page choose **Contact us**, enter the no-fill topic, review the suggested resources, select **Next step**, and use the offered publisher-support channel. Google says email replies normally arrive within two business days. If no contact choice is offered, post the sanitized case in the official AdMob Help Community and include the same console statuses and response IDs.

Code cannot force fill. A no-fill response means the request reached Google's serving system but no eligible creative won for that request after app/account readiness, policy, consent, format, buyer targeting, regional demand and auction constraints. It does not mean Google has exhausted every ad globally. Meta mediation can broaden the eligible demand pool after app readiness is healthy, but an app-level serving restriction can still limit every mediated source.

## Google references

- [Guide to common Google Mobile Ads SDK error codes](https://support.google.com/admob/answer/15090849): no-fill means the request succeeded but no suitable ad was available.
- [About app readiness](https://support.google.com/admob/answer/10564477): new apps must be published, linked to a supported store and approved before full serving.
- [Verify your app with app-ads.txt](https://support.google.com/admob/answer/14538460): app verification and readiness affect full serving.
- [Retrieve Android ad response information](https://developers.google.com/admob/android/response-info): response and adapter data available from load errors should be supplied when the SDK returns it.
- [Contact AdMob publisher support](https://support.google.com/admob/answer/9948073): use **Contact us** at the bottom of the Help Center and follow the category/resource/contact flow.
- [New app review and limited serving](https://support.google.com/admob/answer/12206349): all new apps undergo readiness review and may have limited ad serving.
- [App-ads.txt verification and serving](https://support.google.com/admob/answer/15948559): newer unverified apps can experience limited serving.
- [Ads Activity report metrics](https://support.google.com/admob/answer/10979428): match rate compares matched requests with requests; a matched request does not necessarily become an impression.
- [Troubleshoot ad units with Ad Inspector](https://developers.google.com/admob/flutter/ad-inspector/troubleshoot-ad-units): inspect request waterfalls, no-fill results and buyer-generated diagnostic data.
- [Copy Ad Inspector troubleshooting output](https://developers.google.com/admob/flutter/ad-inspector/copy-troubleshooting-output): export app, adapter and ad-unit test diagnostics for support.
