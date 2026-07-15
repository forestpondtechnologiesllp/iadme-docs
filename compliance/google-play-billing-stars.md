# Google Play Billing for Stars

Android builds distributed through Google Play must sell Stars through Google
Play Billing. Razorpay remains available for dev, staging, admin, web, and
non-Play distribution paths.

Production Android release builds use Google Play Billing when built with
`--dart-define=APP_ENV=prod`. Dev and staging builds stay on Razorpay by
default. Use `--dart-define=BILLING_PROVIDER=google_play` only for an explicit
local Play Billing test, and `--dart-define=BILLING_PROVIDER=razorpay` to force
Razorpay.

## Play Console products

Create these one-time in-app products as consumable products in Google Play
Console. The backend maps each product ID to the existing active Star pack.

| Star pack code | Google Play product ID | Stars |
| --- | --- | --- |
| STARTER | `iadme_stars_starter` | 100 |
| POPULAR | `iadme_stars_popular` | 220 |
| PRO | `iadme_stars_pro` | 600 |
| MEGA | `iadme_stars_mega` | 1300 |

Keep the Play Console prices aligned with `star_purchase_packs.price_in_paise`.
The backend remains the source of truth for Star fulfilment.

## Backend configuration

Set these environment variables for the API environment used by Play builds:

- `GOOGLE_PLAY_PACKAGE_NAME=app.iadme.mobile`
- `GOOGLE_PLAY_SERVICE_ACCOUNT_EMAIL=<service-account-email>`
- `GOOGLE_PLAY_SERVICE_ACCOUNT_PRIVATE_KEY=<service-account-private-key>`

The service account must have Android Publisher API access to the iAdMe Play
Console app. Store the private key with escaped newlines when using `.env`
files.

## Fulfilment and invoices

The Flutter app sends the Play purchase token to
`POST /payments/google-play/verify`. The backend verifies the token with the
Google Play Developer API, creates a `google_play` payment order, and credits
Stars through the existing idempotent wallet fulfilment transaction.

Google Play is the checkout and tax receipt surface for Play purchases, so
iAdMe records the payment for history and wallet audit but does not issue an
iAdMe GST invoice for `google_play` orders. Razorpay purchases continue to use
the existing invoice and credit-note flow.
