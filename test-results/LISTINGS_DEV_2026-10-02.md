# Listings dev activation — 2 October 2026

The owner explicitly authorized completion in **dev only** once the laptop was
free. Staging, production, remote website publication and app-store submission
were not performed. All source remains in the existing working trees.

## Active dev environment

- Local Docker API: `http://127.0.0.1:3000`; worker running alongside it.
- PostgreSQL and Redis stayed running. API/worker restarted with the same new
  locally built image; no GHCR push.
- CloudFormation stack: `iadme-listings-dev`, AWS `ap-south-1`.
- Bucket: `iadme-listings-dev-listingbucket-ob7jqjtgcslo`, separate from public
  reel media, with all four public-access blocks, AES256 encryption and HTTPS-only
  access. GET/HEAD CORS permits `http://localhost:8080`. Existing runtime storage
  permissions were used; no shared IAM policy was changed.
- `LISTING_MEDIA_BUCKET` is saved in ignored `services/api/.env.dev`, loaded by
  both dev containers.
- Admin and policy pages: `http://localhost:8080/admin.html`. The local console
  now defaults to the local API. Admin authorization is still required.
- iOS simulator app: dev `1.0.4+49`, built with the checked launcher, installed and
  launched on the saved iPhone 17 simulator. The previous saved auth session was
  restored after the synthetic native test.

The owner clarified that this phase is **dev testing only**, with no APK, IPA or
app-bundle preparation yet. An unnecessary Android APK attempt was stopped at
the owner's request, including its build-specific Gradle daemon. No Android
package was produced. The simulator `.app` is the debug app needed for dev tests;
no IPA or store bundle was built. Prepare distribution packages only after dev
acceptance and a later owner instruction.

## Database changes

A private local custom-format backup was taken before migration:
`artifacts/dev/listings-20261002/pre-listings-dev.dump` (mode 0600). It contains
dev data; do not commit or share it.

Applied to local `iadme` only:

1. `20261002_local_listings.sql`.
2. Existing `20260930_notification_seen.sql`, after native navigation exposed
   missing `seen_at` and a 500 from `/notifications/unseen-count`.
3. Existing `20260913_add_monitoring_incident_outbox.sql` and
   `20260921_monitoring_signal_counts.sql`, missing prerequisites behind a 503
   from client monitoring.
4. Existing `20260930_video_upload_refunds.sql` unique-index prerequisite.

Notification and message badge endpoints subsequently returned 200; the synthetic
informational monitoring check returned 202.

## Verification

- Backend Docker build / TypeScript compilation passed.
- Full Flutter analysis: **no issues**.
- **97 focused Flutter tests passed**, covering navigation labels/unread badges,
  photo/video upload selection, ad-aware feed reconciliation, photo posts,
  messaging themes, notification routing and sign-out.
- Earlier **10 isolated backend lifecycle tests passed**, including blocked
  relationships, OTP attempt limits, expiry/holds and wallet races.
- The native fixture server is reproducible with
  `services/api/tests/listings.dev-fixture-server.py`; it binds loopback only and
  must be stopped after testing.
- Real dev acceptance (`services/api/tests/listings.dev-smoke.cjs`) passed:
  - both contact confirmation endpoints with synthetic OTP challenges;
  - signed five-photo and one-photo uploads into the real private dev bucket;
  - running worker transformed uploads to WebP;
  - administrator approval, exactly one 10-Star debit despite duplicate approval,
    and exactly 30 × 24 hours of visibility;
  - signed media returned 200; unsigned media returned 403;
  - worker-generated related listings;
  - enquiry idempotency and editable default with no automatic message;
  - buyer/seller messages, rating unavailable before 24 hours and available after
    fixture timestamps crossed that boundary; listing score updated;
  - public comments, saved listings and publication in-app notifications;
  - real worker purged expired fixture media/details and S3 HEAD confirmed every
    fixture source/variant was deleted.
- Native iOS acceptance (`integration_test/listings_dev_smoke_test.dart`) passed:
  three-column Profile with Inbox/Messages/Wallet first, no Profile seller score,
  Messages → Listing enquiries, listing photos/price/seller rating, editable
  enquiry draft, verified category composer, Listings navigation and the
  Video/Photo/Listing upload arch.
- JavaScript syntax and Git whitespace checks passed; the local admin panel
  rendered against the local environment with no browser console errors.

Native screenshots are saved in `artifacts/dev/listings-20261002/`:
`profile-dev.png`, `listing-dev.png`, `enquiry-dev.png`, `composer-dev.png`.
They use explicitly synthetic accounts/content. The fixture server was stopped,
its short-lived session file removed, test photos purged and test accounts
disabled. Existing users' wallets were not changed.

## Scope and remaining public-launch checks

Listings are active in dev. The admin queue remains manual: a post becomes visible
and is charged only after approval. No demo offers remain publicly visible.
The configured dev email/SMS providers are Brevo/MSG91; this pass did not send
real email/SMS, make a store purchase or validate push delivery on physical
devices. Native tests disabled ad inventory; the separate ad-pager regressions
passed. OS background execution remains opportunistic, with the existing next-two
video path and bounded cache unchanged.

Public-launch legal/retention wording, store billing acceptance and physical-device
testing remain separate release checks. No disclaimer is represented as waiving
mandatory platform obligations.


## Follow-up: automatic publication and category-first form

Owner corrections supersede the earlier approval workflow above. Category and
details remain first; there is no automatic photo-picker step. Verification now
lives at Profile → Account → Verify email or phone, and an unverified advertiser
sees that path immediately on opening Listing. The form uses Sub-locality / City /
State and New / Used, “Publish · 10 Stars”, and the 30-day info note.

The API and worker were rebuilt locally and restarted with image
`sha256:bf6219c3cb2548399528501de42531b2f20197e99f314c97377439d06445a158`.
No additional migration was necessary. Local API health confirms dev and connected
PostgreSQL, Redis and BullMQ. Existing data volumes stayed running.

Follow-up checks:
- TypeScript typecheck, Docker compilation, full Flutter analysis and admin
  JavaScript syntax passed.
- 13 isolated backend tests passed, including automatic publication/worker retry,
  no negative wallet balance, later admin review preserving the fee and expiry,
  lost verification preventing publication, and automatic recovery of legacy
  pending listings using already processed photos.
- 3 new listing widget tests passed: immediate verification gate, category-first
  jobs with no condition/photo auto-open, New/Used and all three area choices.
  10 existing upload/navigation regression tests also passed.
- Real dev HTTP/private S3/worker acceptance passed again with automatic
  publication of five-photo and one-photo listings, publication retry plus later
  admin-review retry, exactly one 10-Star debit each, 30-day visibility, private
  media, related listings, enquiries, messages, ratings, comments, saved items and
  in-app publication notifications. Only synthetic accounts/wallets were used.

- Updated native iPhone simulator acceptance passed: Profile → Account → Verify
  email or phone opens both verification controls; an unverified account is
  immediately gated before the listing form; verified accounts see category
  first. Existing listing/enquiry/rating and upload-arch checks also passed.
  The refreshed composer screenshot was visually checked.
- Synthetic fixture media were purged by the worker, S3 deletion confirmed,
  fixture accounts disabled and the private session file removed. The temporary
  port-8081 fixture server was stopped. Previous simulator credentials were
  restored by the test. Actual SMS/email delivery was not exercised.

The owner can continue dev acceptance in the regular simulator app. No APK, IPA,
AAB, staging/prod deployment or remote website publication was performed in this
follow-up.

The regular checked dev run was relaunched successfully on the iPhone 17
simulator, reached `/home` against `http://127.0.0.1:3000`, and was detached with
the app left running. API and worker remain running for owner testing.


## Follow-up: shared Feed presentation and compact details

Owner changes: photos-only confirmed, ScrollScore retained, complete Feed action
rail, compact title/price/rating/Enquire, full details in a sheet, Listings heading,
and Profile order Inbox/Messages/Wallet, Account/History/Settings, Help/i³ AI/Privacy.

The source reuses `FeedVideoHeader`, `PhotoCarousel`, `PostCaption`,
`VideoActionRail`, `VideoEngagementSheet` and the extracted `PostIdentityRow`.
Public Feed/Trending photo caching remains the default. Listings select memory-only
signed-photo rendering, and do not enter the video/Star reward path.

The reported red screen was reproduced in a widget test as a disposed
`TextEditingController` while the focused filter modal was reversing its route
animation. Controllers now belong to the sheet/dialog State until unmount.
The same ownership fix covers listing report dialogs. Photo double-tap handling
is confined to the media, so it does not delay See more and other controls.

Dev database backup `artifacts/dev/listings-20261002/pre-engagement-dev.dump`
(mode 0600) was taken before applying `20261002_listing_engagement.sql` to local
`iadme`. The migration adds reaction, shared and viewed_at while preserving legacy
likes. API/worker are running the local image config
`sha256:af0d1341ab550aa77da30a5ffb299cb0b4616df8145edbca1a91c0d40507ebd9`.
The API health check reports dev with PostgreSQL, Redis and BullMQ connected.

Verified before native acceptance:
- TypeScript typecheck, local Docker compilation and full Flutter analysis pass.
- 16 isolated backend tests pass. New tests cover exclusive reactions, unique
  views/shares, Feed score weights, report deductions, blocked engagement people,
  expired access, no reward debit/credit and optional approximate viewer distance.
- 97 distinct focused Flutter tests pass across the shared Feed header, action
  rail accessibility, captions, photos, Feed/Trending ad pager, navigation,
  filtering, upload refinements, listing composer and compact listing sheet.
  An additional 28 playback lifecycle, distance formatter and next-two preloader
  checks passed. The five listing tests include focused-filter closure and 320×568 light/dark
  layouts at normal and 2× text scale. Long details scroll within the sheet.
- Real dev HTTP/private S3/worker acceptance passes using synthetic accounts:
  1/5-photo auto-publish, private signed access, related worker, 10-Star charge
  once, 30-day expiry, enquiry messages, delayed ratings, comments, saved items,
  exclusive like/dislike/target, unique view/share counts and ScrollScore.

Native iPhone simulator acceptance passed: all nine Profile tiles in order,
verification path, messages submenu, shared header/rail and compact caption,
full details sheet, focused filter apply/clear, actual like/dislike taps and
horizontal photo swipe, Enquire draft, category-first composer and upload arch.
The earlier test timing failure after filter reload was corrected to wait for
the rebuilt screen; the app reported no Flutter exception during the passing run.
Profile, listing and details screenshots were visually inspected.

The test restored the prior simulator credentials. Worker cleanup purged all
synthetic listing photos/details; real S3 deletion was confirmed, fixture
accounts were disabled, the private session fixture removed and the temporary
loopback server stopped. Local API/worker and admin preview stay running.

Sharing currently uses an installed-app custom link; no public HTTPS listing
landing page or remote website rollout is included. No distribution package,
staging or production changes were made. Physical-device, real OTP delivery,
external share delivery and push acceptance remain separate launch checks.


The regular checked dev app was relaunched and opened on the Listings tab using
the restored user session. Visual inspection confirmed the owner's existing
flower listing with the compact Feed rail/caption, ScrollScore, area pill, seller
rating and bottom navigation, with no red screen. The qualified view advanced
ScrollScore from 0 to 1. The app remains running for owner testing.

## Job terms and closure feedback follow-up (17:27 request)

Implemented a single structured job Pay section (fixed/range/to be discussed),
eight pay periods, seven employment types, separate work schedule, and fixed or
flexible working hours with Day/Week units. Application closing date and duplicate
salary text are removed from the UI. Pay wording is shared by the compact reel,
full details and share text. Legacy immutable job drafts remain retryable.

The close endpoint now reports only the first published-to-closed transition,
under a row lock. A native `in_app_review` request follows a successful closure,
with a persistent 120-day cooldown and no dependence on deal success/sentiment.
No prompt for draft disposal, failed/repeated closes or unavailable stores.
Prompt failure never invalidates closure. Existing feedback prompts coordinate
through the same last-attempt record to avoid asking again soon afterward.

Validation completed:
- TypeScript typecheck and dev Docker compile pass.
- 19 isolated backend integration tests pass, including structured job values,
  invalid/reversed ranges, impossible hours, permitted units, immutable draft
  retry, legacy compatibility, closure ownership and concurrent close requests.
- 19 focused Flutter tests pass: job-mode changes and hidden-field omission,
  numeric validation, pay wording, details-sheet grouping, composer choices,
  compact Feed layouts/filter closure, store cooldown and failure handling,
  and existing feedback UI.
- Focused Flutter analysis of changed features and tests: no issues.
- Local API/worker updated to image config
  `sha256:b0c83bd05749333413ce63b94aa0d0077e84e2c38edae2ba3cf8baef06a6c938`.
  `/health` reports dev, PostgreSQL, Redis and BullMQ connected.

Native build initially found missing files in existing CocoaPods SDK caches
(Facebook, Google Mobile Ads and UMP). Repair re-downloads the same locked SDK
versions; plugin registration remains managed by Flutter/CocoaPods.
Store review submission is not part of dev testing. The OS/store decides whether
a native prompt appears; Android Play-distributed acceptance remains unverified.

Native follow-up completed: CocoaPods repaired the missing cached SDK files at
unchanged locked versions. The only new Podfile.lock dependency is in_app_review.
The checked dev launcher completed the iPhone simulator build (94.8 seconds)
and launched against `http://127.0.0.1:3000` using the existing signed-in session.
On-device UI checks confirmed category-first Jobs, fixed/range/discussed pay,
minimum/maximum fields, all eight pay periods, all seven employment types,
independent full-time/part-time/flexible work schedule, and Day/Week hours units.
The pay and employment/hours layouts were visually inspected. No Flutter error
was observed during these checks. The simulator remains on the Jobs composer;
no listing was published or closed and no review was submitted in this UI check.
The native review plugin is generated/registered successfully; prompt eligibility
and failures were exercised in automated tests, not by rating the app in dev.
Dev API/worker remain running. No APK, IPA, AAB, staging or production rollout.

## Direct Profile navigation, listing Targets and publication mail (20:45 IST)

Implemented direct Inbox/Messages entry, preserved Wallet, removed redundant
History listing entries, and added targeted/bookmarked listings to Targets with
pagination, deduplication, correct listing navigation and reload on return.

Publication now atomically creates an owner-only official Inbox receipt and
queues email through the existing Brevo worker for listing, video and photo.
Draft/failure paths do not send publication mail; repeated ready callbacks do not
create extra receipts. No previous posts are backfilled.

Validation:
- TypeScript typecheck passed.
- 21 listing integration tests passed in isolated iadme_listing_test_20261002,
  including new receipt ownership/idempotency and target/bookmark visibility.
- 6 publication integration tests passed: concurrent video-ready callbacks,
  stale job guard, photo processing, failure/no-email behavior, provider retry,
  missing recipient email, HTML escaping and transaction rollback/retry.
  Brevo transport was mocked; no mail was sent to real recipients during tests.
- 17 focused Flutter tests passed across listing feed, targets, jobs composer,
  pay labels and store-review eligibility. New row tests caught a Material surface
  assertion; the surface was fixed and both new tests rerun successfully.
- Focused Flutter analysis: no issues.
- Local dev backed up to artifacts/dev/listings-20261002/pre-publication-inbox-dev.dump
  (mode 0600), then 20261002_publication_inbox.sql applied successfully.
- Updated local dev image config:
  sha256:e782f2615f169929d3539d85ada5add394b6b320f4954b0a84518083de970250.
- Actual mailbox receipt remains to be verified by publishing with a real dev
  account. Dev continues using its existing Brevo configuration. Email processing
  retries are bounded; ambiguous provider acknowledgement can still duplicate
  an external email, while Inbox records remain idempotent.

Simulator acceptance completed with the regular signed-in dev account:
Inbox opens its message list with one tap; Messages opens directly with
All/Listings/Unread filters. History shows Targets, Uploads, Unlocked premium
videos and Activity. The user's existing “Nice Flowers for Sale” target is
visible alongside the existing reel target; opening it reaches the listing reel
with the target still selected. Returning refreshes the list without an error.
Checked dev launcher rebuilt iOS Simulator in 39.1 seconds. App remains on Targets.

Two pre-existing dev startup problems surfaced during the worker check:
- Both .env and .env.dev pointed MediaConvert at iadme-staging-transcode and
  iadme-staging-mediaconvert-events (us-east-1), while AWS_REGION is ap-south-1.
  Polling failed with SignatureDoesNotMatch. A read-only query for dev-prefixed
  queues was denied (AccessDenied); no AWS resource was created or changed.
  .env.dev now sets MEDIACONVERT_EVENTS_ENABLED=false and
  MEDIACONVERT_SUBMISSIONS_ENABLED=false to avoid staging access. This intentionally
  leaves live dev video processing unavailable until a separate dev pipeline is
  configured. Publication receipts for video are implemented and integration
  tested, but full live dev video-upload/email acceptance remains blocked by this
  existing configuration. Listing/photo processing is unaffected.
- Sentry's recursive telemetry redactor overflowed on cyclic objects and flooded
  startup logs. Added circular-reference/depth guards, preserving secret
  redaction and input immutability. All four auth/redaction tests and TypeScript
  typecheck passed. Sentry remains enabled.

Final dev image config sha256:f2e3c74aa8be309f72e200dad90ef90b6507e88a3a9c307d55292c6d356e276a;
manifest sha256:87b015a355d82444b73b374537ef50807595cbc129eaebc303242f8717c4acc6.
No staging/production deployment and no APK/IPA/AAB generated.

The final runtime check also exposed Zod coerce.boolean treating the string
false as true for MediaConvert switches. Both switches now parse true/false
explicitly; three parser tests and another TypeScript check passed. The worker
was briefly stopped during the corrective rebuild to avoid staging-linked polls.
Final runtime verification: /health reports dev with PostgreSQL, Redis and BullMQ
connected; API and worker both run the final manifest above. Worker logs confirm
MEDIACONVERT_EVENTS_WORKER_DISABLED, with zero staging-queue polling failures,
zero telemetry stack overflows and zero official Inbox/email sweep errors.
