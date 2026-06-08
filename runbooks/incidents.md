

# iAdMe Incident Runbook

## Purpose

This document provides operational procedures for handling production incidents in iAdMe.

---

# Severity Levels

## SEV-1 Critical

Examples:

- Production API unavailable
- Database unavailable
- Authentication failure for all users
- Upload pipeline completely down
- Data loss risk

Response Target:

- Immediate investigation
- Restore service as quickly as possible

---

## SEV-2 Major

Examples:

- Worker processing stopped
- Feed degradation
- Notifications failing
- Coupon redemption failures
- Wallet issues

Response Target:

- Investigate within 1 hour

---

## SEV-3 Minor

Examples:

- UI issues
- Analytics inconsistencies
- Non-critical admin console problems

Response Target:

- Schedule fix in next release

---

# API Down

## Symptoms

- Health endpoint failing
- Mobile application cannot load feed
- Admin console requests failing

## Investigation

Check containers:

```bash
docker ps
```

Check API logs:

```bash
docker logs iadme-api --tail 200
```

Check health endpoint:

```bash
curl https://api.iadme.app/health
```

## Recovery

Restart API:

```bash
docker compose -f infra/docker/docker-compose.yml restart api
```

Verify:

- Health endpoint returns OK
- Feed loads
- Admin dashboard loads

---

# Worker Down

## Symptoms

- Uploaded videos never complete processing
- Queue depth increasing
- Operations Health shows worker failures

## Investigation

Check worker logs:

```bash
docker logs iadme-worker --tail 200
```

Check queue metrics from Admin Console.

## Recovery

Restart worker:

```bash
docker compose -f infra/docker/docker-compose.yml restart worker
```

Verify:

- Worker metrics healthy
- New uploads process successfully

---

# Redis Down

## Symptoms

- Queue failures
- Worker failures
- BullMQ errors

## Investigation

```bash
docker logs iadme-redis --tail 200
```

## Recovery

```bash
docker compose -f infra/docker/docker-compose.yml restart redis
```

Then restart worker:

```bash
docker compose -f infra/docker/docker-compose.yml restart worker
```

Verify queue metrics.

---

# Database Down

## Symptoms

- API returns database errors
- Authentication failures
- Feed unavailable

## Investigation

```bash
docker logs iadme-postgres --tail 200
```

Verify connectivity.

## Recovery

Restart database service.

If database corruption is suspected, follow recovery.md.

---

# Bad Release

## Symptoms

- New deployment causes failures
- Elevated error rates
- Critical functionality broken

## Recovery

Rollback to previous known-good image tag.

Redeploy:

```bash
docker compose -f infra/docker/docker-compose.yml up -d
```

Verify:

- Feed
- Uploads
- Authentication
- Admin Console

---

# Security Incident

Examples:

- Credential exposure
- Unauthorized admin access
- JWT compromise

Actions:

1. Rotate affected secrets.
2. Revoke active sessions if required.
3. Review audit logs.
4. Verify AWS, DB and Redis credentials.
5. Document incident.

---

# Incident Closure Checklist

- Root cause identified
- Service restored
- Monitoring verified
- Audit logs reviewed
- Follow-up action documented
- Roadmap item created if needed