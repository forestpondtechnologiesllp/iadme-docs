# iAdMe Local Market — discussion and implementation record

Recorded: 2026-09-30 (Asia/Kolkata). Updated: 2026-10-02. Parent backlog: IADME-040.

**2 October update:** [UI decisions and design record](LOCAL_MARKET_UI_2026-10-02.md) supersedes earlier seven-day visibility, optional listing video, navigation alternatives and deferred seller ratings. Current direction: one-month visibility (30 days proposed), up to five photos only, Listings in the former Messages tab, three-column Profile, upload arch and seller interaction ratings on listing reels. The later screenshot revision keeps every current profile function in submenus, removes profile seller ratings, and confirms bottom order Feed / Trending / Upload / Profile / Listings.
Status: planning only; no feature implementation, policy publication or legal approval.
This consolidates the conversation, owner decisions, draft communication and technical recommendations. Recommendations are not approved requirements unless identified as owner decisions.

## Confirmed owner decisions

- Introduce a local marketplace for listings and leads, including local job vacancies.
- Every marketplace post costs **10 stars**, including jobs. This supersedes the original IADME-040 five-star job price and the earlier suggested category-specific prices.
- Listings are publicly available for **one month** and then expire (owner update on 2026-10-02; 30 days proposed for implementation). Automatic storage deletion is requested for consideration; the retention design below is a recommendation, not a settled deletion schedule.
- Reuse the existing feed view with filters, age pill, reaction rail, advertiser name/avatar and an additional action to ping the advertiser. Marketplace browsing has **no scroll score**.
- An advertiser must have **both phone and email verified** before requesting publication.
- Clearly document what may and may not be advertised.
- iAdMe provides advertising space and leads/contact facilities only. The owner does not want to assure listing quality or advertiser claims, arrange transactions, handle users' transaction taxes, or receive a percentage commission.
- Preserve this discussion for implementation when ready. No implementation or release is authorized by this record.

## Product scope and earlier discussion

Launch categories proposed: local jobs, ordinary physical items and local services. Final categories require approval. Current media decision: up to five photos only. The original optional short-video proposal was superseded on 2026-10-02.

Navigation decision, 2026-10-02: Listings occupies the former Messages bottom-nav position. Messages moves into Profile. Profile uses three icons per row, beginning Inbox / Messages / Wallet, with submenus and unread badges. The bottom + opens an arch of Video / Photo / Listing; normal video/photo composer UI stays the same.

Use visual listing tiles/feed, with structured information and locality/category filters. Show title, category, price or salary, locality, availability and age. Search, stable pagination and expiry filtering are required. Reuse feed components without inheriting reel reward behavior.

The owner wants the reaction rail. The earlier recommendation was to defer public comments to reduce spam and disclosure; whether the reused rail includes comments remains open. Define permitted reactions, comment moderation and whether any other engagement rewards apply. No scroll score means no marketplace scroll-star accrual, qualifying events or client/backend reward side effects.

Ping should open a listing-linked enquiry using the existing messaging system, rather than reveal contact details or send an unexplained notification. Proposed label: “Send enquiry” or “Message advertiser.” Inbox filter proposal: All / Personal / Market. Apply block checks, spam limits, duplicate-send protection and notification preferences. Existing enquiries should show an expired/unavailable listing label after expiry.

Show iAdMe ads between listings with a clear Sponsored label and controlled frequency. Ordinary posting fees do not promise promotion; paid placement does not imply endorsement. Dedicated promoted listings are a later proposal.

Defer checkout, escrow, delivery booking, commissions and transaction guarantees. Seller interaction ratings are now required; the 2 October record proposes collection after an eligible conversation without claiming a verified purchase. Star listing-fee refunds are separate from refunds for user-to-user deals.

## Listing fields and publication lifecycle

Use category templates and plain-text descriptions, not arbitrary HTML.

- Jobs: employer, role, locality, work arrangement, job type, salary/range or disclosure choice, hours, requirements and application deadline. No public CV or identity-document uploads.
- Items: title, category, price, condition, locality, description and collection/delivery information.
- Services: service type, service area, price basis, availability and description.

Proposed states: Draft, Pending review, Published, Rejected, Closed (Sold/Filled), Expired, Removed, Purged. Advertiser controls: preview, edit, close and explicit repost. The one-month window (proposed 30 days) should run from server-recorded publication/approval time, not draft submission. Store timestamps in UTC and render locally. An earlier deadline or advertiser closure may end visibility sooner. Edits must not silently reset expiry.

Show 10 stars and balance before final submission. Server controls price, balance and verification eligibility. Reserve stars while review is pending if the ledger supports reservations; debit once on publication and release reservations on rejection. Alternatively use an atomic debit-at-publication design. Insufficient balance must not publish. Use a durable idempotency key and atomic ledger/listing transition to prevent duplicate charges and concurrent overspending. Drafts and technical failures are not chargeable.

Pending decisions: active-post limits, account-age restrictions, free-edit rules, cancellation/refund treatment, rejected/removed post treatment and reposting mechanics. Proposed repost: explicit fresh confirmation, another 10 stars and a fresh one-month window (proposed 30 days); no automatic debit or renewal.

## Expiry and storage retention — recommendation

**Recommendation: automatic expiry plus staged deletion, rather than deleting every related record immediately on expiry.** Public expiry and retention are different obligations.

1. At the end of the one-month window (proposed publication + 30 days), hide the listing from feed, search, profile listings, recommendations, share previews and new enquiries. Enforce this in authoritative reads even if the expiry worker is delayed. An old link should return an unavailable state without exposing retained content.
2. Proposed ordinary-media purge: within 7 days after expiry (by publication + 37 days if the 30-day window is accepted). Remove original media, transformed images, thumbnails, derived descriptions/embeddings and orphaned uploads. This grace period is proposed for operational recovery; approve it before implementation.
3. Remove expired listing descriptions and nonessential listing-specific personal data on the purge schedule. Keep a minimal expired-listing stub only where needed for existing enquiries or accounting: listing ID, advertiser ID, category, lifecycle timestamps and status; retained title/context needs a justified bounded retention period.
4. Existing user messages follow a separately disclosed messaging retention/deletion policy. Listing expiry must not wipe a user's entire conversation, account, verification information or shared media used elsewhere. Media access must end at expiry even if physical deletion occurs later.
5. Keep minimal star-ledger/payment-related records for applicable accounting and audit requirements. Duration needs legal/accounting approval; do not delete balances or historical ledger entries with the listing.
6. Preserve only necessary evidence for an active report, lawful request or legal hold, with restricted access, an expiry/review date and a documented release-to-purge process. A report must not become indefinite retention of all listing data.
7. Purge database content, object storage and derivatives, search indexes, caches/CDN and queued jobs. Account for object versioning, replicas and backups. Backups may age out on their documented cycle; restore procedures must reapply deletion tombstones so deleted listings are not republished.
8. Use retryable, idempotent cleanup jobs, reference checks for shared objects, per-resource deletion status and monitoring for expired-but-visible listings, orphaned media and failed deletion. Avoid sensitive text in logs. Document local-client cache expiry and access revocation.

Why: immediate total deletion removes enquiry context, charge evidence and abuse evidence. Indefinite storage creates privacy and cost burdens. The final retention schedule must be purpose-specific and disclosed; one-month visibility does not itself justify any particular legal retention duration.

Draft user copy: “Listings are visible for one month. After expiry, they are removed from discovery. Listing media is scheduled for deletion under our retention policy. Limited records may be retained for existing enquiries, account records, abuse reports or legal requirements.” Replace this with exact approved durations before launch.

## Advertiser verification and privacy

Require both verified phone and verified email server-side before accepting a publication request and recheck eligibility at publication. Handle contact changes, verification revocation, disabled accounts and pending requests. Rate-limit verification attempts; expire OTPs, prevent replay and avoid account enumeration. Never store raw verification codes in logs.

Verification proves control of the contact methods only. Use “Phone and email verified,” not “Trusted seller,” “Verified employer” or “iAdMe approved.” Explain that claims, identity and quality are not guaranteed. Do not make phone/email public by default; enquiries use in-app messaging.

Use locality/approximate distance rather than public exact addresses or live location. Strip location metadata from uploaded media. Warn contextually about sharing phone numbers, identity documents, OTPs, banking credentials or advance payments. Limit staff access, document report review, provide account/content deletion controls and publish retention rules.

## Allowed and prohibited listings — proposed launch policy

Publish this in category selection, posting rules, Terms and Help. The lists below require jurisdiction-specific legal review and owner approval before becoming live policy; category eligibility must also be enforced server-side.

**Proposed allowed:** lawful ordinary used/new household items, furniture, books, clothing, electronics owned by the advertiser; lawful local services within approved categories; genuine local job vacancies from employers or authorized recruiters, with no application fee. Advertisers must have the right to advertise, use uploaded media and offer the item/service/job.

**Proposed prohibited:**

- Illegal, stolen or counterfeit goods; pirated software/media; goods the advertiser has no right to sell.
- Weapons, ammunition, explosives, dangerous chemicals and illicit drugs or controlled substances.
- Prescription medicines, medical devices requiring regulated authorization, tobacco/vapes and alcohol at launch.
- Sexual services, exploitation, trafficking, sexual content involving minors and abusive/discriminatory or threatening content.
- Scam jobs, candidate application/security/training deposits used to obtain employment, pyramid schemes, money-mule recruitment, misleading earnings claims and unlawful work.
- Loans, investments, securities, crypto schemes, gambling and unapproved financial services.
- Identity documents, bank/social/app accounts, personal datasets, credentials and privacy-invasive services.
- Live animals/wildlife, human body parts or biological material.
- Unapproved regulated/high-risk offerings or links designed to evade these rules.

**Exclude initially pending dedicated review:** real estate, vehicles, food, childcare, healthcare, legal/financial advice and other licensed professional services. Their permitted status must not be inferred from the general “services” category. A category allowlist reduces launch scope; “Other” must not bypass it.

Always prohibit impersonation, spam, duplicates, deceptive media, doxxing and advertising another person's private information without authorization. Explain rejection reasons and offer appeals. A listing fee does not buy an exception to policy.

## Safety and operational controls

Provide reporting for listings, advertisers and messages; blocking; first-poster/suspicious-content review; posting and enquiry rate limits; duplicate detection; abuse queues; takedown/suspension controls and an appeal trail. Assign a review owner and escalation process. Reviewing content for platform rules is not certification of quality.

Flag upfront job fees, requests for OTPs/banking credentials, unusual salaries, copied listings and pressure to move off-platform. No screening guarantees safety. Avoid transaction ratings until an actual transaction can be credibly established.

Link proposal: disable clickable arbitrary description links at launch. A dedicated reviewed business/employer website field may be considered later. If enabled, use parsed URL/scheme/host validation, reject obfuscation/shorteners, screen malicious destinations and show the destination domain before opening. Secure any server-side link previews against SSRF and redirect bypasses. No scanner is a guarantee.

## Platform role, fees and limits — owner intent and legal discussion

iAdMe provides listing publication and contact facilities. Users independently decide whether to transact or apply. The intended scope excludes quality assurance, factual certification of advertiser claims, payment collection for deals, commissions, delivery, escrow, user-transaction invoicing/taxes and adjudication/reimbursement of deals.

Important limit recorded in the conversation: disclaimers cannot guarantee immunity from complaints or claims, waive mandatory duties, or establish a legal classification merely by calling the service “lead-only.” No commission does not eliminate the platform's own tax/accounting obligations on listing-fee revenue. Applicable intermediary, consumer, employment, privacy, tax and app-store rules must be reviewed for the actual product and launch jurisdictions. India was assumed in the earlier discussion; jurisdiction and legal sign-off remain pending.

### Draft public communication

“Discover local listings. Connect directly. iAdMe provides advertising space and messaging so users can find and contact advertisers. Listings are supplied by advertisers. iAdMe does not verify or guarantee their accuracy, quality, legality, availability, or the outcome of any deal or job application. Users arrange any transaction directly with each other, including payment, delivery, refunds, invoices, and applicable taxes. iAdMe does not collect transaction payments or take a commission. Any stars charged are for publishing a listing and do not guarantee enquiries, sales, or hiring.”

This is draft product language, not a legally approved limitation-of-liability clause.

### Placement and copy

- Landing: “Local listings. Contact advertisers directly. iAdMe does not guarantee listings or deals.”
- Listing near enquiry: “Advertiser-provided information. Check details directly before making a commitment.”
- First enquiry: “You are contacting an independent advertiser. Any agreement or payment is arranged directly between you and them. iAdMe does not provide transaction protection.”
- Posting confirmation: “Publishing costs 10 stars. Your listing is visible for one month. This fee covers the listing only and does not guarantee leads or results.”
- Paid placement: “Sponsored — paid placement, not an iAdMe endorsement.”
- First-poster unchecked acceptance: “I am responsible for this listing and its accuracy, permissions, and legal compliance. I will handle any resulting agreement, payment, delivery, refund, invoice, and applicable tax obligations directly. I understand that iAdMe provides listing and messaging services and does not guarantee results.” Record version and acceptance timestamp.
- Report screen: “Report suspected fraud, prohibited content, harassment, or misuse. iAdMe may review the report and take action on the listing or account. iAdMe does not adjudicate transaction disputes or provide reimbursement for transactions between users. This does not affect rights or remedies available under applicable law.”

Update Help, FAQ, Terms, privacy/retention policy, prohibited-listings policy, posting-fee/refund rules, safe buying/selling/job-search guidance and reporting/appeals instructions. Provide a platform grievance/support path; do not promise that reporting will recover money.

## Implementation and release acceptance checklist

- [ ] Approve categories, prohibited-listings rules and comments/reaction behavior. Implement the decided navigation, up-to-five-photo limit, profile grid, upload arch and seller-rating collection after design review.
- [ ] Obtain jurisdiction-specific legal review of platform role, terms, grievance obligations, privacy/retention, job advertising and star/store billing; confirm own-fee tax/accounting treatment.
- [ ] Implement phone AND email verification gate on request and publication.
- [ ] Implement atomic, idempotent 10-star publication charge and clear rejection/refund handling.
- [ ] Reuse feed filters, age pill, reaction rail and name/avatar; exclude marketplace from scroll-score/reward logic on client and server.
- [ ] Implement listing-linked enquiry, blocks, rate limits, inbox context and notification controls.
- [ ] Enforce the approved one-month visibility window in every read/access path and implement expiry worker.
- [ ] Approve exact purpose-specific retention windows; implement media/data purge, legal holds, cache/index cleanup, backup tombstones and monitoring.
- [ ] Implement moderation/reporting/appeals and assign operations ownership.
- [ ] Publish approved copy consistently across screens and policy pages; record acceptance versions.
- [ ] Validate boundary expiry, delayed workers, timezone rendering, edits/reposts, inaccessible old links/media, cleanup retries, shared-object safety and restored backups.
- [ ] Validate double submits, concurrent devices, insufficient stars, verification changes, blocked enquiries, message history after expiry and zero marketplace scroll-score accrual.
- [ ] Measure publication/enquiry rates, time to enquiry, reports, moderation delays, charge failures, storage cleanup failures and expired listing exposure. Do not infer completed sales from enquiries.

## Sources referenced during discussion

These are reference material, not legal approval; verify current rules at implementation.

- Google Play UGC policy: https://support.google.com/googleplay/android-developer/answer/9876937?hl=en — terms acceptance, reporting/blocking and moderation requirements.
- OWASP redirects guidance: https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html — validate destinations and use allowlist-based controls.
- India Department of Consumer Affairs rules: https://consumeraffairs.gov.in/pages/consumer-protection-acts — assess platform duties and current amendments with counsel.
- Information Technology Act: https://www.indiacode.nic.in/indiacode/handle/123456789/18594?view_type=browse — assess applicable intermediary conditions rather than claiming automatic immunity.


## Implementation follow-up — 2 October 2026

The owner authorised implementation after accepting the screenshot-based design.
See [the implementation and verification record](LOCAL_MARKET_UI_2026-10-02.md#implementation-record--2-october-2026).
The source implementation uses 10 Stars charged once on approval, 30 days of
visibility, five photos, verified phone/email, category-specific details,
moderation and listing-linked enquiries. Media/public-content purge is scheduled
seven days after closure, with explicit evidence holds; separate chat, review and
ledger retention remains. This supersedes the original seven-day visibility
proposal. The earlier legal/store/operations release checklist remains a release
checklist, not a claim of legal approval or immunity.

To respect the owner's concurrent ad-video work, heavy builds and runtime
rollouts are deferred. No current users, wallet balances, live listings or
production infrastructure were modified by implementation verification.

## Dev-only activation — 2 October 2026

The owner authorized activation once the laptop was free, with an explicit prohibition on staging or production work. Listings are now active in dev with private storage, local migrations, API/worker, local admin/policy pages and native mobile verification. This supersedes the earlier deferred dev activation status; public-launch legal, retention and store decisions in this record remain separate. [Dev acceptance evidence](../test-results/LISTINGS_DEV_2026-10-02.md).
