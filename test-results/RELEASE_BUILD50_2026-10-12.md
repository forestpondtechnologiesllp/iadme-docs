# iAdMe 1.0.4 build 50 — 12 October 2026

Release preparation and rollout authorized by the owner on 12 October. This
supersedes the earlier backend deployment hold. The owner will upload the AAB
and IPA manually; no store submission is authorized or performed by this task.

## Scope compared with the deployed release

Staging and production previously ran
`release-2026.09.22-build47-upload-cost`, digest
`sha256:9075c6928795cc485753f61719d18a72f3324b1aec2a007594dacded8af5fe9f`.
The latest archived mobile baseline was 1.0.4+48. Some baseline artifacts included
uncommitted source, so the backend comparison uses hashes of the actual running
compiled files: 38 added modules, 47 changed modules, no removed modules.

This release adds:

- **Listings / Local Market and Jobs (IADME-040):** dedicated Listings tab,
  photos-only posts with up to five images, one 10-Star publication charge,
  30-day visibility, account contact verification, category and locality
  discovery, structured job pay/employment/hours, editable enquiries,
  seller ratings after eligibility, related listings, comments, moderation,
  Feed-style reactions/ScrollScore, bookmarks and listing Targets.
- **Photo posts (IADME-035):** 1–5 images for one 5-Star fee, mixed Feed/Trending,
  private originals, WebP display/thumbnail variants, atomic publication,
  bounded retries, cancellation/expiry/failure refunds and legacy video-client
  compatibility. Production transformation uses the existing Lambda, capped at
  two concurrent executions; staging uses local Sharp.
- **Official Inbox (IADME-024):** separate subject/full-message screens, unread
  counts, registration welcomes, admin messages with frozen audiences, email
  delivery/retry state, and compact ads sharing the Comments native units.
  Listing/video/photo publication produces an idempotent owner Inbox receipt
  and queues email when an address is available. No old posts are backfilled.
- **Messages (IADME-026):** foreground/resume reconciliation, cursor pagination,
  direct conversation lookup and idempotent retry IDs.
- **Locality notifications (IADME-029):** an opt-in hourly digest of newly ready
  nearby videos, with a per-user/hour delivery limit and no historical backfill.
- **Feedback and playback reporting (IADME-030/031):** monthly feedback/store
  action, per-reel playback issue form and protected admin views. Closing a
  published listing can request native review subject to the shared cooldown
  and store/OS availability.
- **Shared links (IADME-038):** install-first video/photo landing pages with app
  opening, explicit store choices and link replay; device deferred-link
  acceptance remains separate.
- **Profile and permissions (IADME-039/040):** three-column Profile navigation,
  direct Inbox/Messages, updated History and Targets, Video/Photo/Listing upload
  selection, and tracking/ad-consent/notification prompts after successful
  login/session restoration and biometric unlock.
- **Advertising and reel shop:** Meta mediation integration, native Comments
  and Inbox placements, bounded shared reel-product catalog, image/Shop stamps,
  stable shuffled rotation, image failure/expiry hiding, administrator inventory
  tools, supplied-image imports and one-calendar-month retention. The separate
  URL preparation worker is limited to 0.25 CPU / 192 MiB. New remote catalogs
  start empty and disabled; development product data is not promoted.
- **Reliability:** upload-price refresh and retry handling, exactly-once financial
  refunds and wallet receipts, notification seen/read separation, background
  lease guard correction, telemetry recursion protection, explicit boolean
  configuration parsing, and onboarding text aligned with current navigation.

Existing next-two reel preparation remains prioritized. The approximately
200 MB allowance is disk cache, and extra preparation remains bounded by network,
power, storage and OS scheduling. Background execution is not guaranteed.

Planning-only work is excluded: creator monetization/profile pages, more
countries, the SaaS/instance plan, PostgreSQL scale/search projects, iCube content
and comments expansion, premium-unlock counts, and card-based onboarding. Tracker
closure does not imply those features shipped or a complete dependency upgrade.

## Deployment

- Backend source: `7bcd7386c10dfd6080c7cf0b8319993bb945eb1d`.
- Mobile source: `986d55f` (version `1.0.4+50`).
- Website source: `325349d`.
- Image: `ghcr.io/forestpondtechnologiesllp/iadme-api:release-2026.10.12-build50-7bcd738`.
- Immutable digest: `sha256:2226e98cf6c990c14d89bba6c495ac9834c3c5bbebd2194c3f0b58825d4a0996`.
- Both API/worker environments deployed with the same digest, plus independent
  affiliate workers. Staging compiled-file hashes exactly matched local release
  output. GitHub backend CI run `38213595507` succeeded for the exact commit.
- Separate private listing buckets were created using the existing CloudFormation
  template. No runtime IAM permissions were broadened; the existing role already
  had bucket access and the scoped production Lambda invocation policy.
- The existing media bucket is shared by staging and production. Its current
  CloudFront allow rule was preserved, with an added deny for private
  `photo-sources/*` originals. A two-day source-only lifecycle rule is prepared
  but **not applied, pending explicit owner approval after automatic review
  rejected the deletion policy**. Normal worker cleanup remains active.

## Migrations and rollback

All 13 migrations were first rehearsed against an isolated PostgreSQL 18 instance
loaded with production schema only. They were then applied to staging, verified,
and applied to production in a single transaction per environment, with bounded
lock timeouts, advisory locking and filename/SHA-256 records in
`release_schema_migrations`.

1. `20260927_official_messages.sql`
2. `20260927_official_inbox.sql`
3. `20260927_refresh_welcome_inbox_copy.sql`
4. `20260928_engagement_and_locality_digest.sql`
5. `20260929_photo_posts.sql`
6. `20260930_notification_seen.sql`
7. `20260930_video_upload_refunds.sql`
8. `20261002_local_listings.sql`
9. `20261002_listing_engagement.sql`
10. `20261002_publication_inbox.sql`
11. `20261007_affiliate_reel_products.sql`
12. `20261007_affiliate_reel_assets.sql`
13. `20261008_affiliate_reel_monthly_retention.sql`

Custom-format database backups, successful `pg_restore --list` output, checksums,
and prior Compose/runtime files are private on the server:

- Staging: `/opt/iadme-staging/backups/release-20261012-build50/`.
- Production: `/opt/iadme/backups/release-20261012-build50/`.

Migration invariants were unchanged: staging 7 users / 26 videos / 41 ledger
entries / 860 net Stars; production 21 / 57 / 120 / 11,757. Later application
activity and separately recorded synthetic staging tests are outside that
before/after comparison.

Each private backup directory also retains `remote-release.py` and the applied
`migrations.sql`, so recovery does not depend on `/tmp`. Run the helper with
`python3 <backup-directory>/remote-release.py staging rollback` or `prod rollback`
on the deployment host: restore saved configuration and old API/worker image,
remove only the new affiliate container, and retain additive schema. A database restore is a separate, destructive recovery operation and is
not an automatic rollback step.

## Verification

- TypeScript typecheck/build and GitHub CI passed.
- Backend unit suite: 151 passed, 7 opt-in skips. All 68 selected PostgreSQL
  integration tests passed across the final main run and corrected auth-fixture
  rerun, including money races/refunds, photos/Lambda failure handling, listings,
  receipts, notification state, product retention and social-account races.
- Flutter analysis: no issues. Full suite: 554 passed, 4 skips, one outdated
  onboarding expectation failed. The expectation and a stale History sentence
  were corrected; all three onboarding tests then passed (555 unique tests
  passing overall). Dev/staging/prod launcher preflights passed.
- Website: 9 tests and JavaScript syntax checks passed.
- Real staging HTTP/worker/private-S3 acceptance passed: 1/5-photo listings,
  signed reads and unsigned 403, 30-day visibility, one 10-Star debit each,
  enquiry/message retry idempotency, Targets, a five-photo post, legacy-client
  exclusion, cancellation refund once, and three publication Inbox receipts.
  Synthetic accounts were disabled, content hidden and exact media keys deleted.
- Standalone live Lambda test passed with the production worker role: two valid
  WebP outputs via CloudFront, original 403, version 1. Only disposable objects
  were created and then removed; no feed post or wallet change was made.
- Staging API/worker/affiliate worker health: running, zero restarts and no
  logged errors at both release and subsequent soak checkpoints. Production
  endpoint and legacy-client smoke checks also passed; all three
  production services are running with zero restarts and no logged errors, and
  their compiled JavaScript hashes exactly match the release. The temporary
  system test identity was removed; no production post, purchase or wallet test
  entry was created.

Local heavy tasks run sequentially with lower process priority. Docker compilation
was capped at two CPUs and 3 GiB with one BuildKit operation at a time; Xcode
archive jobs are capped at two, and Android uses two Gradle workers/JVM processors.
Temporary test containers were stopped before native builds. Dev services remain
running. Available macOS thermal checks reported no warning; this is not a
physical temperature measurement or a mobile-device thermal certification.

Artifacts, checksums, raw logs, prepared lifecycle policy and operational scripts:
`artifacts/releases/1.0.4-build50/` in the workspace. Private database/schema
evidence is excluded from Git. Both signed packages are built and verified as
`app.iadme.mobile`, version `1.0.4`, build `50`:

- Android: `iadme-android-1.0.4+50.aab`, 80,909,421 bytes. Bundle validation,
  expected upload-key certificate, release marker and non-debuggable manifest
  passed. The bundle requests 16 KB ZIP alignment; all 12 bundled 64-bit native
  libraries passed LOAD and RELRO layout checks.
- iOS: `iadme-ios-1.0.4+50.ipa`, 39,459,475 bytes. Strict signature, App Store
  provisioning, production push, package/version/build, release marker and native
  Facebook-configuration checks passed. The distribution profile expires
  9 June 2027.

SHA-256 checksums:

```text
aa16ac806a446a66a0a77ed78c60e6186c5d30ab7899c1dcc853c408b5115184  iadme-android-1.0.4+50.aab
9fee84135a2c4875635a3b77b63093a4ae3b46c19b91ed17fe1262e808da810e  iadme-ios-1.0.4+50.ipa
```

The first Android attempt hit a Gradle distribution-download timeout. The exact
Gradle 9.1.0 archive was downloaded from the official server, checksum verified,
and cached; the checked production build then succeeded without dependency
version changes. Flutter warns that several plugins need built-in Kotlin
migration for future Flutter releases; the present build succeeds.

The initial additional RELRO heuristic incorrectly required every protection
range to end at a 16 KB boundary. Inspection of all flagged segments showed each
covers its entire LOAD segment; the verifier was corrected to follow the explicit
whole-LOAD exemption in [Android's Bionic loader](https://android.googlesource.com/platform/bionic/+/android16-qpr2-release/linker/linker_phdr_16kib_compat.cpp).
The final checks passed without changing binaries. Alignment validation follows
[Android's page-size guidance](https://developer.android.com/guide/practices/page-sizes)
and is structural validation, not a substitute for physical-device testing.

The matching website update is published; the served reel-shop admin JavaScript
SHA-256 matches the committed file exactly.

Physical-device playback/background behavior, real billing, provider sign-in,
permission/store review acceptance and actual mailbox/push delivery remain
manual acceptance checks. Automated, simulator or backend results do not replace
them. Store upload is manual.
