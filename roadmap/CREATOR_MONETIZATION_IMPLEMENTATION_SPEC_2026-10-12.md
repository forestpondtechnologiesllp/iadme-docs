# iAdMe Stars, Creator Monetization and Creator Plus — Implementation Reference

**Document version:** 1.0  
**Prepared:** 12 October 2026, Asia/Kolkata  
**Related backlog:** IADME-043; premium unlock reporting dependency in IADME-032  
**Status:** Detailed design reference. Application implementation, store configuration and public policy publication are separate work. This document does not establish that any proposed feature is deployed.  
**Audience:** Product, backend, mobile, website, QA, finance and support.

## Contents

1. [Purpose and how to use this document](#1-purpose-and-how-to-use-this-document)
2. [Decision register](#2-decision-register)
3. [Current evidence and code starting points](#3-current-evidence-and-code-starting-points)
4. [Product terminology and balances](#4-product-terminology-and-balances)
5. [Revised Star packs and welcome grant](#5-revised-star-packs-and-welcome-grant)
6. [Premium unlock behavior](#6-premium-unlock-behavior)
7. [Earned creator route](#7-earned-creator-route-15-public-videos-to-five-premium-videos)
8. [Creator Plus subscription](#8-creator-plus-subscription)
9. [Cancellation, expiry and viewing rights](#9-cancellation-expiry-and-viewing-rights)
10. [Revenue attribution and payout formulas](#10-revenue-attribution-and-creator-payout-formulas)
11. [Worked financial examples](#11-worked-financial-examples)
12. [Subscription economics and company break-even](#12-subscription-economics-and-company-break-even)
13. [Suggested backend model](#13-suggested-backend-model)
14. [Transaction workflows](#14-transaction-workflows)
15. [Refunds, reversals, fraud and corrections](#15-refunds-reversals-fraud-and-corrections)
16. [API and configuration design](#16-api-and-configuration-design)
17. [Mobile, web and creator experience](#17-mobile-web-and-creator-experience)
18. [Policy and FAQ content drafts](#18-policy-and-faq-content-drafts)
19. [Migration and compatibility](#19-migration-and-compatibility)
20. [Delivery phases and implementation checklist](#20-delivery-phases-and-implementation-checklist)
21. [Acceptance-test matrix](#21-acceptance-test-matrix)
22. [Observability, reconciliation and operational review](#22-observability-reconciliation-and-operational-review)
23. [Open decisions](#23-open-decisions-to-resolve-for-the-affected-feature)
24. [Scope boundaries](#24-scope-boundaries)
25. [Source notes and verification scope](#25-source-notes-and-verification-scope)
26. [Compact handoff checklist](#26-compact-handoff-checklist)

## 1. Purpose and how to use this document

Build a Stars economy in which viewers can try premium videos, creators can earn from purchases, and iAdMe retains a measurable contribution toward operating expenses.

This reference captures the owner's requested changes and the latest design discussion. It includes commercial rules, formulas, examples, suggested architecture, policy drafts, migration requirements and acceptance tests. It deliberately separates decisions the owner explicitly accepted from follow-up recommendations that still need a commercial decision.

Do not interpret a recommended technical design as evidence that the existing code implements it. Do not treat illustrative costs, refund rates or tax assumptions as measured production facts. Do not publish draft policy language without reconciling it with the final product behavior, applicable requirements and provider agreements.

### 1.1 Decision labels

| Label | Meaning |
| --- | --- |
| **OWNER-CONFIRMED** | Explicitly accepted or specified by the owner in the discussion. |
| **OWNER-REQUESTED** | Requested direction, such as increasing welcome Stars to 50; adopted for this working design. |
| **WORKING BASELINE** | Recommended in the follow-up response, but not separately confirmed by the owner. Build configurability where useful; settle material commercial choices before enabling the affected feature. |
| **ENGINEERING RECOMMENDATION** | A proposed way to implement the behavior safely and consistently; exact schemas and endpoint names may change. |
| **OPEN** | A commercial, finance, operational or entitlement detail that must be resolved for the relevant production feature. |
| **ILLUSTRATION** | A calculation using stated assumptions, not a payout promise or measured bill. |

Open commercial points should not prevent independent schema, accounting, UI or test work. Resolve routine implementation choices through repository conventions and engineering judgment. A production rollout remains separate from creating this document.

### 1.2 What supersedes earlier discussion

- Use **50**, not 20, welcome Stars in the working design.
- Allow welcome Stars to unlock premium videos. The earlier upload-only recommendation is superseded.
- Keep the existing **four-minute** video limit. The earlier 60-second proposal is superseded.
- Completing 15 qualifying public uploads permits **five**, not one, of those videos to become premium.
- Keep the ₹99 Creator Plus proposal with ten premium publications per billing month.
- Do not reuse the earlier estimated ₹32–₹44 subscription contribution as an established margin. It depended on an unmeasured ₹25 servicing allowance and a shorter-video proposal.
- Stars do not have a fixed 1:1 exchange or withdrawal relationship with INR.

## 2. Decision register

| ID | Rule or direction | Status | Implementation consequence |
| --- | --- | --- | --- |
| P01 | Explicitly state that ₹1 is not necessarily 1 Star and 1 Star is not redeemable for ₹1 | OWNER-CONFIRMED | Update relevant Terms, FAQs, payment/Stars and creator policies, wallet and purchase explanations. |
| P02 | New pack totals: 100 / 210 / 550 / 1,150 at ₹99 / ₹199 / ₹499 / ₹999 | OWNER-CONFIRMED | Version the catalogue and preserve historical purchase quantities. |
| P03 | Increase the proposed 20-Star welcome grant to 50 | OWNER-REQUESTED | New registrations receive 50 once; existing balances are preserved. |
| P04 | Free Stars must be usable for premium viewing | OWNER-REQUESTED | Same viewing access whether an unlock is paid-funded or promotional. |
| P05 | Standard premium unlock costs five Stars | Working design retained from owner's scenario | Verify live configuration and explicitly roll out five; source defaults currently differ. |
| P06 | Ordinary video upload costs ten Stars | Working design retained from owner's scenario | Charge once for a qualifying publication; define failure reversal behavior. |
| P07 | Earned route requires 15 qualifying public videos and permits five of those videos to become premium | OWNER-CONFIRMED | Count qualifying videos and record earned premium grants independently of subscription status. |
| P08 | Social sharing is encouraged, not mandatory | OWNER-CONFIRMED | No sharing gate in qualification or subscription benefits. |
| P09 | ₹99/month immediately enables premium publishing | OWNER-CONFIRMED | Provider-verified membership grants eligibility without the 15-video prerequisite; content rules still apply. |
| P10 | Ten premium publications per billing month, without an additional ten-Star upload charge | OWNER-CONFIRMED | Subscription quota substitutes for the charge on included premium uploads. |
| P11 | No quota rollover; conversion of an existing video consumes one slot | OWNER-CONFIRMED | Count successful publications and conversions consistently. |
| P12 | Keep four minutes per video | OWNER-CONFIRMED | Validate up to 240 seconds on the server and client. |
| P13 | Creator receives 70% of qualifying net unlock revenue; company retains 30% | OWNER-CONFIRMED | Apply the split to attributable net transaction revenue, not a nominal rupee-per-Star rate. |
| P14 | Cancellation ends future membership benefits at the paid period's end | OWNER-CONFIRMED | Turning off renewal is not immediate entitlement expiry. |
| B01 | Promotional-only unlocks generate no creator cash earnings unless separately sponsored | WORKING BASELINE | Zero purchase revenue is allocated; show promotional unlocks separately to creators. |
| B02 | The five-video earned benefit is a one-time milestone benefit, not five more videos each month | WORKING BASELINE | Do not silently multiply earned grants by time or every additional 15 uploads. |
| B03 | Subscription provides publishing services, no spendable Stars or free viewing allowance | WORKING BASELINE | Viewing still consumes five Stars from a valid viewer balance. |
| B04 | On expiry, stop new sales of subscription-only premium videos while preserving existing buyers' access | WORKING BASELINE | Separate sale eligibility from previously acquired viewing access. |
| B05 | Earned premium grants survive subscription cancellation | WORKING BASELINE | A creator's earned five do not depend on renewal. |
| B06 | ₹500 minimum payout, monthly settlement and a 30-day pending period | PROPOSAL, OPEN | Do not hard-code or publish these as final policy yet. |
| B07 | Promotional Stars spent first, then purchased lots in acquisition order | ENGINEERING/PRODUCT RECOMMENDATION, OPEN | Clearly disclose the chosen deterministic spending order. |

## 3. Current evidence and code starting points

The following was observed in this workspace and discussion on 12 October 2026. These are starting points for implementation, not a complete audit.

### 3.1 Observed behavior and gaps

1. The public production catalogue at `https://api.iadme.app/payments/star-packs` returned totals of **100, 220, 600 and 1,300** for ₹99, ₹199, ₹499 and ₹999 during this discussion. The revised quantities are not assumed deployed.
2. The initial pack migration contains those same quantities.
3. The registration configuration and a production migration set the welcome grant to **100**. Both ordinary and social registration read that configuration.
4. The mobile upload screen declares `_maxVideoDurationSeconds = 240`.
5. `uploads.costStars` is used for video upload pricing; the inspected fallback is ten Stars.
6. The checked-in `premium.defaultUnlockStars` default and the inspected historical migration contain **2**, despite the owner's intended five-Star scenario. Runtime configuration can override defaults. The live authenticated value was not verified here.
7. The inspected unlock service debits the viewer wallet and creates an unlock. It does not credit a creator INR earnings ledger in that flow.
8. The inspected unlock flow performs separate repository writes. The new financial path must establish atomicity and concurrency safety rather than assume these already exist.
9. Payment orders already snapshot pack quantities and price. The inspected creation paths use a catalogue version value of `1`; extend version handling rather than assuming a working catalogue-version lifecycle exists.
10. Existing automatic premium promotion is a separate path. It includes prior-view access behavior. Its interaction with creator eligibility and earnings must be resolved explicitly.
11. A `subscription.discountPercent` configuration default of 50 exists in source. This is not evidence of a functioning subscription or an agreed viewing discount. The proposed Creator Plus plan does not include that discount.
12. Existing media processing produces multiple HLS renditions. Cost estimation must account for the actual pipeline rather than one source file alone.

### 3.2 Repository map

Paths below are relative to the iAdMe workspace root.

| Area | Starting points |
| --- | --- |
| Pack catalogue migration | `iadme-backend/database/migrations/20260707_create_star_purchase_packs.sql` |
| Registration configuration | `iadme-backend/database/migrations/20260819_set_registration_bonus_stars_100.sql`; `iadme-backend/services/api/src/main/modules/platform-config/platform-config.defaults.ts` |
| Normal/social registration | `iadme-backend/services/api/src/main/modules/auth/auth.service.ts`; `auth/social-auth.service.ts` |
| Payment verification, snapshots, refunds | `iadme-backend/services/api/src/main/modules/payments/payments.service.ts`; `payments.repository.ts`; `payments.types.ts`; `payments.routes.ts` |
| Wallet ledger | `iadme-backend/services/api/src/main/modules/wallet/wallet.repository.ts`; `wallet.service.ts` |
| Unlock flow | `iadme-backend/services/api/src/main/modules/videos/video-unlocks.service.ts`; `video-unlocks.repository.ts` |
| Upload fees/publication | `iadme-backend/services/api/src/main/modules/uploads/upload-config.service.ts`; `videos/videos.service.ts` |
| Premium settings | `iadme-backend/services/api/src/main/modules/platform-config/premium-config.service.ts`; `platform-config.defaults.ts` |
| Automatic premium promotion | `iadme-backend/services/api/src/main/modules/premium-promotions/` |
| Video processing | `iadme-backend/services/api/src/main/modules/videos/video-processing-job-settings.ts`; `video-processing.service.ts` |
| Mobile pack model | `iadme-mobile/apps/iadme_app/lib/features/payments/models/star_pack.dart` |
| Mobile store billing | `iadme-mobile/apps/iadme_app/lib/features/payments/data/apple_in_app_purchase_service.dart`; `google_play_billing_service.dart` |
| Mobile purchase UI/API | `iadme-mobile/apps/iadme_app/lib/features/payments/presentation/payments_screen.dart`; `data/payments_api.dart` |
| Mobile upload UI | `iadme-mobile/apps/iadme_app/lib/features/upload/presentation/upload_screen.dart` |
| Website policies | `iadme-website/terms-and-conditions.html`; `refund-and-cancellation-policy.html`; `privacy-policy.html` |
| Billing notes | `docs/compliance/apple-app-store-billing-stars.md`; `google-play-billing-stars.md` |
| Build/release instructions | `IADME_BUILD_DEPLOY_COMMANDS.md`; `iadme-mobile/AGENTS.md` |
| Background playback constraints | `iadme-mobile/docs/background-reels-2026-09-18.md` |

Before modifying app behavior, reread current files and applicable repository instructions. Some historical billing notes describe older provider routing; use the current checked launcher/build documentation and implementation to resolve discrepancies.

## 4. Product terminology and balances

### 4.1 User-facing concepts

- **Stars:** In-app credits spent on eligible actions. No fixed 1:1 relationship with INR.
- **Welcome/promotional Stars:** Credits granted without a customer purchase. They can unlock premium videos under this design.
- **Purchased Stars:** Credits from a verified purchase, including the bonus Stars sold as part of that pack.
- **Premium video:** A video requiring an unlock for a viewer who does not already have access.
- **Earned creator:** A creator who qualifies through the 15-public-video milestone.
- **Creator Plus:** The proposed ₹99/month premium-publishing subscription.
- **Creator earnings:** A separate INR-denominated balance attributable to valid revenue-generating activity.
- **Viewing access:** The viewer's entitlement after an unlock, independent of the creator's current membership status.

Use “Creator Plus” instead of using “premium user” for both uploaders and viewers. Purchasing one premium video does not grant creator publishing privileges. Buying Creator Plus does not imply unlimited premium viewing.

### 4.2 Internal distinctions

The wallet may show one total Star balance, with a clear purchased/promotional breakdown. Internally, every spend needs provenance.

Three systems must remain separate:

1. The **Star ledger**: integer credit/debit quantities and their sources.
2. The **publication entitlement system**: milestone grants, membership periods and monthly quota.
3. The **creator INR earnings ledger**: pending, available, held, paid and adjusted earnings.

Do not implement cash withdrawals by converting an arbitrary Star wallet balance. A user's purchased Stars are not their creator earnings. An account may be both viewer and creator without merging those balances.

## 5. Revised Star packs and welcome grant

### 5.1 New purchase catalogue

| Pack code | Customer price | Base Stars | Pack bonus | New total | Previous total |
| --- | ---: | ---: | ---: | ---: | ---: |
| STARTER | ₹99 | 100 | 0 | **100** | 100 |
| POPULAR | ₹199 | 200 | 10 | **210** | 220 |
| PRO | ₹499 | 500 | 50 | **550** | 600 |
| MEGA | ₹999 | 1,000 | 150 | **1,150** | 1,300 |

These are the proposed India customer prices. The verified store price and transaction currency remain authoritative for each purchase. Do not force an advertised INR amount into receipts from another currency or storefront. International expansion is separate scope.

All Stars in one purchased pack share that pack's attributable funding. The 150 bonus Stars in the new MEGA pack are **not** zero-funded registration promotions. Allocate the pack's net revenue across all 1,150 Stars.

### 5.2 Welcome grant

- Working quantity: **50 Stars once per newly registered eligible user**.
- Do not award the grant again on login, account linking, reinstall, device change or retry.
- Apply the same behavior to normal registration and each supported social registration flow.
- Allow these Stars to pay for premium unlocks and ordinary video uploads.
- Do not make the new rule retroactively subtract Stars from existing accounts.
- Existing accounts do not automatically receive an additional 50-Star grant.
- Any expiry policy for newly granted promotional Stars is a separate decision; do not add an expiry silently.
- Purchased Stars must not be expired to simplify accounting or migration.

### 5.3 Welcome-Star examples

| Use of the 50-Star grant | Result |
| --- | --- |
| All on viewing | Ten five-Star premium unlocks |
| All on ordinary uploads | Five ten-Star uploads |
| Three uploads, then viewing | Three uploads plus four premium unlocks |
| Four unlocks, then uploads | Four unlocks plus three uploads |

For the owner's audience example, assume 90 of 100 users use all welcome Stars for viewing and ten use all of theirs for uploads. The resulting workload is **900 promotional unlocks plus 50 uploads**. Purchase revenue from those grants is zero. Actual delivery and processing expenses must come from the acquisition budget.

Do not call the grant “₹50 free cash” or “₹50 withdrawable balance.” It is 50 Stars.

## 6. Premium unlock behavior

### 6.1 Standard transaction

- Working price: **five Stars per distinct viewer/video unlock**.
- Accept promotional, purchased or mixed funding.
- Provide the same content access regardless of funding source.
- Once validly unlocked, repeat viewing must not debit another five Stars or generate another creator earnings event.
- Previews and ordinary views do not themselves constitute paid unlocks.
- An owner can view their own eligible content without purchasing it; they cannot manufacture a self-unlock earning event.
- A viewer does not need Creator Plus to unlock a premium video.
- The server validates price, wallet, current offer eligibility and prior access; the client cannot declare an unlock paid.

### 6.2 Spending order

**Recommendation, not an owner-confirmed policy:** spend promotional Stars first, then purchased lots in acquisition order, using stable tie-breaking identifiers. This lets users consume the welcome benefit before purchased credits.

Whichever order is chosen, it must be deterministic, disclosed in the wallet/FAQ, audited and applied to all relevant spend paths. Do not silently choose the funding lot that minimizes creator earnings. Changes to spending order require a versioned policy and a migration assessment.

### 6.3 Funding classes and reporting

| Unlock funding | Viewing access | Purchase-derived creator earnings | Purchase-derived company revenue |
| --- | --- | --- | --- |
| Five promotional Stars | Full unlocked access | Zero | Zero |
| Five purchased Stars | Full unlocked access | 70% of attributable net revenue | 30% of attributable net revenue |
| Purchased plus promotional Stars totaling five | Full unlocked access | 70% of the purchased portion's attributable net revenue | 30% of that same portion |

Use mutually exclusive categories for reporting:

```text
total unlocks = fully promotional unlocks + mixed unlocks + fully purchased unlocks
revenue-generating unlocks = mixed unlocks + fully purchased unlocks
```

Never multiply all “revenue-generating unlocks” by the full five-purchased-Star payout: mixed unlocks can earn less. Sum actual allocations.

Show creator dashboards both access activity and earnings, for example: “100 total unlocks: 60 promotional, 10 mixed, 30 purchased.” Earnings must be derived from the underlying purchased-Star allocations, not the displayed count alone.

### 6.4 Promotional payouts

The working baseline is **no automatic cash payout for a promotional-only unlock**. This still gives viewers access and creators audience exposure, but it must be disclosed before creator enrolment.

If iAdMe later sponsors promotional earnings, use a separate campaign with a fixed budget, eligibility rules, funded liability, start/end conditions and an audit trail. Record these payments as company-funded incentives, not as revenue from free Stars. Do not create an uncapped fixed-rupee payout for free grants.

## 7. Earned creator route: 15 public videos to five premium videos

### 7.1 Core rule

The owner wants this route to grow the public video library. After **15 qualifying public uploads**, a creator may select **five of those videos** to become premium.

Working baseline:

- The milestone confers a one-time five-video benefit.
- The five earned grants do not require a monthly subscription.
- Selecting one of the qualifying videos does not trigger a second ordinary upload fee: its original publication was already charged.
- Premium-only uploads do not count as public uploads just because their owner is subscribed.
- Public videos must be distinct, successfully published and eligible under content rules.
- Social sharing is optional and is not a qualification counter.
- Fifteen qualifying videos with five promoted leaves ten of that group public at the time of selection.

Do not silently grant five more premium videos every month or for each additional 15 uploads. Repeated milestones and replacement of the five selections are open commercial choices.

### 7.2 Qualification evidence

Record the video IDs that established the milestone and why they qualified. Use server publication/moderation evidence rather than a client upload count.

Drafts, failed processing jobs, duplicate publication retries, disallowed content and photos do not count toward the 15-video milestone. A policy is still needed for substantive reuploads, videos subsequently deleted, and videos that were public only briefly.

**Recommended implementation:** persist the awarded milestone rather than recomputing it from current public visibility. Otherwise, promoting the first five to premium would reduce the public count to ten and incorrectly revoke the very benefit just earned.

Milestone retention after later deletion or a policy violation must follow an explicit rule. Do not invent irreversible revocation behavior while implementing the counter.

### 7.3 Qualification cost example

```text
15 uploads × 10 Stars                 = 150 Stars
Welcome grant reserved for uploads    =  50 Stars
Remaining Stars required              = 100 Stars
One STARTER pack for ₹99              = 100 Stars
```

This is a **conditional minimum**, not a guarantee that every user qualifies for ₹99. If welcome Stars were already spent on viewing or other actions, the user needs additional Stars. Moderation and successful publication requirements still apply.

The earlier assertion that a ₹199 pack would be necessary does not apply when all 50 welcome Stars remain available. Do not advertise the milestone as a completely free route: ordinary uploads consume Stars.

## 8. Creator Plus subscription

### 8.1 Included service

| Property | Working specification |
| --- | --- |
| Name | Creator Plus |
| India target price | ₹99 per month, subject to the actual store product price and displayed tax treatment |
| Eligibility start | After verified successful subscription purchase |
| Public-upload prerequisite | None for subscription premium publishing |
| Allowance | Ten premium publications in each provider-defined paid billing period |
| Maximum video duration | Four minutes each |
| Upload charge within allowance | No additional ten-Star charge |
| Existing public video converted to premium | Consumes one publication slot |
| Unused quota | Does not roll over |
| Viewing benefit | No included viewing-Star grant or unlimited premium viewing in the working baseline |
| Creator share | Same 70% of qualifying net unlock revenue |
| Content standards | Unchanged; payment does not bypass moderation or guarantee publication |

Ten publications per period are not ten publications for the lifetime of the membership. Older eligible premium publications can remain saleable while the membership is active under the working baseline. This means catalogue storage can accumulate; ten new uploads is not a cap on all stored media.

The plan does not currently specify extra paid premium slots after ten, a higher tier, annual billing or prorated upgrades. Do not silently charge Stars for an eleventh subscription-only premium publication. Show the limit and next verified renewal date until an overage product is designed.

### 8.2 Distinguishing the two routes

| Scenario | Earned route | Creator Plus |
| --- | --- | --- |
| Initial work | Publish 15 qualifying public videos | Purchase a verified subscription |
| Initial cash example | ₹99 for 100 Stars if all 50 welcome Stars were saved | ₹99 for the first paid period |
| Premium allowance | Five selected qualifying videos | Ten premium publications each billing period |
| Repeat payment | None for the earned five | Required for continued subscription-only benefits |
| Purpose | Reward contribution to the public library | Sell immediate and recurring premium-publishing capacity |

The initial ₹99 amount overlaps, but the ongoing benefits differ. Do not hide that subscribers can bypass the public-library milestone. That is an intentional tradeoff of offering immediate premium eligibility.

### 8.3 Billing periods and quota

- Use provider-verified period start/end timestamps. A month is not always 30 days.
- Store timestamps in UTC; show clear local dates to the user.
- Reset quota only for a distinct verified renewal period, not on app open or the calendar's first day.
- Restoring purchases does not grant another ten slots for the same period.
- Duplicate or out-of-order notifications do not create extra quota.
- Maintain `reserved`, `consumed` and `released` publication slots so concurrent uploads cannot exceed ten.
- Bind a reservation to a specific user, video intent, period and idempotency key.
- Release a reservation for a genuinely failed technical publication according to the finalized failure policy. Retry should reuse the original intent.
- Converting an already-premium video again is not a new publication.
- Do not automatically refund a consumed slot merely because the creator deletes the published video; that would permit unlimited processing through upload/delete loops.

**Open timing choice:** an upload started before period expiry may finish afterward. Recommended behavior is to reserve against the verified period at acceptance and permit completion within a bounded reservation window without moving the charge to a later period. Define the window and refund/failure semantics before enabling it.

### 8.4 Coexisting earned and subscribed rights

A creator can hold both earned rights and a subscription. Store each video's qualifying grant explicitly.

- The five earned video grants remain independent of subscription renewal.
- Subscription-only publications depend on membership for new sales under the baseline.
- Do not accidentally consume both an earned grant and a subscription slot for one conversion.
- An already earned-premium video should not need another slot merely to remain premium after subscribing.
- If the user wants a new qualifying video to use an earned grant instead of a subscription slot, that choice must comply with the final earned-selection/replacement policy.
- A public video uploaded by a subscriber is not automatically an included premium upload. Public-upload charging and qualification follow their own rule until an expanded benefit is explicitly designed.

## 9. Cancellation, expiry and viewing rights

### 9.1 Separate states

Do not represent everything with a single `isPremium` flag. At minimum distinguish:

1. Whether the creator has any earned premium grants.
2. Whether a subscription is active for the current period.
3. Whether auto-renewal is enabled.
4. Whether a particular video can accept new unlocks.
5. Whether a particular viewer already has access.
6. Whether a content or account restriction independently blocks access.

### 9.2 Working transition rules

| Event | New subscription premium publications | New sales of subscription-only videos | Existing buyers | Earned five | Accrued earnings |
| --- | --- | --- | --- | --- | --- |
| Active paid membership | Allowed within quota | Allowed | Keep access | Remain eligible | Continue under normal rules |
| Auto-renew switched off before period end | Allowed within remaining quota | Allowed until paid end | Keep access | Remain eligible | Preserved |
| Paid period expires | Not allowed through subscription | Stop under baseline | Keep access | Remain eligible | Preserved and settleable |
| Valid resubscription | New verified-period quota | Resume eligible offers under finalized rules | No duplicate purchase required | Remain eligible | Preserved |
| Subscription refunded/revoked | Provider status controls future membership | Evaluate revocation policy | Do not blindly erase unrelated viewer purchases | Independently assessed | Do not confiscate unrelated earnings |

Billing grace periods, account hold, refunds and revocations are not equivalent to ordinary cancellation. Follow verified provider states and a documented entitlement policy. Resolve the grace-period rules before production enablement.

### 9.3 Durable viewing entitlement

An unlocked viewer should not lose access merely because the creator cancels Creator Plus. New-sales eligibility and existing access must be evaluated separately. Do not turn expired premium content public automatically; it may remain visible as a preview with purchases unavailable while prior buyers retain access.

Content removal, legal restrictions, blocked accounts, viewer refunds and deletion require their own documented rules. This reference does not promise that prohibited or deleted content remains available forever. Those exceptions must not become a mechanism to revoke valid purchases just because a creator stopped paying.

## 10. Revenue attribution and creator payout formulas

### 10.1 Production net revenue definition

For a purchased Star lot, start from the transaction's actual customer consideration and reconcile:

```text
Actual attributable net receipts
  = Customer payment
  − applicable indirect tax that is not iAdMe revenue
  − actual store/payment processing fees
  − applicable nonrecoverable transaction/fee taxes
  − actual refunds and chargebacks
  + actual reversals or recoveries of earlier deductions
```

Map fields from real provider statements. Do not subtract tax or fees again from a report field already stated net of those items. Do not treat recoverable withholding, tax credits or settlement timing differences as permanent economic losses without finance review.

Until final settlement is available, any estimate must be marked provisional and reconcilable. The store fee is channel- and transaction-dependent. A fixed 15% fee is not a universal production constant.

### 10.2 Lot allocation

For a simple settled purchase lot:

```text
S = total Stars supplied in that purchased pack, INCLUDING pack bonus
N = attributable net receipts for that lot
v = N / S                         # accounting allocation, not a retail exchange rate

For unlock j:
A_j = sum(purchased Stars consumed from lot i × v_i)
Creator earning_j = A_j × 70%
Company share_j   = A_j × 30%
```

Promotional Stars contribute zero to `A_j` unless a separate sponsored program explicitly provides funding. Actual refunds and reversals after consumption require allocation adjustments; do not simply change a mutable global Star value and lose the history.

### 10.3 Forecasting formula used in this discussion

The following is an **illustration**, not production tax logic:

```text
P = tax-inclusive customer pack price
t = assumed included tax rate
f = assumed store commission rate on tax-exclusive price
r = forecast loss allowance applied after that fee
S = total pack Stars including bonus
q = purchased Stars used for this unlock

Estimated net pack revenue = P / (1 + t) × (1 − f) × (1 − r)
Estimated unlock revenue   = Estimated net pack revenue / S × q
Estimated creator earning  = Estimated unlock revenue × 0.70
Estimated company share    = Estimated unlock revenue × 0.30
```

Illustrative inputs in the worked examples: `t = 18%`, `f = 15%`, `r = 3%`, `q = 5` for a fully purchased-Star unlock. Verify actual India classification, tax treatment, fee bases, fee taxes and provider settlement reporting before production use.

### 10.4 Refund allowance versus payout reserve

These are different concepts:

- **Forecast refund allowance:** an assumed expected loss used in planning. The 3% factor below belongs here.
- **Actual refund:** a provider-confirmed financial event that adjusts the relevant purchase and allocations.
- **Payout reserve:** money temporarily withheld from otherwise recognized earnings to cover risk. It needs an explicit duration, release process and adjustment rule.

Do not permanently keep 3% merely because it appeared in this forecast. Do not deduct a forecast loss and the same actual refund a second time. A reserve is not additional company revenue; show it separately and release or apply it transparently.

## 11. Worked financial examples

### 11.1 Why one five-Star unlock can earn ₹2.42

A viewer purchases 100 Stars for ₹99. All figures below use the illustrative 18% included tax, 15% store fee and 3% forecast loss assumptions.

| Step | Formula | Result before display rounding |
| --- | --- | ---: |
| Tax-exclusive customer price | `99 / 1.18` | ₹83.89830508 |
| Store commission | `83.89830508 × 0.15` | ₹12.58474576 |
| Receipts after store commission | `83.89830508 × 0.85` | ₹71.31355932 |
| Forecast loss allowance | `71.31355932 × 0.03` | ₹2.13940678 |
| Estimated net pack revenue | `71.31355932 × 0.97` | ₹69.17415254 |
| Net allocation per purchased Star | `69.17415254 / 100` | ₹0.6917415254 |
| Allocation to one five-Star unlock | `0.6917415254 × 5` | ₹3.458707627 |
| Creator's 70% | `3.458707627 × 0.70` | **₹2.421095339** |
| Company's 30% | `3.458707627 × 0.30` | **₹1.037612288** |

Display approximately **₹2.42 for the creator and ₹1.04 for iAdMe per such unlock**. ₹2.42 is not the payout for the entire 100-Star pack. The 100 Stars support twenty five-Star unlocks if used entirely for that purpose.

### 11.2 New packs: per-unlock estimates at a 15% store fee

| Pack | Total Stars | Estimated net pack receipts | Creator per five purchased Stars | Company per five purchased Stars |
| --- | ---: | ---: | ---: | ---: |
| ₹99 | 100 | ₹69.174153 | ₹2.421095 | ₹1.037612 |
| ₹199 | 210 | ₹139.047034 | ₹2.317451 | ₹0.993193 |
| ₹499 | 550 | ₹348.665678 | ₹2.218782 | ₹0.950906 |
| ₹999 | 1,150 | ₹698.030085 | ₹2.124439 | ₹0.910474 |

The creator earns a different amount because the customer paid a different effective price for each Star. The public unlock price is still five Stars.

### 11.3 Stress case: 30% store fee

Keep the same 18% included tax and 3% forecast loss assumptions; change only the commission to 30%.

| Pack | Creator per five purchased Stars | Company per five purchased Stars |
| --- | ---: | ---: |
| ₹99 / 100 Stars | ₹1.993843 | ₹0.854504 |
| ₹199 / 210 Stars | ₹1.908489 | ₹0.817924 |
| ₹499 / 550 Stars | ₹1.827232 | ₹0.783099 |
| ₹999 / 1,150 Stars | ₹1.749538 | ₹0.749802 |

Additional nonrecoverable fees or fee taxes would lower the amount further. These tables exclude operating expenses.

### 11.4 Volume example using only STARTER-funded unlocks

| Five-Star purchased unlocks | Creators collectively earn | Company retains before operating costs |
| --- | ---: | ---: |
| 1 | ₹2.42 | ₹1.04 |
| 20 | ₹48.42 | ₹20.75 |
| 100 | ₹242.11 | ₹103.76 |
| 1,000 | ₹2,421.10 | ₹1,037.61 |
| 10,000 | ₹24,210.95 | ₹10,376.12 |

Totals use unrounded calculations, not the displayed ₹2.42/₹1.04 multiplied by volume. Creator totals accrue to whichever creators supplied the unlocked videos; one creator earns the total only if all unlocks were for their videos.

Across all four new packs at the 15% fee assumption, 1,000 fully purchased-Star unlocks produce approximately ₹2,124–₹2,421 for creators and ₹910–₹1,038 for the company before operating costs.

### 11.5 Mixed and promotional examples

**Two promotional Stars plus three STARTER-funded Stars:**

```text
Viewer cost                  = 5 Stars
Purchased-funded allocation  = 3 × ₹0.6917415254 = ₹2.075224576
Creator estimate             = ₹2.075224576 × 70% = ₹1.452657203
Company estimate             = ₹2.075224576 × 30% = ₹0.622567373
```

**Five welcome Stars:** viewer pays five Stars and gets access; creator and company purchase-derived revenue are both zero. Delivery still costs money.

**Two Stars from one purchased pack and three from another:** use each lot's attributable value for its portion. Do not price all five using the user's most recent or highest-value pack.

### 11.6 Creator subscription recovery example

At the 15% fee assumption, roughly **41–47 fully purchased-Star unlocks** recover a creator's ₹99 subscription expense, depending on pack mix. At the 30% assumption, the range is roughly **50–57**.

This is an arithmetic expense-recovery example, not an earnings guarantee. It excludes the creator's own production costs and taxes. Promotional-only unlocks do not contribute to that recovery. A payout minimum can delay withdrawal even when earnings have accrued.

## 12. Subscription economics and company break-even

### 12.1 Net subscription receipts

Under the same forecast assumptions:

```text
₹99 / 1.18 × 0.85 × 0.97 = ₹69.17415254  # 15% store fee
₹99 / 1.18 × 0.70 × 0.97 = ₹56.96694915  # 30% store fee
```

These are iAdMe's estimated subscription receipts before servicing and shared expenses. Do not split the Creator Plus fee 70/30 with the subscribing creator. The agreed revenue share applies to viewer unlock revenue.

### 12.2 Four-minute allowance and cost sensitivity

Ten videos at four minutes each allow **40 minutes of newly uploaded content per period**. Actual costs depend on duration, encoding outputs, storage duration, moderation, support and audience traffic. Stored content from previous periods also has an ongoing cost.

| Assumed monthly servicing cost per subscriber | Contribution with 15% store fee | Contribution with 30% store fee |
| --- | ---: | ---: |
| ₹25 | ₹44.17 | ₹31.97 |
| ₹40 | ₹29.17 | ₹16.97 |
| ₹60 | ₹9.17 | **−₹3.03** |
| ₹80 | **−₹10.83** | **−₹23.03** |

Every servicing number in this table is illustrative. The ₹25 row is included only to explain the earlier conversation, not as a validated target for the four-minute plan. The previous inline calculator also used that unverified default and is not the authoritative cost estimate.

At full 40-minute usage, allocating all net receipts to servicing would imply ceilings of about ₹1.73 or ₹1.42 per new minute at 15% or 30% fees, respectively. Those are arithmetic ceilings with **no contribution left for other company expenses**, not safe encoding budgets. Ongoing storage and support also consume the same receipts.

### 12.3 Avoid double-counting revenue and expenses

For a contribution model:

```text
Company result
  = company share of purchased-Star unlock revenue
  + net Creator Plus receipts
  + attributable net receipts from separately charged upload Stars
  + attributable net receipts from other fulfilled paid features
  + net advertising/other revenue
  − upload processing, storage, moderation and support costs
  − viewing, preview, replay and speculative download costs
  − shared infrastructure, staff, marketing and other expenses
  − separately funded creator incentives
```

Do not count the original Star-pack cash receipt again on top of the net revenue already allocated from its spent Stars. Track unspent funded Stars separately; incoming cash is not automatically free profit. Formal revenue recognition and liabilities require finance reconciliation.

Do not count ten-Star upload revenue for the ten uploads already included in a subscription. Do not charge a cost to subscriber servicing and then subtract that same cost again in a per-unlock line.

### 12.4 Break-even formulas

Let:

- `M_sub` = net subscription receipts minus the assigned per-subscriber servicing cost.
- `M_unlock` = average company unlock share minus the assigned per-unlock variable cost.
- `U` = revenue-generating unlock volume, weighted by actual purchased-Star funding rather than assuming every unlock is fully funded.
- `O` = other monthly costs not already included above.
- `R_other` = other net contribution, such as separately charged uploads or advertising.

For a simplified uniform-unlock forecast:

```text
Monthly result = subscribers × M_sub + U × M_unlock + R_other − O

Subscribers needed = ceiling(max(0, O − U × M_unlock − R_other) / M_sub)
```

The subscriber formula is valid only when `M_sub > 0`. If it is zero or negative, more subscriptions cannot solve that cost structure. In the actual model, sum per-transaction contributions rather than assuming one universal unlock margin.

Example using ₹40 servicing cost, ₹10,000 other costs and no other revenue: approximately **343 subscribers** are needed at the 15% fee assumption and **590** at the 30% fee assumption. These are illustrative workload economics, not user-growth forecasts.

### 12.5 Measurements required before claiming profitability

- Customer receipts, channel fees, tax treatment, refunds and settlement timing.
- Paid-Star purchases, unspent balances and actual consumption mix.
- Promotional-only, mixed and fully purchased unlocks.
- Watch minutes and bytes for previews, initial plays and replays.
- Public-feed delivery and extra background/preparation bytes.
- Processing cost per successful upload and per source minute, including all renditions and retries.
- Storage growth across active and cancelled creator catalogues.
- Public uploads subsidized by welcome Stars.
- Creator payout processing and support costs.
- Subscriber quota utilization, renewal, cancellation and paid-unlock conversion.

The previous ₹0.40 per-unlock figure was an illustrative operating target, not an observed CDN cost. Four-minute playback and repeated viewing can materially change the result. Do not assume advertising covers free traffic until receipts demonstrate it.

## 13. Suggested backend model

This section is an **engineering recommendation**. Reuse existing payment, wallet, media and notification infrastructure. The conceptual names below are not a requirement to create every table verbatim.

### 13.1 Purchased/promotional Star lots

Each credit source needs an immutable identity and sufficient data to attribute later consumption.

| Field group | Suggested data |
| --- | --- |
| Ownership | Lot ID, wallet/account ID, user ID |
| Provenance | Purchase, registration grant, coupon, campaign, support adjustment, legacy-unclassified |
| Quantity | Stars issued, Stars consumed/reserved/reversed, remaining quantity |
| Purchase link | Payment order, provider transaction, provider environment, catalogue version |
| Financial snapshot | Currency, gross consideration, indirect tax, fees, net funding, reconciliation status/version |
| Promotion link | Grant/campaign ID, funding source if separately sponsored |
| Audit | Created time, effective time, source event, policy version |

Do not store only an average cash value on the user record. That would obscure refunds, historical pack quantities and mixed purchases. A support adjustment must declare whether it restores an old lot, creates a new zero-funded promotion or draws from an explicitly funded budget.

### 13.2 Spend allocation records

For each Star debit, link the operation to one or more lots:

```text
spend_id
wallet_transaction_id
operation_type: premium_unlock | ordinary_upload | other_feature
operation_id
lot_id
stars_consumed
attributable_net_amount
currency
funding_state: promotional | provisional | settled | adjusted
valuation_version
created_at
```

The sum of lot allocations must equal the debit quantity. A split spend must not lose the promotional/purchased distinction. Reversals point to original allocations and restore or adjust the appropriate source; they do not become arbitrary new money.

### 13.3 Creator earnings ledger

Use append-only financial entries with at least:

- Creator and currency.
- Unlock ID and its underlying funded allocations.
- Share rate and monetization-policy version effective for the event.
- Precise gross attributable amount and creator share.
- Pending/available/held/paid classification represented by balanced transitions.
- Settlement and maturity evidence.
- Reversal/adjustment references and reason codes.
- Payout batch/provider references where applicable.

Do not overwrite a past earning entry to hide a refund. Post a linked adjustment. Do not present pending, reserved or failed-payout money as already paid.

Keep the platform's corresponding allocation reconcilable. This can be implemented within the existing ledger architecture or a dedicated accounting journal, but amounts must balance.

### 13.4 Milestones and publication grants

Suggested entities:

- `creator_milestones`: creator, milestone type/version, qualification date and evidence set.
- `creator_milestone_videos`: qualifying video IDs and eligibility evidence.
- `premium_publication_grants`: video, creator, source (`earned`, `subscription`, authorized legacy/admin route), source entitlement ID, effective dates and sale status.
- `creator_subscriptions`: provider subscription identity, original purchase identity, current verified status, renewal setting and current paid period.
- `creator_subscription_periods`: verified period ID, start/end, quota total, consumed/reserved amounts and purchase state.
- `publication_reservations`: user, video intent, period, idempotency key, lifecycle and expiry.

Allow a video to retain evidence of multiple grant sources without charging twice. Define a deterministic effective entitlement calculation; do not replace an earned grant with a subscription grant merely because the creator subscribed later.

### 13.5 Viewer entitlements

The unlock record should identify the viewer, video, Star transaction, funding classification, unlock time and policy/price version. The monetary allocation may settle later without delaying valid viewing access.

Preserve the distinction between:

- A purchased or promotional five-Star unlock.
- Owner access.
- Existing automatic-promotion prior-view access.
- Explicit support or migration grants.

Access without a paid unlock does not automatically create creator revenue.

### 13.6 Financial precision and currency

- Keep Star quantities as integers.
- Keep customer charges in integer minor units, such as paise, with currency metadata.
- Use an exact decimal or rational allocation representation with sufficient sub-paise precision. Binary floating-point is unsuitable for the authoritative ledger.
- Perform creator splits at sufficient precision and round only at a defined settlement boundary.
- Define who owns any rounding residual and keep it explicitly reconciled; do not let rounding create or destroy value silently.
- Never sum different currencies as if they were the same. INR conversion, if needed later, requires explicit rates, timestamps and policy.
- The two-decimal values displayed in examples are presentation values, not per-event rounding instructions.

### 13.7 Accounting invariants

At a consistent financial snapshot, the net funded value of a purchased lot must reconcile to:

```text
unspent funded allocation
+ company allocation from non-shared paid features such as ordinary uploads
+ creator allocation from funded unlocks
+ company allocation from those same unlocks
+ explicit rounding/financial adjustments
= attributable net funding for that lot
```

Refunds and financial adjustments must update both sides consistently through traceable entries. Creator allocation may then be split among pending, available, reserved for payout and already paid; moving between those states does not create another earning.

Key invariants:

1. One valid unlock has at most one original creator earning event.
2. One provider purchase cannot credit Stars twice.
3. One subscription period cannot grant quota twice.
4. Consumed plus reserved quota never exceeds the available period quota.
5. A normal operation cannot spend more available Stars than exist.
6. Promotional-only spend cannot create purchase-funded earnings.
7. Creator plus company shares equal the applicable funded unlock amount, subject only to explicitly managed precision residuals.
8. Same-currency creator available balance equals eligible ledger credits less reversals, holds and payouts.
9. Catalogue changes cannot rewrite an already verified purchase's quantities or original financial snapshot.
10. A cancelled creator subscription alone cannot remove a viewer's prior valid unlock.

## 14. Transaction workflows

### 14.1 Registration grant

1. Establish the new account according to the existing authentication flow.
2. Use an idempotent registration-grant identity associated with that account.
3. In one transaction, create the zero-funded 50-Star lot and matching wallet credit, or return the already-applied grant.
4. Apply the same invariant during social registration and subsequent account linking.
5. Publish notifications only after commit; failed notification delivery must not duplicate the credit.

Abuse controls should prevent repeated claims while following the app's privacy and account policies. Do not embed an undocumented assumption that one device equals one real person.

### 14.2 Purchased Star credit

1. Load the purchase intent/catalogue snapshot where applicable.
2. Verify the purchase server-side with the correct provider and environment.
3. Validate product identity, ownership/account binding, payment state, quantity, currency and revocation/refund state.
4. Reject sandbox/test evidence from real-cash creator funding.
5. Lock or uniquely claim the provider transaction.
6. Atomically create or reuse the payment order, original quantity snapshot, wallet credit and purchased funding lot.
7. Mark funding provisional where final fees or settlement are not yet known.
8. Perform provider acknowledgment/consumption using a retryable, idempotent flow appropriate to the product type.
9. Reconcile final receipts when settlement data becomes available.

A Star consumable and an auto-renewing subscription are different product types. Do not run subscription verification through a consumable-Star credit path.

### 14.3 Premium unlock

An implementation should follow the intent of this transaction, adapted to existing database patterns:

```text
BEGIN
  lock or serialize the viewer's relevant wallet/lot state
  check for an existing viewer/video access grant
  if already unlocked, return prior success without another debit
  validate video offer, owner restriction, current policy and five-Star price
  choose deterministic funding lots and validate total available balance
  insert one idempotent wallet transaction
  insert the Star debit and exact lot-consumption allocations
  insert the durable viewer/video unlock
  insert funded creator/platform allocation entries, if any
  insert an outbox event for post-commit work
COMMIT
```

Use a unique viewer/video key and operation idempotency to prevent two concurrent requests from creating two debits. A uniqueness conflict must roll back any attempted debit and return the original result. A balance check outside the transaction is insufficient.

The authoritative five-Star debit and viewer access should commit together. Revenue reconciliation may later adjust money allocations without asking the viewer to unlock again. Do not grant access on a wallet debit that rolled back, or debit Stars when the unlock record failed.

### 14.4 Ordinary upload

- Validate ownership, media type, duration and processing availability before charging where feasible.
- Reserve/charge ten Stars once using the same provenance-aware spending mechanism.
- Persist the video/publication intent and its charge linkage atomically.
- Handle processing completion/failure idempotently through the existing worker system.
- Define whether a technical failure releases a reservation or posts a source-preserving reversal; the user must not pay twice for a retry of the same failed publication.
- Only a qualifying completed public publication advances the earned milestone.
- No creator self-payout is generated by spending Stars to upload.

The exact charge timing must be reconciled with the current upload flow before implementation. Do not copy the existing separate writes without concurrency and failure analysis.

### 14.5 Included subscription premium upload/conversion

1. Verify the subscription entitlement and current period server-side.
2. Reserve one slot under a lock or conditional update.
3. Validate content or conversion eligibility and record an idempotent publication intent.
4. On successful premium publication, consume the reserved slot and create the subscription publication grant.
5. Apply **no additional ten-Star debit**.
6. On technical failure, release according to the finalized reservation policy.
7. On retries, return/reuse the original publication rather than another slot.

Existing public-video conversions still consume one slot. Deleting and recreating an intent must not bypass the quota. Moderation rejection versus technical failure requires a documented quota/refund policy.

### 14.6 Earned premium conversion

1. Verify the recorded milestone and remaining earned capacity.
2. Verify that the chosen video belongs to the qualifying set and creator.
3. Apply moderation and sale-eligibility checks.
4. Atomically create the earned grant and consume the earned selection capacity.
5. Do not charge another upload fee or subscription slot.
6. Do not invalidate the milestone when the converted video's visibility changes.

### 14.7 Provider subscription events

Maintain a verified event inbox with a unique provider event identity and environment. Handle duplicate delivery, missing events, retries, delayed renewals and out-of-order states. A periodic reconciliation process should repair missed events against provider truth.

Do not let an older expiry event revoke a later renewed period. Restoring an existing purchase must restore the same entitlement, not manufacture a new billing period. Local device time, UI state and unverified notifications are not entitlement authority.

### 14.8 Payout lifecycle

Suggested progression:

```text
earning pending
  → reconciled and matured
  → available
  → reserved for payout batch
  → provider transfer submitted
  → paid, or failed and released/retried safely
```

The proposed ₹500 minimum, monthly schedule and 30-day pending period remain open. Actual cash settlement, provider risk and finance policy must be considered before marking funds withdrawable. A 30-day timer alone does not prove that money has settled or can no longer be refunded.

Payout retries use a stable idempotency key. Reconcile uncertain provider outcomes before retrying. Cancelling Creator Plus must not make an otherwise valid accrued earning disappear or require a new subscription to withdraw it.

## 15. Refunds, reversals, fraud and corrections

### 15.1 Purchased-Star refunds

Link the refund to its original purchase lot and provider event. Separate unspent Stars, already consumed Stars and creator amounts already paid.

- Unspent refunded credits: reverse the appropriate remaining Star quantity under the finalized policy.
- Consumed credits: record the financial adjustment and identify affected earnings; do not erase history.
- Pending earnings: adjust or hold the linked amount where the policy permits.
- Already paid earnings: record the exposure/receivable and follow explicit recovery rules. Do not silently debit an unrelated bank account.
- Partial refunds: use documented quantity/value allocation with reconciliation; do not assume every provider refund neatly corresponds to whole Stars.
- Restored credits for a technical failure: restore the original provenance and valuation linkage.

The same refund must not be applied twice because it arrived through a webhook and a reconciliation job. A refunded purchase must not continue creating new cash earnings from its original funding.

### 15.2 Viewer access and refund scope

A viewer's own refunded content transaction may require an access adjustment, but refund scope must be mapped precisely. A creator's subscription refund is not automatically a refund of all unrelated viewers' purchases.

Do not implement a global “disable creator account” action as the sole response to ordinary subscription cancellation. Entitlement, account safety and financial debt are different concerns.

### 15.3 Abuse controls directly relevant to this model

- Block owner self-unlock revenue.
- Detect coordinated purchased-Star/refund loops and repeated welcome-grant abuse.
- Do not credit real earnings from sandbox or synthetic test purchases.
- Check server-side access to the original video and full media, not only a client paywall.
- Detect quota recycling through rapid publication/deletion/reconversion.
- Avoid reward rules based solely on easily duplicated public upload counts.
- Keep creator verification, payout destination validation and financial adjustments auditable.

Use proportionate controls and an appeal/support path. Do not invent legal eligibility ages, tax withholding rates or identity-document requirements in code from this document; these are separate finance/legal/product decisions.

### 15.4 Support/admin actions

Administrative grants, balance corrections, entitlement changes and payout holds require a reason, actor identity, source references and audit timestamp. Distinguish restoring a user's existing entitlement from granting a new promotional benefit. Staff should not be able to label unfunded Stars as purchased without an actual funding event.

## 16. API and configuration design

Endpoint names below are suggestions. Reuse the current API's naming and error conventions.

### 16.1 Server read models

| Capability | Minimum response data |
| --- | --- |
| Pack catalogue | Catalogue version, product IDs, price/currency, base/bonus/total Stars, effective availability |
| Wallet | Total spendable Stars, purchased/promotional breakdown, funding-aware transaction history |
| Creator eligibility | Qualifying public count, target 15, milestone status, earned selections and available capacity |
| Creator subscription | Provider-verified plan/status, renewal setting, period start/end, quota consumed/reserved/remaining |
| Premium publication check | Allowed source(s), relevant costs/slots, denial reason and next eligible date where applicable |
| Video access | Existing viewer entitlement, current price if purchasable, preview/access state, purchase availability |
| Creator earnings | Pending, available, held and paid INR amounts; promotional/mixed/purchased counts; statement period |
| Payout history | Batch/provider status, amount, currency, adjustments and failure/retry state safe to disclose |

Server responses should explain a denied action specifically: quota exhausted, subscription expired, insufficient Stars, video not eligible, content restriction or already unlocked. Do not turn every failure into a generic “Buy premium” prompt.

### 16.2 Proposed write operations

- Verify a store subscription or restore an existing verified subscription.
- Create a publication reservation/intent.
- Select an eligible earned premium video.
- Request an idempotent premium unlock.
- Request a payout only if the final payout policy supports user-initiated withdrawals.
- Receive verified provider events and reconcile financial statements.

All financial writes require authenticated ownership checks. Store secrets, payout credentials and settlement details stay server-side. Never accept a client-supplied cash value for Stars, creator share or remaining quota.

### 16.3 Configuration candidates

| Setting | Working value | Notes |
| --- | --- | --- |
| `wallet.registrationBonusStars` | 50 | Existing key; affects new grants only. |
| `uploads.costStars` | 10 | Existing key; not charged on included subscription premium uploads. |
| `premium.defaultUnlockStars` | 5 | Existing key; inspect all environment overrides and old-client behavior. |
| Creator net revenue share | 7,000 basis points | Versioned policy; snapshot at earning event. |
| Earned qualification count | 15 | Count qualifying public videos. |
| Earned premium selection capacity | 5 | One-time milestone is working baseline. |
| Subscription publication quota | 10 | Per verified billing period. |
| Maximum video duration | 240,000 ms | Keep four-minute owner requirement. |
| Promotional cash earnings | Disabled by baseline | Campaign-funded exceptions need their own budgets. |
| Payout minimum/hold/schedule | Unresolved | Do not silently set the earlier proposal as final. |
| Spending order | Unresolved; promo-first recommended | Version and disclose. |

New key names should follow current configuration conventions. Do not hard-code a single store fee or a 3% refund deduction as a universal payout setting. Price changes affect future transactions through an explicit version, not historical ones.

The existing `subscription.discountPercent` must be audited and either made irrelevant to Creator Plus or explicitly redesigned. Do not accidentally halve the five-Star viewing price because a creator subscribed.

## 17. Mobile, web and creator experience

### 17.1 Wallet and pack purchase

- Show “Stars” rather than a rupee symbol for the Star balance.
- Show actual customer price, base Stars, bonus Stars and total Stars before purchase.
- Explain that Stars have no fixed 1:1 INR value.
- Make welcome/promotional versus purchased provenance understandable without requiring users to read a ledger.
- Explain the finalized spending order.
- Preserve the displayed and verified quantity for purchases made around a catalogue change.

### 17.2 Viewing and unlock

- The normal action says “Unlock for 5 Stars”; it does not say “Purchased Stars only.”
- A balance containing welcome Stars can complete the same unlock.
- Insufficient-balance UI shows the current balance and shortfall accurately.
- An already unlocked video opens without another charge.
- Expired creator membership does not show a new purchase prompt to an existing buyer.
- A video unavailable for new sales explains that status without exposing private account/payment details.

### 17.3 Creator onboarding

Present the two routes side by side: 15 qualifying public videos for five earned premium selections, or Creator Plus for immediate monthly capacity.

Before enrolment, explain:

- Eligibility does not guarantee audience purchases or income.
- Promotional-only unlocks provide access but do not generate purchase-funded earnings under the baseline.
- The 70% share is of attributable net revenue, not of a nominal five rupees.
- Both routes remain subject to content rules.
- Sharing is optional.

Display milestone progress from the server. Do not imply that subscribing is mandatory for creators who already earned their five-video benefit.

### 17.4 Upload/conversion flow

- Show the route used by this publication: ordinary ten-Star upload, earned premium selection, or included subscription slot.
- Show “3 of 10 publications used this billing period” and the renewal date for Creator Plus.
- Confirm that converting an existing public video uses a slot before doing so.
- Enforce the four-minute limit in both client and server validation.
- Prefer precise duration validation in milliseconds; accepting `duration.inSeconds <= 240` alone can let a file slightly over four minutes slip through due to truncation.
- Use “up to 4 minutes” consistently if exactly 240 seconds is allowed.
- Never debit ten Stars and consume a subscription slot for the same included publication.

### 17.5 Creator earnings dashboard

Display distinct figures for pending, available, held and paid earnings, with currency. Explain estimates and final settlement adjustments. Show total, promotional, mixed and purchased unlocks distinctly.

Provide a statement that lets a creator understand an earning without exposing the viewer's identity, payment credentials or another user's transaction details. Aggregate appropriately while retaining an internal auditable link.

### 17.6 Account and subscription management

Show the verified plan, renewal setting, paid-through date, remaining publication quota, and the appropriate provider subscription-management route. When cancellation is pending, say the benefits remain active until the paid-through date.

At expiry, show which earned videos remain eligible and which subscription-only offers have stopped accepting new purchases. Accrued earnings and payout history must remain accessible.

### 17.7 Four-minute playback and background preparation

Preserve the existing next-two preparation fast path, one-player ownership and bounded cache behavior. Do not disable or redesign playback as an incidental part of monetization.

The documented background feature uses an approximately 200 MB video disk-cache allowance and a separate extra-preparation traffic budget. OS scheduling is best effort; do not promise guaranteed background execution or offline availability of premium content.

Full paid media and entitlement-sensitive responses must not leak through a public preview, public URL or background snapshot. Review thumbnail, preview, playlist, segment and signed-URL access together. Preserve valid cached/replay behavior for authorized users while respecting account changes and revocations. Reading a cached entitlement claim alone must not become server authorization.

## 18. Policy and FAQ content drafts

These are implementation-aligned drafts, not a claim that existing public policies have been changed. Update only relevant policy areas. A pricing change by itself does not require unrelated privacy-policy edits; new payout/verification data collection may require them.

### 18.1 Core Stars clause

> Stars are iAdMe's in-app credits. There is no fixed 1:1 exchange rate between Stars and Indian Rupees: ₹1 does not necessarily purchase 1 Star, and 1 Star is not redeemable for ₹1. The number of Stars supplied depends on the selected pack and any included bonus. A user's Star balance is separate from eligible creator cash earnings, which are calculated under the Creator Monetization Policy.

### 18.2 Welcome-Star clause

> Eligible new users receive 50 welcome Stars once. Welcome Stars can be used for eligible iAdMe features, including premium-video unlocks and ordinary video uploads. At current prices, 50 Stars can unlock ten five-Star premium videos, fund five ten-Star uploads, or be used for a combination of those actions.

Add the final eligibility and spending-order rules. Do not describe both full allowances as simultaneous benefits. Do not introduce a retrospective expiry for purchased Stars.

### 18.3 Creator revenue clause — baseline requiring confirmation

> Eligible creators receive 70% of the net revenue attributable to purchased Stars used to unlock their videos. Net revenue reflects the relevant purchase price, applicable transaction taxes, store or payment fees, refunds and adjustments described in this policy. Pack bonus Stars share the revenue allocation of their purchased pack. Free promotional Stars have no purchase revenue behind them; unlocks funded entirely with those Stars do not generate creator cash earnings unless a separately announced funded promotion applies. Mixed unlocks earn on the purchased-Star portion only.

The final policy must define allowed deductions precisely. It must not allow arbitrary operating expenses to be silently deducted before the 70% split. Ordinary processing, hosting and support are paid from the company's revenue share and other revenue unless a different commercial model is explicitly agreed.

### 18.4 Creator Plus benefit clause

> Creator Plus costs ₹99 per month at the advertised India price and immediately enables premium publishing after successful payment verification, subject to content rules. Each billing month includes up to ten premium-video publications, with no additional upload-Star charge for those included publications. Each video may be up to four minutes long. Converting an existing video to premium uses one publication slot. Unused slots do not roll over.

Supplement with the actual store's price display, billing/renewal disclosures, period dates, management link and finalized failure/quota rules. Do not promise ten successful moderated publications if the user submits disallowed content.

### 18.5 Viewing versus publishing FAQ

**Does Creator Plus let me watch every premium video for free?**

> No. Creator Plus provides premium-publishing capacity for your own videos. Unlocking another creator's premium video still costs five Stars. You can use available welcome/promotional or purchased Stars for that unlock. The subscription does not itself add viewing Stars under this plan.

**Can I watch premium videos using my welcome Stars?**

> Yes. A five-Star premium unlock can use your welcome Stars and provides the same viewing access as an unlock funded with purchased Stars.

**Why did some unlocks generate no earnings?**

> Viewers can use free welcome or promotional Stars. Those unlocks do not have purchase revenue behind them. Your dashboard separates promotional, mixed and purchased unlocks and shows the earnings attributable to purchased Stars.

**Do 100 Stars mean a creator earns ₹100?**

> No. Stars are app credits, not a fixed rupee amount. Creator earnings depend on how many purchased Stars viewers actually spend on their videos, the revenue attributable to those Stars, and the creator revenue share.

### 18.6 Earned creator FAQ

**Can I publish premium videos without subscribing?**

> After completing 15 qualifying public-video uploads, you can select five of those videos to become premium under the earned-creator route. The five earned selections do not require a monthly subscription under this plan. Ordinary uploads cost ten Stars each.

**Can I reach the milestone with one ₹99 Star pack?**

> If you still have all 50 welcome Stars, adding the 100-Star ₹99 pack gives you the 150 Stars needed for fifteen ten-Star uploads. If you have spent some of your welcome Stars, you may need additional Stars. Uploads must still meet the qualification and content rules.

**Must I share videos on social media?**

> No. Sharing is encouraged but is not required to obtain these benefits.

**Do I receive five more earned premium videos every month?**

> Under the proposed earned route, the five selections are a one-time milestone benefit. Creator Plus provides a recurring monthly publication allowance.

The last answer depends on confirming the one-time milestone baseline and final replacement policy.

### 18.7 Cancellation FAQ

**What happens when I cancel Creator Plus?**

> Turning off renewal does not immediately end your paid benefits. They continue until the end of your current paid billing period. After expiry, subscription-only premium publishing and new sales of those subscription-only videos stop under this plan. Viewers who already unlocked a video retain their access, and your accrued earnings are preserved. Premium videos separately earned through the 15-upload milestone remain eligible under the earned route.

Add separate treatment for provider refunds, revocations, grace periods, content removal and legal restrictions. Do not present these exceptional events as ordinary cancellation.

### 18.8 Earnings and payout FAQ

**Does paying ₹99 guarantee earnings?**

> No. The subscription provides publishing capacity. Earnings depend on eligible revenue-generating unlocks of your content and the applicable creator terms.

**Is 70% calculated from five rupees each time?**

> No. It is 70% of the net revenue attributable to the purchased Stars used in the unlock. A five-Star unlock is not necessarily a five-rupee transaction.

**When can I withdraw?**

> Publish this answer only after the settlement schedule, verification requirements, minimum amount, pending period, payout fees and adjustment rules are finalized.

Do not publish the earlier ₹500/30-day/monthly proposal as final merely because it appears in this reference.

### 18.9 Content synchronization matrix

| Surface | Required content |
| --- | --- |
| Terms and Conditions | Stars versus INR; separate creator earnings; plan and content obligations |
| Stars/Payments Policy or equivalent section | Pack quantities, promotional use, spending order, non-expiry where required, refunds and provenance |
| Creator Monetization Policy | Eligibility, net-revenue definition, 70/30 split, zero-funded unlocks, adjustments and payout terms |
| Subscription terms/paywall | ₹99 recurring price, ten publications, four minutes, immediate eligibility, no rollover, cancellation |
| Refund/Cancellation Policy | Provider purchase refunds, earning reversals, period-end cancellation, existing viewer rights |
| FAQs/help | Simple explanations and examples consistent with the actual product |
| Wallet/purchase screens | Correct quantity/bonus/price and no fixed INR redemption promise |
| Upload/conversion screen | Ten-Star charge or included slot; conversion consumes quota; earned-route eligibility |
| Creator dashboard | Funding-based earnings, separate pending/available/held/paid amounts |
| Privacy Policy | Update only if payout/verification/provider data processing changes; describe actual new handling |
| Support and in-app assistant content | Same approved rules; no stale claims about free viewing or rupee conversion |

## 19. Migration and compatibility

### 19.1 Preserve existing customer value

Do not edit existing wallet balances to match the new welcome grant or new pack totals. A historic ₹999 purchase that supplied 1,300 Stars remains a 1,300-Star purchase. If its funding can be reconstructed, its actual net receipts are allocated across **1,300**, not the new 1,150.

Preserve existing legitimate video unlocks. A new pricing model does not authorize resetting `premium_unlocks` or charging prior buyers again. Historical SQL scripts that reset premium videos/unlocks must not be reused as migration shortcuts.

### 19.2 Catalogue transition

1. Create a new effective catalogue version with the agreed future quantities.
2. Snapshot catalogue/version/quantity at the correct purchase boundary.
3. Decide whether each store product requires a new identifier or metadata update. Verify provider requirements and outstanding purchases; do not assume reusing an ID is safe.
4. Coordinate backend fulfilment, app display and store descriptions so an old promise is not fulfilled with fewer Stars.
5. Handle purchases initiated before the change and completed afterward using provable advertised/transaction context.
6. If older clients cannot reliably convey or recover that context, use a compatible fulfilment rule or separate products rather than under-crediting users.
7. Preserve old purchase snapshots and provider transaction identities.

The current app model receives totals from the API, while store metadata may have its own description. Both surfaces must agree for the relevant transaction.

### 19.3 Existing Star provenance

Backfill lots from verified payment orders, registration events, coupons, support adjustments and wallet history where evidence permits.

- Reconcile reconstructed lot balances exactly to the current Star ledger.
- Do not classify all existing Stars as purchased or all as promotional for convenience.
- Old spending may not have recorded which source was consumed. If the remaining paid/promotional split cannot be established, label it `legacy_unclassified` and adopt an explicit funded migration policy.
- A migration policy must preserve the user's legitimate access/spending rights while controlling new cash liabilities; unknown funding is not proof of customer misconduct.
- Do not start cash payouts based on guessed backing. Resolve or explicitly fund the legacy exposure before enabling the affected earnings path.
- Do not award historical creator earnings retroactively unless separately authorized and funded.

### 19.4 Existing premium videos and automatic promotion

Inventory existing premium videos by source: creator/admin selection, automatic promotion, or other historical behavior. Decide how each source maps into earned/subscription/legacy grants before activation.

The existing automatic-promotion system can promote public videos independently of the new creator routes. The implementation must choose explicitly whether it continues, requires creator eligibility, or uses a separate monetization program. Do not allow an automatic process to bypass the new publishing rules unintentionally.

Preserve existing automatic-promotion prior-view access where applicable. Such access is not a newly purchased unlock and must not manufacture creator earnings.

### 19.5 Subscription launch

Create and verify subscription products separately from Star consumables. Provider notifications, restore, cancellation, expiry, renewal and reconciliation must work before the paid plan is offered.

Do not infer an existing user's active subscription from a client-side flag or an unused configuration key. Do not retroactively charge users for previously uploaded videos.

### 19.6 Rollback

Use additive migrations and independent feature flags where feasible. A rollback can stop new subscription sales, new cash-earning events or new premium publications while preserving:

- Valid existing viewer access.
- Verified purchases and historical quantities.
- Earned and paid-period entitlements already sold.
- Accrued financial obligations and payout records.

Do not delete financial ledgers, erase purchases or repurpose wallet balances to roll back a feature. Reconcile any partially processed event before enabling retries.

## 20. Delivery phases and implementation checklist

### Phase 1 — Product and financial definitions

- [ ] Record decisions for the open commercial items in section 23.
- [ ] Confirm the exact net-revenue field mapping for each provider.
- [ ] Confirm catalogue transition and historical-balance treatment.
- [ ] Establish the initial cost budget and measurement plan for four-minute media.
- [ ] Agree customer and creator copy, including promotional-only earnings treatment.

### Phase 2 — Core accounting and migration foundations

- [ ] Extend versioned purchase snapshots and funding provenance.
- [ ] Implement exact spend allocation across purchased/promotional lots.
- [ ] Add creator/platform allocation journals and reconciliation.
- [ ] Make unlock debit, access and original earning attribution atomic and idempotent.
- [ ] Dry-run legacy reconstruction and report unresolved balances.
- [ ] Add source-preserving refund and correction paths.

### Phase 3 — Packs and welcome grant

- [ ] Add the new catalogue without altering old purchases.
- [ ] Align store products/descriptions and app display.
- [ ] Change new registration grants to 50 in both normal/social flows.
- [ ] Permit welcome Stars for premium viewing with correct provenance.
- [ ] Verify the five-Star configuration across supported environments and clients.

### Phase 4 — Earned creator route

- [ ] Define and implement qualifying public publication evidence.
- [ ] Persist the 15-video milestone and qualifying set.
- [ ] Allow five earned premium selections without duplicate upload/slot charges.
- [ ] Preserve milestone validity after legitimate promotion changes visibility.
- [ ] Resolve interaction with automatic promotion and historical premium videos.

### Phase 5 — Creator Plus

- [ ] Implement provider subscription verification, notifications and reconciliation.
- [ ] Add per-period quota reservations and consumption.
- [ ] Include ten premium publications with no additional Star charge.
- [ ] Count public-to-premium conversion as one slot.
- [ ] Retain four-minute client/server validation.
- [ ] Handle renewal, restore, cancellation, expiry and account hold.
- [ ] Preserve prior viewer access and separately earned rights.

### Phase 6 — Earnings, payout and policies

- [ ] Show funding-based earnings and clear activity categories.
- [ ] Implement the finalized pending/available/reserve/payout lifecycle.
- [ ] Reconcile settlement and payout-provider outcomes.
- [ ] Update relevant public policies, FAQs, paywall, wallet and support content.
- [ ] Verify creator acceptance of the final monetization terms where required.

### Phase 7 — Controlled release and observation

- [ ] Run the acceptance matrix below in isolated test environments.
- [ ] Complete provider sandbox tests without generating real payout liabilities.
- [ ] Run end-to-end tests on supported Android/iOS purchase flows.
- [ ] Verify a limited real transaction and its settlement through the authorized release process.
- [ ] Enable in stages with dashboards and reversible controls.
- [ ] Review actual servicing costs, refunds and conversion before scaling.

During mobile work, follow `IADME_BUILD_DEPLOY_COMMANDS.md` and `iadme-mobile/AGENTS.md`. Keep dev API/worker services running; restart any service stopped for necessary work. Preserve the next-two playback fast path and existing background platform constraints. Store upload/deployment is separate from building an artifact or editing this reference.

## 21. Acceptance-test matrix

These are target tests for the future implementation. Their presence here does not mean they have been run or passed.

### 21.1 Catalogue, registration and wallet

| ID | Scenario | Expected result |
| --- | --- | --- |
| CAT-01 | New purchase of each revised pack | Exactly 100/210/550/1,150 Stars, matching the verified product/version. |
| CAT-02 | Duplicate verification or webhook | One purchase credit and one funding lot. |
| CAT-03 | Old 1,300-Star MEGA purchase after catalogue change | Original quantity and financial snapshot retained. |
| CAT-04 | Purchase begins before transition and finishes after | Fulfilment matches the valid historical promise. |
| CAT-05 | Sandbox purchase submitted to production earning flow | No real-cash funding or payout liability. |
| REG-01 | New ordinary registration | One 50-Star zero-funded grant. |
| REG-02 | New social registration | Same one-time 50-Star result. |
| REG-03 | Retry, account link, reinstall or login | No duplicate grant. |
| REG-04 | Existing account at launch | Existing balance preserved; no automatic extra grant. |
| WAL-01 | Spend across promotional and multiple purchase lots | Debit equals allocation quantities; deterministic source order. |
| WAL-02 | Concurrent spends with insufficient combined balance | At most valid affordable operations succeed; no hidden overspend. |

### 21.2 Unlocks and earnings

| ID | Scenario | Expected result |
| --- | --- | --- |
| UNL-01 | Five welcome Stars unlock a video | Five-Star debit, durable access, zero purchase-derived earnings. |
| UNL-02 | Five purchased Stars unlock | Access plus correct 70/30 net allocation. |
| UNL-03 | Two promotional plus three purchased Stars | Full access; money allocation uses only the purchased three. |
| UNL-04 | Purchased Stars from different packs | Each portion uses its own purchase snapshot and funding. |
| UNL-05 | Replay an unlocked video | No new charge or earning. |
| UNL-06 | Concurrent duplicate unlock requests | One debit, one access record, one original earning event. |
| UNL-07 | Failure between debit/access/earning writes | Transaction rolls back; no partial financial state. |
| UNL-08 | Creator attempts to buy own video | No self-generated earning; owner access handled separately. |
| UNL-09 | Client sends manipulated price/share | Server price and policy prevail. |
| UNL-10 | Automatic-promotion prior viewer | Existing access preserved without a fabricated paid unlock. |
| UNL-11 | Public preview/background response | No unauthorized full premium-media access. |
| UNL-12 | Historical unlock after new price activation | No second charge. |
| FIN-01 | STARTER illustration with t=.18, f=.15, r=.03 | Per-five forecast ≈₹2.421095339 creator and ₹1.037612288 company. |
| FIN-02 | Same illustration at volume 1,000 | ≈₹2,421.095339 / ₹1,037.612288 before display rounding. |
| FIN-03 | Display rounding versus settlement | Totals reconcile from precise allocations, not rounded UI values. |
| FIN-04 | Final net receipts differ from estimate | Linked, auditable adjustment; provisional status clears appropriately. |

### 21.3 Earned route and publication

| ID | Scenario | Expected result |
| --- | --- | --- |
| ERN-01 | Fourteen qualifying public videos | Milestone not yet granted. |
| ERN-02 | Fifteenth qualifies | One milestone with evidence; five-selection capacity. |
| ERN-03 | Promote first five from qualifying set | No extra upload fee; milestone persists after visibility changes. |
| ERN-04 | Attempt a sixth earned selection | Rejected under the one-time five-selection baseline. |
| ERN-05 | Draft, failure, photo or duplicate retry | Does not increment the video milestone. |
| ERN-06 | No external social share | Does not block otherwise valid qualification. |
| ERN-07 | Concurrent fifth/sixth earned selections | Capacity cannot be exceeded. |
| ERN-08 | Choose a video outside the eligible set | Final qualification/selection rules enforced. |
| PUB-01 | Included subscription publication | One slot, zero extra ten-Star debit. |
| PUB-02 | Public-to-premium conversion | One slot; no media reupload required merely for entitlement. |
| PUB-03 | Technical failure then retry | Correct reservation release/reuse; no duplicate charge or quota consumption. |
| PUB-04 | Delete a successfully published subscription video | No automatic quota recycling. |
| PUB-05 | Exactly 240,000 ms video | Accepted if otherwise valid. |
| PUB-06 | 240,001 ms or manipulated client metadata | Server rejects or requires valid shortening; no truncation bypass. |

### 21.4 Subscription and access lifecycle

| ID | Scenario | Expected result |
| --- | --- | --- |
| SUB-01 | First verified ₹99 subscription with zero public uploads | Immediate subscription publishing eligibility and one period's quota. |
| SUB-02 | Ten publications consumed | Eleventh blocked; no hidden Star overage charge. |
| SUB-03 | Two concurrent final-slot requests | At most one obtains the final slot. |
| SUB-04 | Genuine verified renewal | One new period with ten slots; no rollover. |
| SUB-05 | Restore or duplicate renewal event | Same period restored; no duplicate quota. |
| SUB-06 | Cancel auto-renew before paid end | Benefits remain until verified period end. |
| SUB-07 | Paid period expires | Subscription-only publishing and new sales stop under baseline. |
| SUB-08 | Prior buyer plays after creator expiry | Existing access remains. |
| SUB-09 | Creator also holds earned five | Earned grants remain eligible after subscription expiry. |
| SUB-10 | Creator cancels with accrued earnings | Earnings/payout eligibility are not erased by cancellation. |
| SUB-11 | Old expiry arrives after newer renewal | New period is not incorrectly revoked. |
| SUB-12 | Upload reservation crosses period boundary | Finalized timing rule applied once to the correct period. |
| SUB-13 | Subscription user unlocks another creator's video | Normal five-Star debit; no unintended 50% discount. |
| SUB-14 | Grace, hold, refund or revocation | Verified provider state and finalized rules applied distinctly. |

### 21.5 Refunds, payout, migration and presentation

| ID | Scenario | Expected result |
| --- | --- | --- |
| REF-01 | Full refund before any Stars spent | Appropriate lot reversal and reconciled financial state. |
| REF-02 | Partial refund after some funded unlocks | Precise adjustments; no duplicate loss deduction. |
| REF-03 | Same refund via event and reconciliation | Applied once. |
| REF-04 | Technical credit restoration | Original source/provenance restored. |
| PAY-01 | Transfer submission times out | Reconcile provider outcome before any retry. |
| PAY-02 | Transfer fails definitively | Funds safely released/retried without double payment. |
| PAY-03 | Creator has only promotional unlocks | No purchase-funded available payout. |
| PAY-04 | Refund affects already paid earnings | Explicit exposure/recovery entries; history preserved. |
| MIG-01 | Backfill known historical sources | Totals reconcile exactly to legacy wallet balance. |
| MIG-02 | Ambiguous historic remaining balance | Explicit unresolved/funded policy; no guessed cash backing. |
| MIG-03 | Roll back new sale features | Existing access, purchases and accrued obligations preserved. |
| UX-01 | Wallet, packs, Terms and FAQs | Same quantities and no fixed 1:1 INR claim. |
| UX-02 | Creator statement | Promotional/mixed/purchased activity and money states distinguishable. |
| UX-03 | App purchase and restore on both platforms | Verified result matches provider and backend; no duplicate benefits. |
| UX-04 | Login/account switch with cached media | Account-bound access and cache invalidation remain correct. |

## 22. Observability, reconciliation and operational review

### 22.1 Metrics

Record operational counts and financial totals separately:

- Welcome grants and duplicate-grant attempts.
- Pack sales by provider, product, catalogue version and currency.
- Purchased/promotional/legacy-unclassified Star balances and consumption.
- Unlock success/failure/retry counts and funding categories.
- Actual attributable creator and platform amounts, adjustments and unreconciled totals.
- Milestone qualification, earned selections and invalidation/review events.
- Subscription activation, renewal, cancellation, expiry and restore.
- Per-period quota reservations, consumption, release and contention failures.
- Payout balances, batches, provider statuses and unresolved outcomes.
- Media processing, stored bytes, delivered bytes, replay and preparation traffic.

### 22.2 Reconciliation jobs

Use appropriate scheduled or operational reconciliation to compare:

1. Provider purchases against credited wallet lots.
2. Wallet debits against spend allocations and unlock/publication records.
3. Subscription provider periods against entitlement periods and quota.
4. Settlement reports against provisional net funding.
5. Funded unlock allocation against creator/platform journals.
6. Payout journals against provider-confirmed transfers.
7. Legacy migration sums against the original ledger snapshot.

Unreconciled amounts should be visible and held from payout where necessary, without withholding unrelated valid balances indiscriminately.

### 22.3 Cost review

Review actual contribution by cohort and channel. A positive average can hide loss-making four-minute-heavy subscribers or high-replay videos. Include a mature catalogue scenario, because storage and prior buyers' access continue after initial upload and can outlive a subscription.

Separate acquisition spending on welcome Stars from ordinary paid-user unit economics. Establish a welcome-grant abuse limit and a promotional budget based on observed usage. Do not imply free Stars cost nothing merely because no creator cash payout is generated.

The initial 30-day observation window is a recommendation, not a scheduled automation. This document does not create monitoring jobs or reminders.

## 23. Open decisions to resolve for the affected feature

The following must not be silently treated as owner-approved. Some have recommended baselines to keep implementation moving.

| ID | Decision | Current recommendation or boundary |
| --- | --- | --- |
| D01 | Cash treatment of promotional-only unlocks | Zero purchase-derived payout; separately funded campaign if desired. Clearly disclose to creators. |
| D02 | Earned milestone recurrence | One lifetime/account milestone yielding five selections; no repeated monthly grant by default. |
| D03 | Replacing/swapping earned selections | Not defined. Decide limits and preserve previous buyers' access. |
| D04 | Public qualification and retention | Define duplicates, minimum public availability, deletion and policy-violation treatment without revoking the milestone solely because five were promoted. |
| D05 | Earned/subscription overlap | Persist independent grant sources; define any user choice of route and prevent double charging. |
| D06 | Welcome-Star spending order | Promo-first followed by purchased FIFO recommended; disclose final rule. |
| D07 | Subscription-only offers at expiry | Stop new sales, retain prior buyers, preserve earned five; baseline needs final adoption. |
| D08 | Billing grace, hold and revocation | Follow verified provider states; specify exact business entitlement rules. |
| D09 | Upload reservation timing/failure | Decide bounded completion window across expiry, moderation versus technical failure, and charge/release timing. |
| D10 | Payout minimum, schedule and pending period | Earlier ₹500/monthly/30-day proposal remains unconfirmed. Reconcile cash availability and provider risk. |
| D11 | Verification, withholding and payout fees | Determine with finance/legal/provider requirements; do not invent statutory rates or rules. |
| D12 | Exact net-revenue deduction definition | Actual provider accounting, applicable tax and fees; no automatic permanent 3% haircut. |
| D13 | Legacy ambiguous Star funding | Preserve legitimate balances; explicitly reconcile or fund exposure before cash payouts. |
| D14 | Historical creator earnings | No retroactive earnings unless separately approved and funded. |
| D15 | Existing premium and automatic promotion | Define eligible grant source, creator consent and interaction with new routes. |
| D16 | Store product/catalogue migration | Determine identifiers, effective version and old-client/in-flight purchase treatment. |
| D17 | Four-minute ₹99 plan economics | Measure costs; ₹99 is the requested design price, not a verified profitable price. |
| D18 | Previously purchased content after deletion/removal | Define storage, moderation, refunds and access exceptions; cancellation alone is not deletion. |
| D19 | Extra premium publications beyond ten | No automatic overage charge or higher tier in this scope. |

Record future resolutions with dates and policy versions in this file or a linked decision record. Update affected FAQs, tests and migrations together. Do not resolve a commercial ambiguity by quietly shipping restrictive behavior that was never explained.

## 24. Scope boundaries

This design does not itself authorize or specify:

- Guaranteed creator income, returns or a fixed rupee redemption rate for Stars.
- Cash withdrawal of purchased or promotional viewer Stars.
- Paying cash for all free unlocks without a funded incentive program.
- An annual plan, extra subscription tiers or unlimited viewing.
- International currency conversion or country expansion.
- A redesign of public feed ranking, ads frequency or playback caching.
- The full visual design of the separate creator profile on `iadme.app`.
- Retroactive removal of existing balances or valid unlocks.
- Automatic migration of every existing premium video to a paid membership requirement.
- Store submission, production deployment, live payout transfers or public policy publication merely because this Markdown exists.

## 25. Source notes and verification scope

### 25.1 Owner discussion

This document consolidates the pricing and creator-monetization discussion and owner annotations on 12 October 2026. The decision register records which items were explicit owner decisions and which came from follow-up recommendations. The owner's reported low current user base and unknown operating costs are why costs remain illustrative.

### 25.2 Official external references

Consulted during the discussion; verify current terms again before implementing provider-specific behavior or publishing prices/policies:

- [Google Play service fees](https://support.google.com/googleplay/android-developer/answer/112622?hl=en): eligible fee tiers and subscription fees; do not assume one universal commission.
- [Apple App Store Small Business Program](https://developer.apple.com/app-store/small-business-program/): eligibility and the reduced commission program.
- [Apple auto-renewable subscriptions](https://developer.apple.com/app-store/subscriptions/): subscription proceeds and lifecycle guidance.
- [Google Play tax rates and VAT](https://support.google.com/googleplay/android-developer/answer/138000?hl=en-en): market/developer-location tax handling; use actual applicable treatment.
- [Apple App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/): in-app purchases, purchased-credit non-expiry and subscription presentation requirements.
- [CBIC service-rate reference](https://cbic-gst.gov.in/hindi/gst-goods-services-rates.html): background for tax review; the 18% examples are not a determination of iAdMe's final classification.
- [AWS CloudFront pricing](https://aws.amazon.com/cloudfront/pricing/pay-as-you-go/): a reference for cost investigation, not the source of a verified iAdMe delivery bill.

Actual applicable provider agreements, tax classification, fee-tax recoverability and settlement reports control financial implementation. The formulas in the illustration deliberately expose assumptions instead of presenting them as confirmed business expenses.

### 25.3 Local related references

- [IADME-043 and surrounding backlog](BACKLOG.md)
- [Apple Stars billing notes](../compliance/apple-app-store-billing-stars.md)
- [Google Play Stars billing notes](../compliance/google-play-billing-stars.md)
- [Build and deployment reference](../runbooks/IADME_BUILD_DEPLOY_COMMANDS.md)
- [Background reels constraints](../../iadme-mobile/docs/background-reels-2026-09-18.md)

### 25.4 Document validation versus application validation

This reference was checked for calculation consistency, source pointers, Markdown structure and coverage of the discussion. No app feature is marked implemented by that review. The acceptance tests in section 21 remain future implementation requirements. Record actual migrations, test runs, provider verification, builds and releases in their normal evidence locations when that work is performed.

## 26. Compact handoff checklist

Before coding the relevant feature, an implementer should be able to answer:

1. Which rows in section 2 are owner-confirmed, and which open decisions affect this change?
2. Which current data can prove a Star's funding source?
3. How are retries and concurrent requests prevented from charging or paying twice?
4. Which grant lets this creator publish this particular premium video?
5. Does this operation consume Stars, a subscription slot, or an earned selection—and exactly one appropriate charge?
6. What happens to existing buyers when the creator stops subscribing?
7. Are examples being mistaken for actual settlement data or measured costs?
8. How will old catalogue quantities, old balances and already unlocked videos be preserved?
9. Which policy/FAQ/screens and acceptance tests must change alongside the behavior?
10. What evidence will demonstrate correct finances, viewer access and four-minute-plan costs before rollout?
