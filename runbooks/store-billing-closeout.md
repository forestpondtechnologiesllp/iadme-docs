# Store billing close-out

Last reviewed: 4 September 2026

This runbook separates Google Play release compliance, Google merchant payout
setup, and Apple In-App Purchase setup. They are independent workstreams.

## Current repository state

- Android production release builds select Google Play Billing.
- The resolved Android release dependency is
  `com.android.billingclient:billing:8.0.0` through
  `in_app_purchase_android 0.5.2`.
- iOS always selects Apple In-App Purchase; Razorpay cannot be selected on iOS.
- The API verifies Play purchase tokens and App Store transaction IDs before
  crediting Stars, with idempotent fulfilment.
- Mobile version `1.0.3+29` is the next Android candidate.

## Google Play: unblock releases and complete E2E

1. Build a signed production candidate using Java 17 or 21 and all production
   build-time values:

   ```bash
   flutter build appbundle --release \
     --dart-define=APP_ENV=prod \
     --dart-define=APP_RELEASE=iadme_app@1.0.3 \
     --dart-define=SENTRY_DSN=<production-sentry-dsn> \
     --dart-define=ADMOB_ANDROID_NATIVE_AD_UNIT_ID=<production-native-ad-unit-id>
   ```

2. Upload the AAB to **Internal testing**. Do not reuse the older bundle shown
   by Policy status; Play evaluates the Billing Library inside each uploaded
   artifact.
3. Confirm all four one-time products are active and available in the tester's
   country:

   - `iadme_stars_starter`
   - `iadme_stars_popular`
   - `iadme_stars_pro`
   - `iadme_stars_mega`

4. Add the tester to the Internal testing list and to **Settings > License
   testing**. Install from the Play opt-in URL with the same Google account.
5. Confirm the production API has:

   - `GOOGLE_PLAY_PACKAGE_NAME=app.iadme.mobile`
   - `GOOGLE_PLAY_SERVICE_ACCOUNT_EMAIL`
   - `GOOGLE_PLAY_SERVICE_ACCOUNT_PRIVATE_KEY`

6. Test each pack at least once. For every purchase, verify: Play checkout,
   success return, one wallet credit, one purchase-history record, correct
   pack/product/order IDs, and successful repurchase after consumption.
7. Test cancel, pending payment, interrupted network/app restart, duplicate
   verification, refund, and repurchase. Confirm no duplicate wallet credit.
8. Promote the tested artifact to Production. After Play finishes processing,
   check Policy status again. The warning closes only after a compliant bundle
   reaches every actively published track that Play identifies.

If production must be released before the candidate is ready, use **Request
more time** on the policy issue. The extension is temporary; it is not a fix.

## Google merchant profile after BillDesk approval

BillDesk approval is merchant KYC for the India cross-border payment profile.
It does not provide an SDK key and does not configure Google Play Billing in
the app.

In **Play Console > Settings > Payments profile**:

1. Open the `Cross border` payments account and confirm there is no BillDesk,
   identity-verification, or payout hold banner.
2. Open **How you get paid** and confirm the bank account is present, verified,
   and uses the exact legal account-holder name. The bank account must be in
   India and capable of receiving the configured payout method.
3. In Google Payments Center, confirm the organisation name, registered
   address, primary contact, public merchant information, customer-support
   email, and statement descriptor.
4. Under India tax information, add or verify the business address, state and
   GSTIN. Reconcile this with finance/tax advisers; Google does not complete
   the merchant's GST filing.
5. Confirm the separate `India only` profile remains available for domestic
   transactions and that the cross-border profile is the one with completed
   BillDesk verification.
6. Check **Order management** and **Download reports > Financial** after the
   first licensed test/production order. Check the next payout statement and
   request FIRC through Google when required for foreign inward remittances.

## Enrolment in the 15% service-fee tier

This is recommended if the developer and all associated developer accounts are
eligible. It affects fees, not Billing Library compliance or BillDesk KYC.

1. Select **Manage account group** from the Payments profile page.
2. Create an account group with this developer account as the primary account.
3. Declare every associated developer account, including accounts controlled
   by the same organisation or controlling/controlled entities. Declare none
   only if that is factually correct.
4. The account owner accepts the 15% service-fee Terms and Conditions.
5. Return to the programme page and confirm enrolment is active.

The reduced tier is generally applied to the first USD 1 million of annual
earnings, subject to Google's current programme rules. Do not create or omit
associated accounts to influence eligibility.

## Google keys used by the API

There is no Google Play Billing secret in the Android app.

1. In a Google Cloud project, enable **Google Play Android Developer API**.
2. Create a dedicated service account and a JSON key.
3. In Play Console **Users and permissions**, invite the service-account email.
4. Grant the iAdMe app access plus **View financial data, orders, and
   cancellation survey responses** and **Manage orders and subscriptions**.
5. Store JSON `client_email` as `GOOGLE_PLAY_SERVICE_ACCOUNT_EMAIL` and
   `private_key` as `GOOGLE_PLAY_SERVICE_ACCOUNT_PRIVATE_KEY` in the production
   secret store. Never put the JSON or private key in Flutter or source control.
6. The Android upload/signing key is separate. The existing release keystore
   signs the AAB; Play App Signing signs APKs delivered to users.

## Apple In-App Purchase and IPA setup

Stars are digital credits, so iOS must use Apple In-App Purchase. Do not expose
Razorpay or an external checkout for Stars in the iOS build unless a specific
storefront entitlement and its requirements have been implemented.

1. The Account Holder accepts the active **Paid Apps Agreement** and completes
   banking and tax information in App Store Connect.
2. Under the app's **Monetization > In-App Purchases**, create and submit these
   products as **Consumable**:

   - `iadme_stars_starter`
   - `iadme_stars_popular_v2`
   - `iadme_stars_pro_v2`
   - `iadme_stars_mega_v2`

3. Add localisation, price, availability, tax category, and review screenshot
   for every product. The first IAPs must be submitted with an app version.
4. In **Users and Access > Integrations > In-App Purchase**, generate an
   In-App Purchase key and download the `.p8` once.
5. Store the following only in the production API secret store:

   - `APP_STORE_BUNDLE_ID=app.iadme.mobile`
   - `APP_STORE_ISSUER_ID` from the In-App Purchase integration page
   - `APP_STORE_KEY_ID` shown beside the generated key
   - `APP_STORE_PRIVATE_KEY` containing the downloaded `.p8` contents

6. The `.p8` key is for server verification; it is not embedded in the IPA.
   IPA signing instead uses the Apple Distribution certificate and App Store
   provisioning profile managed by Xcode/App Store Connect.
7. Use a Sandbox Apple Account on a development-signed build or TestFlight.
   Test all four packs, cancel, interrupted purchase, duplicate delivery,
   server-verification failure, and refund/revocation handling before release.

## Production follow-ups

- Add Google Real-time Developer Notifications or a scheduled Voided Purchases
  API reconciliation so refunded/charged-back Play purchases can reverse or
  flag spent Stars.
- Add App Store Server Notifications V2 so Apple refunds and revocations are
  handled after the original purchase.
- Keep the store receipt as the customer tax receipt for store purchases; do
  not issue a duplicate iAdMe GST invoice for the same store transaction.
