# Local Market UI decisions and prototype — 2 October 2026

Status: implemented and activated in dev on 2 October 2026. Private storage, database migration, API/worker, local admin/policy pages and mobile are active or built for dev. Native iOS acceptance passed. Staging and production remain untouched. See the dev activation record below.

## Owner decisions

- Profile becomes a grid with exactly three icons per row and a small label below each icon. First row: **Inbox, Messages, Wallet**. Tapping an icon opens its submenu.
- **Listings replaces Messages in its existing bottom navigation position.** Messages moves into the profile grid. Keep unread indicators on Messages and Profile so replies remain discoverable.
- Tapping the bottom **+** opens three circular choices arranged in an arch: **Video, Photo, Listing**. Video and Photo route to the existing composers with their current UI.
- Listing creation starts with category selection/identification and then asks relevant category-specific questions. If category is inferred from media/text, show it for confirmation and allow correction.
- Listing feed uses the existing full vertical feed pattern, filters, age pill, reaction rail and seller name/avatar. **Listings contain photos only, up to five.** This replaces the earlier possible video option.
- **Seller rating replaces scroll score on listing reels only.** The screenshot follow-up explicitly removes the rating score and Seller ratings tile from Profile. Do not add a seller-rating summary to the public profile. Rating collection may still occur in eligible listing conversations. There is no marketplace scroll score or scroll-star accrual.
- Listings disappear after **one month**. This supersedes seven-day visibility. Proposed implementation/UI interpretation: **30 × 24 hours after publication**. Calendar-month semantics are not yet settled; show “30 days” only if that interpretation is accepted.
- Add a worker for related listing/feed selection and preloading support.
- **Enquire** opens the advertiser's message conversation with the default question **“is this still available”**. Replies follow the existing notification flow.
- Existing decisions remain: every listing costs 10 stars; both phone and email must be verified before requesting publication; iAdMe provides listing/contact services without taking transaction commission or guaranteeing deals.

## Current screenshot-based design

The owner liked the profile direction and selected Grouped tiles, then supplied three current Profile screenshots. The revision keeps the current monochrome appearance, notification bell, edit action, avatar, name/email and **Followers / Following / Uploads** counts. Remove the earlier invented profile seller score, verification footer and extra rating tile.

Two variations are available: **Grouped tiles** with the current centered identity layout, and **Compact header** with the same elements in a shorter left-aligned identity row. Both retain exactly three tiles per row:

| Row | First | Second | Third |
| --- | --- | --- | --- |
| 1 | Inbox | Messages | Wallet |
| 2 | i³ AI | History | Settings |
| 3 | Help | Privacy | Account |

Each tile opens a submenu. The grouping below preserves all functions visible in the screenshots:

- **Inbox:** official updates from iAdMe.
- **Messages:** personal messages and marketplace enquiries; no direct seller-rating entry on the profile grid.
- **Wallet:** Wallet (balance and rewards), Wallet history, Purchase history (payments and refund status).
- **i³ AI:** i³ — iAdMe In-house AI; official insights, milestones and support.
- **History:** Targets, Uploads, Unlocked premium videos, Activity. Uploads also provides access to My listings as the planned marketplace extension. The Uploads statistic opens the same uploads submenu.
- **Settings:** Video storage, Appearance, Haptic feedback, Notification preferences.
- **Help:** iAdMe Tour, Help/FAQs, Rate iAdMe (app-store rating), Contact. App-store rating remains separate from seller feedback.
- **Privacy:** Ad Privacy Choices, Blocked users.
- **Account:** Connected accounts, Change password, Biometric unlock as shown in the screenshot, Sign out, Delete account. Keep delete distinct and red. Its presence in this design records the supplied screen; any prior backlog cancellation of biometric work must be reconciled before implementation, not silently reversed here.

Correct bottom-nav order from the screenshot: **Feed / Trending / Upload (+) / Profile / Listings**. Profile stays fourth. Listings replaces Messages in the fifth/rightmost position. The original prototype mistakenly swapped those last two positions; this revision corrects that. Keep the rounded floating bottom-bar style.

Seller-rating display is confined to the listing reel and its review details opened from the listing. Collection remains in eligible listing conversations, not on the Profile page. The upload arch and listing flows remain available for design review.

Current prototype source (conversation preview only): `/Users/saisrikrishnakumaradavikolanu/.codex/visualizations/2026/09/30/01a0f1d0-4ad3-7851-a772-d1c3e1d5a71b/iadme-profile-current-elements.html`.

The earlier prototype is retained at `iadme-market-designs.html` only as a historical concept; it is superseded for Profile and bottom-nav order. Profile content in the current preview follows the owner's screenshots. Listing names, ratings and conversations remain illustrative; the chair is generated sample imagery. No prototype action changes an account, sends a real message or posts a listing.

## Listing request flow

1. **Category and eligibility:** select/confirm an allowed category; verify both phone and email before requesting publication. Explain missing verification without exposing contact details. Link to allowed/prohibited listings.
2. **Category-specific questions:** common title and description, then tailored details. Furniture: condition, material, dimensions, price, collection. Electronics: brand/model, condition, defects, price and included accessories. Jobs: employer, role, location/work arrangement, employment type, compensation, hours, requirements and deadline; prohibit applicant fees. Services: service type, coverage, price basis, availability and included/excluded work. Do not imply these advertiser answers are certified by iAdMe.
3. **Photos and locality:** maximum five photos; choose cover and order; no listing video. Strip location metadata and show locality rather than an exact public home address. Reuse suitable photo controls without changing the normal photo-post composer.
4. **Review and request:** show 10-star listing fee, proposed 30-day visibility, explicit rules acceptance and expected moderation state. Keep server-side atomic/idempotent charging and rejection/refund rules from the main plan. No charge for simply exploring the prototype or saving a draft.

## Seller ratings — recommended placement and meaning

**The enquirer rates the seller; a seller never supplies their own rating.** The owner's phrase about asking the seller for a rating is interpreted as requesting a seller-rating collection flow. This interpretation should remain visible in future implementation discussions.

- Show aggregate rating and count on every listing in the space used for scroll score. Tapping opens review details from that listing. Do not show the rating on Profile or add a Seller ratings tile. Use “No ratings yet” for a new seller rather than zero stars.
- Primary request location: an inline card in the **listing-linked conversation**, after meaningful two-way interaction. Proposed timing: 24 hours after the first seller reply, on the enquirer's next visit, with “Rate seller” and “Not now.” Do not prompt on first Enquire tap or on a one-way unanswered message.
- Keep “Rate seller” in the conversation menu when eligible, so a dismissed prompt can be revisited. Do not add a Profile → Seller ratings entry.
- A seller marking a listing Sold/Closed may make eligible conversations easier to find, but it must not assert that a particular enquirer purchased anything or send rating prompts to every contact.
- Ask “How was your conversation with this seller?” and allow 1–5 stars, optional communication tags and an optional short review. Frame the public score as **seller interaction feedback**. iAdMe does not verify purchases or certify product quality.
- Proposed anti-abuse rules: no self-rating; require a real eligible listing conversation; one active review per reviewer/seller pair (editable rather than stacked across repeated posts); rate limits; suspicious-account/review checks; report/appeal route; seller may reply or report but cannot remove an unfavourable review. Do not reward reviews with stars or show prompts only to likely positive reviewers.
- Keep rating records and permitted enquiry evidence under an explicit retention policy. Expiring/deleting a listing should not reset the seller's rating history. Access/deletion requests and reported content still need policy handling.
- A rating reminder notification is optional and separately configurable. No new email/SMS campaign is approved. Avoid repeated reminders and suppress them after dismissal/submission as defined in the final policy.

## Enquiries and notifications

Recommended behavior: tapping Enquire opens an editable draft with exactly “is this still available”; the user taps Send. No automatic outgoing message is authorized by the default question alone. Existing conversation and listing context should be reused, with duplicate-thread/send prevention.

Send the normal new-message event/push only after a successful send. Keep listing context in deep links; tapping a reply notification opens the correct thread despite Messages moving into Profile. Respect block rules, quiet hours/preferences and lock-screen preview privacy. Avoid duplicate notifications from push, polling and foreground sync. Expired listings show an unavailable stub in existing conversations, while new enquiries from their public entry are closed.

## Related listings worker and preloading

Interpret “related feed” initially as relevant **marketplace listings** by locality, confirmed category, availability and permitted relevance signals. Mixing ordinary reels into marketplace recommendations remains undecided.

- Backend worker prepares/ranks eligible candidate listing IDs and updates them on publication, category/location changes, closure, expiry, blocking and moderation. Apply visibility checks again when serving the feed; cached recommendations must not revive expired or removed content.
- Client prefetches the next two listing cover photos first, then bounded extra carousel images only when useful. Keep stable pagination, deduplicate IDs and cancel stale prefetch work on filters or navigation changes.
- Reuse the existing feed/cache architecture where appropriate; do not change the normal video path, add background players or displace its next-two fast path.
- Use explicit image byte/count/age budgets, metered-network and low-power preferences, and cache invalidation at listing expiry. Do not assume the existing approximately 200 MB video cache is an image-cache entitlement.
- Server ranking work and client preloading are separate responsibilities. Phone OS background execution remains opportunistic, never guaranteed.
- Validate stale caches, filters, expired objects, logged-out/blocked users, low storage, failed fetches, background limits and zero marketplace scroll-score events.

## Expiry and storage update

Public disappearance is now one month, with 30 days proposed for implementation. Existing staged-retention reasoning still applies. Proposed ordinary-media purge remains **within seven days after expiry**; under the 30-day interpretation this is by day 37, not day 14. It is not yet an approved purge schedule.

Do not silently remove existing messages, user accounts, contact verification, review history or accounting entries with a listing. Retain only justified minimal records for defined periods, and implement legal holds and backup deletion rules as previously recorded. Renewal/repost is an explicit new 10-star request, proposed; edits do not extend the live window automatically.

## Checks performed on the design

- Fragment and JavaScript syntax read back and validated.
- Local interaction logic checked for the original listing flows and both revised profile variants. The revision checks screenshot identity/counts, all submenu entries, first-row order, correct bottom-tab positions, upload arch and absence of ratings from Profile.
- Browser inspection of a local file was blocked by the browser URL policy; rendered layout was not verified in a browser. The inline conversation preview is the review surface.
- No app code or existing dev services changed.


## Implementation record — 2 October 2026

The approved grouped Profile grid, rightmost Listings tab, circular upload arch,
category-driven photo composer, listing reel, filters, saved/My listings, public
comments, enquiries and conversation-based seller ratings are implemented in
source. Current Profile actions remain in their grouped submenus. Messages has
All / Listings / Unread filters; the Profile tile and navigation badge retain
unread visibility. Listings use 30 × 24 hours from approval and cost 10 Stars
across all categories. No seller score is added to either profile surface.

The backend provides phone/email verification, private S3 uploads and EXIF-free
variants, manual moderation, atomic idempotent publication charging, report/hold
controls, expiry/purge, related-listing preparation and publication notifications.
The website admin console has listing/review/comment moderation; policy pages
and in-app Help explain lead-only operation, fees, prohibited categories, safety,
ratings and retention. There is no deal-payment, commission or tax-filing flow.

Enquire opens an editable `is this still available` draft without sending it.
A buyer can rate after 24 hours following a two-way exchange; the listing chat
shows a prompt and menu action. One current rating per reviewer/seller pair counts.
The listing card and its review panel show the score. A verified contact or a
rating is never labelled a quality guarantee or verified purchase.

Verification evidence: backend TypeScript check and ten isolated PostgreSQL
lifecycle tests pass; website JavaScript parses. Initial Flutter analysis found
one picker-test override mismatch, now corrected, plus style diagnostics.
A follow-up focused analyzer was capped at 512 MB and low CPU priority; it ran
out of its capped heap. It was not restarted with more memory. Final mobile
analysis, updated widget tests, simulator/physical UI acceptance and ad-pager
regressions remain pending. No simulator build or container rebuild was started.
Existing dev services were kept running.

Activation order and operations are recorded in `iadme-backend/README.md`,
section Local Listings. Provision the separate private bucket, apply the migration
before API/worker rollout, then release the website/mobile updates. Confirm
public policy wording and the existing jurisdiction/store-billing release items
from the planning record before public launch. This source change does not
constitute production activation or completion of those release checks.

## Dev activation — 2 October 2026

Follow-up owner direction: test in dev first. Do not prepare APK, IPA or app
bundles yet. The extra Android packaging attempt was cancelled; keep the local
API, worker, admin preview and iOS debug simulator available for dev acceptance.

The owner subsequently confirmed the laptop was free and explicitly authorized all work in dev only. Dev activation supersedes the earlier deferred-check status above. The separate private S3 bucket and local migration are installed, API/worker rebuilt and healthy, and the admin/policy pages served locally. Full mobile analysis has no issues; 97 focused tests and native iOS profile, listing, enquiry, category and upload-arch checks passed. Real dev uploads, worker conversion, moderation, idempotent charging, 30-day expiry, ratings, notifications and media deletion passed with synthetic accounts. Test media was purged and synthetic accounts disabled. See [the complete evidence and limits](../test-results/LISTINGS_DEV_2026-10-02.md). No staging/prod or store rollout.


## Dev follow-up — owner corrections, October 2

These decisions supersede the earlier manual-approval flow and the proposed photo-first flow:
- Keep category/details first, including job vacancies. Choose photos within the form.
- Publish automatically after photo processing; debit 10 Stars atomically once. Admins review/remove live listings afterward. Review never charges a second fee or resets expiry.
- Check both verified contacts immediately on opening Listing. Put verification under Profile → Account → Verify email or phone, with a direct link from the gate. Remove verification rows from the listing form.
- Show “Publish · 10 Stars” and a subtle info note “Listing will be active for 30 days”. Remove the large fee/duration/photo-count heading and all submit-for-review language.
- Reuse reel/photo location data with Sub-locality, City, State choices. Never silently publish a fallback GPS area; allow typed area when location cannot resolve. Store/display only the chosen area label.
- Goods condition is exactly New or Used, enforced by client and API; jobs/services retain their relevant questions.
- Dev simulator and local API/worker only; no staging/production rollout or distribution packages.


## Latest dev follow-up — Feed parity and compact listing text

The October 2, 16:43 screenshots and subsequent owner corrections supersede the
older instruction to replace ScrollScore. Listings remain photos-only (up to
five); video upload was explicitly deferred by the owner.

- Profile order is Inbox / Messages / Wallet, Account / History / Settings,
  Help / i³ AI / Privacy. Existing submenus remain available.
- Top heading is Listings. The shared Feed header, photo carousel, creator and
  location row, compact caption and seven-action rail are used by Listings.
  The rail contains views, like, dislike, target, comments, share and report.
- ScrollScore uses the existing Feed engagement weights. It is separate from
  Stars and creates no watch/scroll rewards. Seller interaction rating remains
  a compact extra on the listing, never on Profile.
- Listing title is a one-line Feed caption. See more is always available and
  opens a draggable sheet containing the full title, description, price, area,
  category-specific details and platform-role notice. Small price, rating and
  Enquire controls replace the oversized listing text.
- The location pill uses the Feed distance formatter when approximate location
  data is available, falling back to the chosen area. Sub-locality listings may
  include coordinates rounded to two decimal places. City/State and typed/fallback
  locations remain text-only. API responses never expose seller coordinates.
- Shared photo rendering keeps private listing images in bounded memory only;
  public Feed photo disk caching and the next-two video preparation path retain
  their existing behavior. Photo indicators replace video-only playback controls.
- Sharing opens the system share sheet with an installed-app `iadme://listing/…`
  link. A successful native share records one share per account/listing. An HTTPS
  public listing landing page is not part of this dev change.
- The filter/report controller lifecycle fix keeps controllers alive through the
  closing sheet/dialog animation. A regression test reproduces the former disposed
  controller crash when applying a focused search field.

Runtime and test evidence is tracked in
[the dev acceptance record](../test-results/LISTINGS_DEV_2026-10-02.md).

## Job form and closure feedback — October 2, 17:27 follow-up

Owner requested clearer pay and working-hours units, selectable employment types,
removal of the application closing date and an app-store feedback prompt after
closing a listing. These decisions apply to dev first.

- Jobs use one Pay section: Fixed amount, Range, or To be discussed. Fixed and
  range pay require a period: Hour, Day, Week, Fortnight, Month, Quarter, Year or
  One-off. A range has minimum and maximum amounts. Discussed pay hides amounts
  and period and displays “Pay to be discussed”, never a misleading ₹0 salary.
- Employment type choices are Permanent, Temporary, Contract, Casual, Freelance,
  Internship and Apprenticeship. A separate Work schedule chooses Full-time,
  Part-time or Flexible, since a permanent job can still be part-time.
- Working hours choose Fixed hours or Flexible / to be agreed. Fixed hours have
  a numeric total and Day/Week dropdown. Do not offer month/quarter/year hours:
  these obscure the recurring work schedule. Validate the possible totals
  (at most 24 per day or 168 per week), and require positive pay and valid ranges.
- Remove the duplicated free-text salary field and application closing date from
  the composer and details sheet. Existing immutable upload requests still retry
  without alteration; old salary wording is displayed without guessing a period.
- Reel and share text use the same compact pay label, such as ₹200 / day or
  ₹200–₹300 / day. Full job terms remain in See more. Photos-only and category-first
  upload behavior, the 10-Star fee and 30-day duration continue.
- On the first successful advertiser closure of a published listing, request the
  native App Store / Play Store rating sheet. Do not ask for a positive outcome,
  incentive, rating prediction or compulsory review. Draft disposal, failed
  requests and repeat closure requests do not trigger it. Closure succeeds even
  when the review API fails or cannot display a prompt.
- Native requests have a persisted 120-day device cooldown. The existing manual
  Profile → Help → Rate iAdMe action remains available. Automatic monthly feedback
  respects a 30-day gap from a recent prompt; showing an existing prompt also
  defers the new closure request to avoid duplicate nagging.
- Store quotas and user settings control whether the native sheet appears. Dev
  checks cannot establish actual review submission. No review is submitted by
  implementation/testing. Android distribution acceptance remains a later store
  test, not a reason to build a distribution bundle now.

References: [Apple ratings guidance](https://developer.apple.com/app-store/ratings-and-reviews/),
[Google in-app review guidance](https://developer.android.com/guide/playcore/in-app-review).

## October 2 follow-up: direct navigation, Targets and publication receipts

- Profile Inbox and Messages open their screens directly. Messages retains its
  existing filters. Wallet retains all three submenu entries.
- Remove My listings and Saved listings from the History sheet. Owner management
  remains under Listings → My listings and History → Uploads → My listings.
- Targets includes a compact Listings section for either a target reaction or a
  saved bookmark, deduplicated per listing, with pagination and refresh on return.
  Feed reel/photo goal-progress behavior remains unchanged. Expired/removed or
  blocked listings follow normal listing visibility rules.
- Every newly published listing, video and photo post receives one owner-only
  official Inbox receipt and an email delivery job for the account email address.
  Subject: Your listing / video / photo post is published. Drafts, pending uploads
  and failures do not receive publication receipts. No historical backfill.
- Enqueue the receipt and email job in the publication transaction. Deterministic
  content IDs prevent repeated processing callbacks creating extra Inbox items.
  Reuse the existing official-email worker for escaped HTML, bounded retry and
  delivery status; keep the existing publication bell/push without a second push.
  An account without email still receives Inbox and skips external email.
- Apply 20261002_publication_inbox.sql before this backend code. It permits
  automatic transactional publication campaigns without misattributing an admin.
- Provider transport still has normal at-least-once retry limitations if a worker
  stops after the provider accepts a send but before recording success. Do not
  promise exactly-once external mailbox delivery.

Dev video acceptance dependency: existing local MediaConvert values point at
staging, so local submission/event polling are now disabled. Configure dedicated
dev processing resources before end-to-end video-upload acceptance; video receipt
logic is already integration tested. Do not re-enable the staging-linked values
for a dev-only test. Listing/photo publication and email receipt paths are active.
