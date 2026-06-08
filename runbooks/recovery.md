# iAdMe Disaster Recovery Runbook

## Purpose

This document describes recovery procedures for restoring iAdMe production services after data loss, infrastructure failure, deployment failure, or catastrophic outage.

---

# Recovery Objectives

## Recovery Priority Order

1. PostgreSQL Database
2. Redis
3. API Service
4. Worker Service
5. Admin Console Verification
6. Mobile Application Validation

---

# Database Recovery

## When To Use

- Database corruption
- Accidental data deletion
- Failed migration
- Infrastructure failure

## Restore Process

Identify available snapshot:

bash ls -lrt db-snapshots/ 

Restore database:

bash psql -U postgres iadme < snapshot.sql 

Or restore from compressed backup:

bash gunzip snapshot.sql.gz psql -U postgres iadme < snapshot.sql 

## Verification

Verify critical tables:

sql SELECT COUNT(*) FROM users; SELECT COUNT(*) FROM videos; SELECT COUNT(*) FROM comments; SELECT COUNT(*) FROM wallet_transactions; 

Verify latest records:

sql SELECT created_at FROM users ORDER BY created_at DESC LIMIT 10; 

---

# Redis Recovery

## When To Use

- Redis corruption
- Queue failures
- BullMQ failures

## Recovery

Restart Redis:

bash docker compose -f infra/docker/docker-compose.yml restart redis 

Restart worker:

bash docker compose -f infra/docker/docker-compose.yml restart worker 

## Verification

Verify queue metrics from Admin Console.

Confirm uploads are processing successfully.

---

# API Recovery

## Recovery

Restart API:

bash docker compose -f infra/docker/docker-compose.yml restart api 

## Verification

Health endpoint:

bash curl https://api.iadme.app/health 

Verify:

- Authentication
- Feed loading
- Comments
- Upload API

---

# Worker Recovery

## Recovery

Restart worker:

bash docker compose -f infra/docker/docker-compose.yml restart worker 

## Verification

Check:

- Queue depth decreasing
- New uploads processing
- HLS generation successful

Review logs:

bash docker logs iadme-worker --tail 200 

---

# Full Environment Recovery

## Recovery Sequence

Step 1

Restore PostgreSQL

Step 2

Start Redis

Step 3

Start API

Step 4

Start Worker

Step 5

Validate queues

Step 6

Validate application

---

# Production Validation Checklist

## Authentication

- [ ] Login works
- [ ] Refresh token works
- [ ] Email verification works

## Feed

- [ ] Feed loads
- [ ] Trending loads
- [ ] Video playback works

## Upload

- [ ] Upload succeeds
- [ ] Video processes
- [ ] Thumbnail generated
- [ ] HLS generated

## Engagement

- [ ] Like
- [ ] Dislike
- [ ] Comment
- [ ] Reply
- [ ] Share

## Wallet

- [ ] Wallet balance loads
- [ ] Coupon redemption works
- [ ] Unlock works

## Admin

- [ ] Dashboard loads
- [ ] Reports load
- [ ] Users load
- [ ] Videos load
- [ ] Queue metrics load
- [ ] Worker metrics load

---

# Launch Sign-Off

Recovery is complete only when:

- Database healthy
- Redis healthy
- API healthy
- Worker healthy
- Feed healthy
- Upload healthy
- Wallet healthy
- Admin Console healthy
- Production validation checklist passed