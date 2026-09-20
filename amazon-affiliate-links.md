# iAdMe manual Amazon affiliate link inventory

Saved and redirect-checked: 2026-09-17.

## Scope and current status

- Reference inventory only. No app implementation, publication, deployment, or live reel placement has been performed.
- User's plan: retain a shared pool of manually generated affiliate links for reuse across reels; use `iadmeapp-21` on both platforms initially.
- Preserve the supplied short URLs exactly. Do not silently rewrite tracking IDs or append parameters to short links.
- Verification here means the observed Amazon product redirect contains the listed tracking ID. It does not establish product availability, commission eligibility, or mobile-app approval.
- Amazon mobile-app approval remains unconfirmed in the conversation. Listing app URLs and generating links are not evidence of approval. This inventory does not authorize publishing the links in the app.

## Main tracking ID: iadmeapp-21

These 13 links have the selected tracking ID in their observed product destination URL.

| Source | Supplied affiliate URL | Observed product ASIN | Observed tracking ID |
| --- | --- | --- | --- |
| Earlier batch | https://amzn.to/4yIvXAF | B0BKJWBMSX | iadmeapp-21 |
| Earlier batch | https://amzn.to/3VyRrBA | B09G16VYXT | iadmeapp-21 |
| Earlier batch | https://amzn.to/3VeNlhV | B0FQFT397C | iadmeapp-21 |
| Earlier batch | https://amzn.to/4dGpPka | B0FC65P9VG | iadmeapp-21 |
| Earlier batch | https://amzn.to/4dkwdxs | 938918729X | iadmeapp-21 |
| Earlier batch | https://amzn.to/3USImDC | B0BRFWVMR3 | iadmeapp-21 |
| Latest batch | https://link.amazon/B02e9F4DF | B08BVJYLZ3 | iadmeapp-21 |
| Latest batch | https://link.amazon/B0bLbScvC | B0F5H242S1 | iadmeapp-21 |
| Latest batch | https://link.amazon/B06caJcEW | B0BQC4Y4TP | iadmeapp-21 |
| Latest batch | https://link.amazon/B0bFRkiE8 | B09V82JWBS | iadmeapp-21 |
| Latest batch | https://link.amazon/B0cKWiRAR | B0FN82NGM3 | iadmeapp-21 |
| Latest batch | https://link.amazon/B0ea3tuie | B0H627M878 | iadmeapp-21 |
| Additional link | https://link.amazon/B01U8NGPk | B0G38HMV1B | iadmeapp-21 |

The seven `link.amazon` URLs redirected through the corresponding `https://amzlinks.in/<same short code>` URL and then to an Amazon.in product URL containing `tag=iadmeapp-21`.

## Other owned tracking ID: iadmeios-21

Retained for completeness, but separate from the chosen single-ID pool. Regenerate through Amazon's link tools with `iadmeapp-21` if these products are to use the main ID.

| Source | Supplied affiliate URL | Observed product ASIN | Observed tracking ID |
| --- | --- | --- | --- |
| Earlier batch | https://amzn.to/3VfwOKE | B0CL3DZKRW | iadmeios-21 |
| Original sample | https://amzn.to/4z2xfa3 | B09G16VYXT | iadmeios-21 |

The original sample points to the same ASIN as `https://amzn.to/3VyRrBA`, but uses the iOS tracking ID. These are 15 affiliate URLs for 14 distinct observed ASINs; do not treat the duplicate as an additional product.

## Excluded ordinary sharing link

`https://amzn.in/d/01HXcNTR` redirected to Amazon.in product ASIN `B0CV3QR7VM` without an affiliate `tag` parameter. It is retained only as a reference, not as a verified affiliate link. Generate an Associates link with `iadmeapp-21` before including this product in the affiliate pool.

## Future activation notes

- Obtain confirmation of the app's permission to use Amazon affiliate links before publishing; see the [Amazon.in Associates Operating Agreement](https://affiliate-program.amazon.in/help/operating/agreement), sections 3 and 7.
- Recheck destinations and product suitability before launch. These redirects were checked without purchasing anything.
- This file supplies URLs and observed IDs only, not licensed product images, current prices, or API credentials.
- Link-generation guidance: [SiteStripe](https://affiliate-program.amazon.in/help/node/topic/GJMMT7G4C8K4Y3AY) and [Mobile GetLink](https://affiliate-program.amazon.in/help/node/topic/GH37MDS5PLQ9Z366).
