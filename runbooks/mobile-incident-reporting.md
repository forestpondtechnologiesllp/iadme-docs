# Mobile incident reporting

Implementation date: 2026-09-13. **Backend migration, API and worker deployed to staging and production as `release-2026.09.13-7eee307`.** Mobile source is committed/pushed; a new Android/iOS build and physical acceptance remain pending. See the [deployment record](../test-results/RELEASE_MONITORING_2026-09-13.md).

## Delivery and issue policy

Android/iOS queue observed ad failures, missed commercial opportunities, media observations, final API transport/server failures, authentication diagnostics and captured application errors. Routine ad loads/displays/impressions provide context but do not create issues. This does not automatically detect every visual defect, native process crash or swallowed exception; those still require reproduction or additional instrumentation.

The mobile secure-storage outbox uploads to `POST /observability/client-events` on startup, foreground resume, successful API activity and a 20-second foreground timer. Offline failures retry with backoff. The backend acknowledges only after its PostgreSQL transaction commits. A separate worker creates or updates GitHub issues; GitHub availability cannot hold up login, Feed or payments.

Issues are grouped by environment, source, release, platform, event and failure kind/error code. Generic exceptions additionally use their exception type when no more specific code exists. Feed/Trending remain distinguishable in samples and counters. Repeated occurrences update the same issue at most every five minutes; recurrence reopens a closed matching issue. Human notes outside the managed section are preserved. This produces supportable incident reports instead of a separate issue for every request.

The UUID of each uploaded batch is stable across lost acknowledgements. PostgreSQL deduplicates that UUID, while new observations retain their own counts. Older clients without an event UUID remain accepted, but their retransmissions cannot receive the same deduplication guarantee.

## AdMob support fields

- UTC capture/receive times, mobile version/build, platform, manufacturer/model, OS version, production/test configuration and ad unit.
- Google Mobile Ads SDK and Flutter plugin versions; mediation adapter class, ad-source names/IDs, available adapter error code/domain/message and latency.
- AdMob response ID when the SDK supplies one. Null/absent values mean unavailable and are never fabricated.
- Feed/Trending placement, consent eligibility, network transport and changes, per-surface request/load/display/impression/failure counters, fresh unused versus cached inventory, retry timing and recent placement timeline.
- Native media/image presence, view/media dimensions and SDK playback callbacks. A missing playback callback is an observation, not proof of a black frame. No automatic ad click or synthetic impression is generated.

No-fill means the SDK returned no eligible ad for that request. It does not independently establish an AdMob outage or defect. Compare request responses with the AdMob account's serving reports; mobile diagnostic counts are not unique users, unique creatives, billable impressions, revenue or an account-wide fill rate.

Names of advertisers and SDK adapter versions are not universally exposed by the current response API. The integration sends available source information; attach the exact dependency lockfiles and an Ad Inspector export if support needs fields the SDK omitted.

## Data limits and privacy

- Phone: 64 queued groups, up to 1,000 occurrences and four recent samples per batch; samples capped at 7,000 UTF-8 bytes. Seven-day queue lifetime. At capacity, routine info is evicted before warnings/errors, and dropped counts are recorded where possible. Storage failures do not block app use. Abrupt termination before persistence and prolonged offline/storage failure can still lose diagnostics.
- Backend: maximum accepted payload 48,000 UTF-8 bytes, last eight batches per incident, durable occurrence counts. Receipt and informational-incident retention is 30 days; warning/error incidents remain until operational cleanup. GitHub receives a bounded sample section.
- Tokens, passwords, cookies, contact fields, personal/advertising identifiers, precise location and raw request/response bodies are removed. URL query strings and SDK adapter mapping credentials are excluded. No automatic screenshots or recordings are uploaded.
- The endpoint is available before login. Client diagnostics are untrusted observations, not an authenticated audit trail. Environment is set by the server and clients cannot claim backend/worker source.

## Release prerequisites

1. Back up the database and apply `iadme-backend/database/migrations/20260913_add_monitoring_incident_outbox.sql` through the normal deployment process. It adds two tables and indexes; no existing user table is changed.
2. Deploy the matching backend API **and worker**. Configure `GITHUB_ISSUES_ENABLED=true`, `GITHUB_OWNER=forestpondtechnologiesllp`, `GITHUB_REPO=iadme-mobile` and a server-side token with issue access to that repo. Literal `false` disables delivery correctly. Never put the token in mobile build flags or source.
3. In a non-production environment, send one clearly marked diagnostic, verify HTTP 202 and the durable row, then verify issue creation/update, restart recovery and GitHub failure retry. Keep genuine production reports separate from synthetic tests.
4. Release the updated Android/iOS mobile build after physical acceptance. Build 38 cannot emit newly added SDK/media/network fields until users install the new build.
5. Inspect delivery lag and pending incidents after rollout. Disabling the GitHub flag pauses delivery while the backend keeps incident data. A backend rollback must leave the additive tables intact; avoid dropping them while a matching API or worker is still running.

The staging/production migration and backend deployment completed on 2026-09-13. Controlled staging delivery, deduplication, grouped update and worker restart recovery passed; synthetic verification issue #23 was closed afterward. No new mobile store build or release was performed in that rollout.

## Read-only operational checks

Run against the explicitly selected environment, using the normal secure database connection:

```sql
SELECT event_name, environment, release, platform, occurrence_count,
       delivered_count, issue_number, first_seen, last_seen,
       next_delivery_at, delivery_attempts, last_delivery_error
FROM monitoring_incidents
WHERE requires_issue AND occurrence_count > delivered_count
ORDER BY first_seen;

SELECT event_name, release, platform, count(*) AS groups,
       sum(occurrence_count) AS diagnostic_occurrences
FROM monitoring_incidents
WHERE environment = 'prod'
GROUP BY event_name, release, platform
ORDER BY diagnostic_occurrences DESC;
```

A worker sweep runs every 30 seconds, claiming at most ten groups with cross-worker leases. GitHub errors retry after 60 seconds with exponential backoff capped at one hour. `MONITORING_GITHUB_DELIVERY_RETRY`, `MONITORING_OUTBOX_SWEEP_FAILED` and `MONITORING_RETENTION_FAILED` identify delivery/storage maintenance trouble without exposing credentials. An API 503 leaves the new mobile batch queued for retry.

## Supporting documentation

- [AdMob response information](https://developers.google.com/admob/flutter/response-info)
- [Ad Inspector request tests](https://developers.google.com/admob/flutter/ad-inspector/test-ad-units)
- [iOS native video callbacks](https://developers.google.com/admob/ios/native/video-ads)
- [Official test inventory](https://developers.google.com/admob/flutter/test-ads)
