# Phone-registered profile hotfix — 12 September 2026

## Problem and fix

A phone-registered iPhone user repeatedly saw “Could not load profile” and Retry. The corresponding production `/me` requests returned HTTP 200 in approximately 3–8 ms. The account legitimately had no email address, so the response contained `user.email: null`. The released Flutter `ProfileModel.fromMeJson` required a string and threw a type error before displaying the profile. This also affects the parser shipped in build 36.

The backend now returns an empty string for a missing email in `/me`. Database values remain unchanged. Existing email, Google and Apple relay addresses are preserved. No migration is required, and existing mobile installations can use the fix immediately after backend deployment.

The mobile parser also accepts null or omitted email, with regression tests. That defensive change is committed for the next mobile build; it is not in the existing 1.0.4+36 AAB/IPA. Those artifacts do not need replacement to receive the backend fix.

## Source and image

| Component | Revision |
| --- | --- |
| Backend | `440e51435adf4b7f57db7b1c79e05ee497384df6` |
| Mobile | `c0d7e1f4295e9dc5a58b5b509b2457ac14d8a5bc` |

Both source commits are pushed to their respective `main` branches.

- Image: `ghcr.io/forestpondtechnologiesllp/iadme-api:hotfix-2026.09.12-profile-440e514`.
- Digest: `sha256:dce10852eb341ef85e5e1d7f0be9c341298d800f18b86d83ba4290f1bcef8c5d`.
- Platform: `linux/amd64`, with the backend revision recorded in OCI labels.
- Only `/me` response normalization, its OpenAPI schema and regression tests changed in the backend.

## Regression verification

- Before the fix, the new backend phone-only test failed because email was null; email/password, Google and Apple cases passed.
- Before the mobile fix, the null-email and omitted-email cases reproduced the type error.
- After the fix: **14 backend tests passed** (5 profile/controller cases plus 9 existing registration/authentication cases). The TypeScript build passed.
- The same **14 tests passed against the exact compiled Docker image** with networking disabled.
- **6 Flutter tests passed**: 4 profile email cases and 2 existing local profile-progress cases. Targeted static analysis found no issues; formatting passed.
- The original parser extracted from mobile commit `24694b1` reproduced the null-email failure and successfully parsed the compatibility response with an empty email.

## Deployment verification

The same immutable image is deployed to API and worker in both environments. Runtime configuration was compared with the previous containers and is identical except for `APP_RELEASE`. JWT secrets and the 90-day session duration are preserved. No database changes or migrations were performed, and Redis was not restarted.

| Environment | API/worker started (UTC) | Verification passed (UTC) | Start time (IST) |
| --- | --- | --- | --- |
| Staging | 15:47:40 | 15:47:47 | 21:17:40 |
| Production | 15:48:31 | 15:48:37 | 21:18:31 |

Both APIs are healthy, their readiness checks can reach the database, the workers produced fresh registration-notification heartbeats, and the containers have zero restarts. Twelve HTTP checks per environment verified health/readiness and expected authentication/validation responses. Public HTTPS readiness returned 200 for both environments.

The post-deployment log check at 15:49:24 UTC found no API or worker error/fatal entries in either environment. This was a short post-deployment observation, not a substitute for physical-device acceptance.

Staging has no existing phone-only account with a null email. That case is covered by the compiled-image controller tests; the deployed staging service is checked against an existing email account. Production verification checks the reported phone-only account and an existing email account using database connections forced into read-only mode.

The production phone-only response has an empty email string, while a subsequent database read confirms the stored email is still null. An existing email account retains its address. A copy of the deployed phone-profile response with identifying fields redacted was successfully parsed by the unmodified build 36 model.

Before-deployment configuration and image references, plus verification reports, are stored under each environment's `backups/releases/hotfix-2026.09.12-profile-440e514/` directory on the deployment host. The rollback target is `release-2026.09.12-d2190d3` at digest `sha256:ec3e648379e953b5255d40c4a27410cc158e6500fd22788c298839e25c191113`. The rollout script would restore the previous API/worker configuration if checks failed; no rollback was needed. The floating `latest` image tag was not modified.

## Mobile acceptance

Reopen Profile or tap Retry on the affected iPhone. No logout, registration, reinstall or new mobile upload is needed. Physical confirmation on that device remains with the owner/tester.

The existing build 36 artifact hashes were rechecked and match the release report:

- AAB: `8def9f5216edb468bd02e6260cccecf2c18d2b10203aebcce27f24a4043713f1`.
- IPA: `185e548de42c805df9b1279f18f1f2191653618ea353c76905cb04e44e9b5977`.

The deferred ad-preload work (IADME-009) and upload-label change (IADME-010) are outside this hotfix.
