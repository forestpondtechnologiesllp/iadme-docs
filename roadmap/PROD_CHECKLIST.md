# Production Readiness Checklist

## Security
- [x] JWT auth
- [x] Refresh rotation
- [x] Auth sessions
- [x] Session revocation
- [x] Password invalidation
- [x] Rate limiting
- [ ] Distributed rate limiting
- [ ] WAF
- [ ] Secrets manager

## Infra
- [x] Dockerized API
- [x] Dockerized Postgres
- [x] Dockerized Redis
- [x] Health checks
- [x] Graceful shutdown
- [ ] Managed Postgres
- [ ] Managed Redis
- [ ] Load balancer
- [ ] Autoscaling

## Observability
- [x] Sentry
- [x] Structured logging
- [x] Request tracing
- [ ] Metrics dashboards
- [ ] Alerting

## CI/CD
- [ ] GitHub Actions
- [ ] Docker image build
- [ ] Staging deploy
- [ ] Production deploy
