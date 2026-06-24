Version: 1.0
Last Updated: 2026-06-24
Owner: Forestpond Technologies LLP

EC2 Recovery Runbook

Purpose

Recover the iAdMe production and staging environments after complete EC2 loss.

Recovery Target (RTO):

< 30 minutes

⸻

Infrastructure Overview

Current Server

Hostname:

ip-172-31-34-125

OS:

Ubuntu 26.04 LTS

Kernel:

Linux 7.0.0-1006-aws

⸻

Architecture

Current Production Image

ghcr.io/forestpondtechnologiesllp/iadme-api:v0.9.3-r6-p0-p2-complete

Current Infrastructure Layout

EC2
├── PostgreSQL (host installed)
├── Redis (Docker)
├── Production API (Docker)
├── Production Worker (Docker)
├── Staging API (Docker)
├── Staging Worker (Docker)
├── Tailscale
└── NGINX

Production

EC2
├── PostgreSQL (host installed)
├── Redis (Docker)
├── API (Docker)
├── Worker (Docker)
├── Tailscale
└── NGINX

Database:

iadme_prod
Owner: iadme_app

Environment:

/opt/iadme/env/.env.prod

Compose:

/opt/iadme/docker-compose.prod.yml

⸻

Staging

Database:

iadme_staging
Owner: iadme_staging_app

Environment:

/opt/iadme-staging/env/.env.staging

Compose:

/opt/iadme-staging/docker-compose.staging.yml

⸻

DNS

Managed via:

Cloudflare

Domains:

iadme.app
api.iadme.app
staging-api.iadme.app

Cloudflare Nameservers:

izabella.ns.cloudflare.com
matteo.ns.cloudflare.com

⸻

Prerequisites

The following credentials must be available before recovery:

AWS account access
AWS-iadme-prod-key.pem
GitHub account access
GHCR credentials
Cloudflare access
Tailscale account access
Production environment files
Staging environment files

Do NOT store credentials in this document.

⸻

Step 1 – Provision New EC2

Launch:

Ubuntu 26.04 LTS

Attach:

Elastic IP (see P3.2)

Current Elastic IP

Elastic IP: 98.84.254.173
Allocation ID: eipalloc-0534fc5ad02c5385e

Recovery:
Associate this Elastic IP to replacement EC2 instance.
No DNS changes required.

Configure Security Group:

SSH (temporary)
HTTPS
HTTP

⸻

Step 2 – Install Core Packages

sudo apt update
sudo apt install -y \
docker.io \
docker-compose-v2 \
postgresql \
postgresql-client \
git \
curl

Verify:

docker --version
psql --version

⸻

Step 3 – Install Tailscale

curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

Verify:

tailscale status

Expected Nodes:

sais-macbook-air
ip-172-31-34-125

⸻

Step 4 – Restore Directory Structure

sudo mkdir -p /opt/iadme
sudo mkdir -p /opt/iadme-staging

Required folders:

/opt/iadme
/opt/iadme/env
/opt/iadme-staging
/opt/iadme-staging/env

⸻

Step 5 – Restore Environment Files

Production:

/opt/iadme/env/.env.prod

Staging:

/opt/iadme-staging/env/.env.staging

Validate:

grep DATABASE_URL /opt/iadme/env/.env.prod
grep DATABASE_URL /opt/iadme-staging/env/.env.staging

⸻

Step 6 – Restore PostgreSQL

Create databases:

iadme_prod
iadme_staging

Current ownership:

iadme_prod      -> iadme_app
iadme_staging   -> iadme_staging_app

Verify:

sudo -u postgres psql -l

⸻

Step 7 – Restore Database Backup

Backups location:

/opt/iadme-staging/backups

Latest backup example:

iadme_prod_YYYYMMDD_HHMMSS.sql.gz

Validate backup before restore:

`gunzip -t backup.sql.gz`

Restore:

gunzip -c backup.sql.gz | psql -U iadme_app iadme_prod

Verify:

sudo -u postgres psql -l

⸻

Step 7.1 – Re-enable Backup Automation

Current Schedule:

`0 2 * * * /opt/iadme/scripts/backup-db.sh >> /opt/iadme/logs/backup.log 2>&1`

Verify:

crontab -l

Expected:

0 2 * * * /opt/iadme/scripts/backup-db.sh >> /opt/iadme/logs/backup.log 2>&1

Confirm backups

Legacy feed refresh cron remains disabled:

#*/5 * * * * /opt/iadme/scripts/refresh-feed-pools.sh >> /opt/iadme/logs/feed-pool-refresh.log 2>&1

⸻

Step 8 – Restore Docker Authentication

docker login ghcr.io

Verify:

cat ~/.docker/config.json

⸻

Step 9 – Restore Production Containers

cd /opt/iadme
IADME_API_TAG=<tag> \
docker compose -f docker-compose.prod.yml up -d

Verify:

docker ps

⸻

Step 10 – Restore Staging Containers

cd /opt/iadme-staging
IADME_API_TAG=<tag> \
docker compose -f docker-compose.staging.yml up -d

Verify:

docker ps

⸻

Step 10.1 – Restore NGINX

Validate configuration:

`sudo nginx -t`

Restart:

`sudo systemctl restart nginx`

Verify:

`sudo systemctl status nginx --no-pager`

NGINX configuration locations:

`/opt/iadme/nginx`
`/opt/iadme-staging/nginx`

⸻

Step 11 – Validation Checklist

Production:

curl https://api.iadme.app/health

Staging:

curl https://staging-api.iadme.app/health

Verify:

API healthy
Worker healthy
Redis healthy
PostgreSQL reachable
Login succeeds
Feed loads
Upload succeeds
Premium unlock succeeds
Notifications delivered
Health endpoints healthy

⸻

Post-Recovery Smoke Test

Verify end-to-end functionality:

□ Login
□ Feed loads
□ Trending loads
□ Profile loads
□ Messages load
□ Upload video
□ Premium unlock
□ Notifications
□ Health endpoint
□ Staging environment reachable

⸻

Recovery Success Criteria

Production API healthy
Production Worker healthy
Staging API healthy
Staging Worker healthy
Database restored
Redis running
Login working
Feed working

Target:

Full recovery in under 30 minutes