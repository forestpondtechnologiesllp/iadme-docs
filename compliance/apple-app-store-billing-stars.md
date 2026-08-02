# App Store billing for Stars

iOS builds distributed through the App Store sell Stars through Apple
In-App Purchase. Razorpay remains available only for development, staging,
web, admin, and non-App-Store distribution paths.

Production iOS release builds automatically use Apple In-App Purchase. For a
local sandbox test, explicitly build with
`--dart-define=BILLING_PROVIDER=app_store`. Set
`--dart-define=BILLING_PROVIDER=razorpay` to force Razorpay in a non-store
test build.

## App Store Connect consumable products

| Star pack code | App Store product ID | Stars |
| --- | --- | ---: |
| STARTER | `iadme_stars_starter` | 100 |
| POPULAR | `iadme_stars_popular_v2` | 220 |
| PRO | `iadme_stars_pro_v2` | 600 |
| MEGA | `iadme_stars_mega_v2` | 1300 |

The App Store price displayed in StoreKit is authoritative for the customer.
The API is authoritative for the number of Stars credited. Keep the India
prices aligned with `star_purchase_packs.price_in_paise`.

## Backend configuration

Configure the API environment serving iOS builds with:

```env
APP_STORE_BUNDLE_ID=app.iadme.mobile
APP_STORE_ISSUER_ID=<app-store-connect-issuer-id>
APP_STORE_KEY_ID=<in-app-purchase-key-id>
APP_STORE_PRIVATE_KEY=<contents-of-the-p8-file>
```

`APP_STORE_KEY_ID` and `APP_STORE_PRIVATE_KEY` must belong to an **In-App
Purchase key** in App Store Connect, not a Sign in with Apple key. Store the
private key with escaped newlines when using an `.env` file. Do not commit it.

## Fulfilment and sandbox testing

The app sends only the StoreKit transaction ID to
`POST /payments/app-store/verify`. The API retrieves the transaction directly
from Apple, checks its bundle ID, product ID, revocation state, and StoreKit
`appAccountToken`, then credits Stars through the existing idempotent wallet
transaction.

For a sandbox purchase, enable Developer Mode, install a development-signed
or TestFlight build, use a Sandbox Apple Account, and purchase a consumable.
The API checks Apple's production endpoint first and then its sandbox endpoint,
so the same server deployment supports both environments.

Apple is the merchant of record for App Store transactions. iAdMe records the
order for wallet audit and payment history, but does not issue an iAdMe GST
invoice for `app_store` orders.
