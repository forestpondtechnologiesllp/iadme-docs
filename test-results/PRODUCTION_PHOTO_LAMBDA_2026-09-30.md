# Production photo Lambda implementation — 30 September 2026

Implemented in the existing `iadme-backend` working tree, preserving the prior IADME-035 changes. No mobile/UI changes or live cloud mutations were made.

## Implementation

- Production-only opt-in `PHOTO_PROCESSOR=lambda` with a qualified regional function ARN; staging/dev continue local Sharp.
- A synchronous, one-photo Lambda request carries object references only. The shared image pipeline validates byte counts/MIME/decoded content, applies orientation, strips metadata and writes display/thumbnail WebP variants. The GET is conditional on the source HEAD's ETag.
- Production orchestration does not read/transform image bytes or fall back to local processing after Lambda failure.
- CloudFormation creates a 1,024 MB, Node 22 x86_64 function with a 60-second timeout and reserved concurrency capped at two, a published version/live alias, scoped execution role, invocation policy and 14-day log group. No provisioned concurrency, VPC, API Gateway, S3 event trigger, database/Redis access or MediaConvert.
- Existing wallet reservation, atomic publication and refund behavior remains in the backend. Lambda throttling is deferred through the durable outbox without consuming processing attempts. Invalid/function-error results cannot publish partial carousels. SDK Invoke retries are disabled and HTTP timeouts are enforced.
- A pinned Linux/glibc deployment ZIP and console/activation runbook are included under `iadme-backend/infra/lambda/photo-transform` and `iadme-backend/docs/photo-lambda-2026-09-30.md`.

## Evidence

| Check | Result |
| --- | --- |
| TypeScript typecheck/build | Passed |
| Backend full unit suite | 117 passed; 6 existing opt-in skips |
| Photo integration suite | 9 passed |
| CloudFormation `cfn-lint`, `ap-south-1` and corrected production `us-east-1` | Passed |
| Lambda Node 22 native x64 runtime, network disabled, 1 GB memory | Handler, both WebP variants, orientation/metadata removal, idempotent retry and bucket restriction passed |
| Whitespace/diff check | Passed |

Integration tests used an empty dedicated `iadme_photo_test_lambda_20260930` database and a separate Redis container on port 56379, real PostgreSQL transactions and Sharp, and mocked S3/Invoke transport. Tests verify deferred throttling, a five-photo publication with one charge, partial-output failures with exactly one refund, ownership, location/feed behavior and existing photo regressions. No user data was copied. Temporary integration resources were removed after verification. The existing dev API/worker/Postgres/Redis remained running.

Tested runtime image: `public.ecr.aws/lambda/nodejs:22`, digest `sha256:5ed6372a9d3a6fa660e0e9734ebf78baa9736e01666c1e426799e73e187ab4b2`.

Final artifact: `photo-transform-c7ef086d1b97d526.zip`, 16,298,474 bytes.

- SHA-256: `c7ef086d1b97d5268edab59deea339bee15ef758ad49ec01f05a5fbb6af7c02a`
- CloudFormation `CodeSha256`: `x+8IbRuX1SaO2rWd7qM5vuFe91itSewB8Fpfu2r3wCo=`

The ZIP is an ignored local build artifact; the build script and separate dependency lock reproduce the package contents. Rebuilding can change the archive hash, so always use the matching generated manifest.

## Deployment boundary

Region correction: the production bucket `iadme-media-prod` was verified via AWS GetBucketLocation (`null` means `us-east-1`), matching the saved production environment. The console guide and worker ARN example now use US East (N. Virginia). Mumbai is the local development region. The region-independent Lambda ZIP and CloudFormation template require no rebuild for this correction.

Initially delivered without a cloud deployment. The user subsequently supplied `arn:aws:lambda:us-east-1:081986192946:function:iadme-prod-photo-transform-v2:live`. Read-only AWS checks confirm that alias points to version 1, Active/Successful, Node 22/x86_64, 1,024 MB, 60-second timeout, reserved concurrency 2, and `PHOTO_BUCKET=iadme-media-prod`; the deployed code hash matches the final artifact above. No VPC is attached.

The user explicitly deferred the production backend build/deployment. No worker environment activation or backend rollout was performed. A standalone image transformation test and verification of the actual worker's IAM invocation permission are still pending; so are the later production upload acceptance tests. Existing photo-post bucket privacy/lifecycle requirements still apply. Do not resume production activation without a new user instruction.
