# Architecture Decisions Log

## 2026-05-14
- Messaging remains manual-refresh MVP.
- Backend contracts remain websocket-ready.
- Wallet uses immutable ledger model.
- Ads integrate as feed entities.
- Video delivery standardized on HLS.
- Moderation prioritized before realtime scale.

## 2026-05-17 — Infra Hardening Sprint Closed

- Standardized local infra around Docker Compose
- Standardized local physical-device networking around `.local`
- Redis adopted as shared infra layer
- BullMQ adopted as async orchestration foundation
- Health/readiness probes required before staging
- CI/CD GitHub Actions foundation introduced
- Environment separation strategy established
