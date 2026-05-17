# Ta'awon SMS — Roadmap

## Legend
- ✅ Done
- 🔧 In progress
- 📅 Planned
- 💡 Proposed

---

## Milestone: v0.5 — Initial Application & Basic Auth ✅
*Date: Pre-April 2026*

- Single-page frontend with site access ID form
- Backend FastAPI with PostgreSQL
- Basic authentication (role-picker dropdown)
- Submit/view applications
- Admin panel for users, roles, groups
- Email notifications on status change
- Deployment guide for Railway

---

## Milestone: v0.8 — Real Authentication ✅
*Date: 3 May 2026*

- Session-based auth with opaque tokens
- bcrypt password hashing (replaces `temp_hash_123` placeholders)
- Two login tracks: staff (username) and external (email)
- Forced password change on first login
- Rate-limited login (5 attempts / 15 min cooldown)
- Auth-gated attachment downloads
- External submitters locked to their organization
- Sessions expire after 8 hours; auto-refreshed on each request
- Removed X-Acting-User-Id header; identity derived from session cookie

---

## Milestone: v0.9 — Dynamic Lookups & Subcontractor Handling ✅
*Date: 9-16 May 2026*

- Lookup tables admin panel (`Lookup_Tables` / `Lookup_Values`)
- Dynamic company and subcontractor dropdowns populated from DB
- Two-tier form: brief (Client/PMC/Main/LDC) vs full (Subcontractor)
- Subcontractor dropdown only appears when "Subcontractor (other)" selected
- Edit modal uses dropdowns from lookup tables
- Excel import/export for lookup values
- Attachment preview modal with auth-credentialed fetch
- Hardcoded fallback options for when DB tables aren't initialized

---

## Current State (16 May 2026)

### Working
- Full application lifecycle: submit → review → approve/reject
- Staff and external authentication
- Role/permission management
- Lookup table management (add/edit/delete/import/export)
- Two-tier form with company-driven variant selection
- Subcontractor name dropdown (merge of lookup table + organizations data)
- Attachment upload and auth-gated download/preview
- Email and WhatsApp notifications
- Workflow history and audit trail
- Deployment on Railway (two repos, auto-deploy)
- Bulk backlog import from Excel (admin panel)

### Known Issues
- **Lookup table data drift**: Database on Railway may have different lookup values than code expects (e.g., "Subcontractors" vs "Subcontractor (other)"). Case-insensitive matching mitigates this.

---

## Upcoming

### 📅 Email Delivery Reliability
- Current SMTP (Gmail) works but can be flagged as spam
- Evaluate SendGrid / Mailgun / Railway's built-in email service

### 📅 WhatsApp Integration
- WhatsApp Business API integration for status notifications
- Requires Meta developer account and business verification

### 💡 Feature Proposals

#### Multi-file attachments
- Allow multiple files per attachment type (e.g., multiple insurance documents)

#### Bulk Excel submission
- Allow admin to submit applications via Excel upload (including attachments as embedded or alongside)

#### Advanced search / filtering
- Search applications by name, company, ID number, date range
- Filter by workflow state, submitter group

#### PDF generation
- Auto-generate printable site access ID card from approved application

#### Rate-limit tuning
- Make login throttle configurable via admin settings

#### Two-factor authentication
- Optional TOTP or SMS-based 2FA for staff accounts

#### Application dashboard
- Summary statistics (applications by status, approval rate, average processing time)
- Chart/graph views

#### Activity log
- Searchable audit log of all user actions (not just workflow state changes)

---

## Operations Guide

### Database initialization (`init-db`)

**When to run**: Only on a completely fresh database (first deployment or after provisioning a new PostgreSQL add-on).

**What it does** (all safe to re-run):
- Creates all tables (`CREATE TABLE IF NOT EXISTS` — skips existing)
- Seeds default data (roles, admin users, lookup tables — `ON CONFLICT DO NOTHING` — skips existing)
- Replaces `temp_hash_123` placeholders with real bcrypt hashes (only affects rows still holding placeholder)
- Runs schema migrations (e.g., adding new columns via `ALTER TABLE ADD COLUMN IF NOT EXISTS`)

**Caution**: On a populated DB the endpoint is **auth-protected** — you must be logged in as an admin with `manage_system_settings` permission. On a fresh DB (no users table), unauth calls are allowed.

**Command**:
```bash
curl -X POST https://securitysms-production.up.railway.app/api/v1/admin/init-db \
  -H "Content-Type: application/json" -d "{}"
```

### Backup & Restore

Railway PostgreSQL add-on has **daily automatic backups** (see Dashboard → PostgreSQL → Backups).

**Manual backup** (run from any machine with `pg_dump`):
```bash
pg_dump --no-owner --no-privileges \
  "postgresql://postgres:PASSWORD@mainline.proxy.rlwy.net:PORT/railway" \
  > sms_backup_$(date +%F).sql
```

**Restore** (loading into an existing or new database):
```bash
# 1. Load the backup
psql "postgresql://postgres:PASSWORD@new-host:PORT/railway" < sms_backup_2026-05-17.sql

# 2. Ensure schema is up-to-date (safe — won't overwrite existing data)
curl -X POST https://securitysms-production.up.railway.app/api/v1/admin/init-db \
  -H "Content-Type: application/json" -d "{}"
```

**Important**: Always call `init-db` after a restore to catch any schema columns added since the backup was made.

### MinIO / S3 Attachment Storage

**Setup on Railway**:
1. Provision a MinIO add-on in the same Railway project as the backend
2. The following environment variables are automatically injected (or set manually):

   | Variable | Example | Purpose |
   |---|---|---|
   | `MINIO_ENDPOINT` | `minio.railway.internal:9000` | MinIO server address |
   | `MINIO_ACCESS_KEY` | `minio_access_key` | S3 access key |
   | `MINIO_SECRET_KEY` | `minio_secret_key` | S3 secret key |
   | `MINIO_BUCKET` | `attachments` | Bucket name (default: `attachments`) |

   The backend also accepts legacy names `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD` and standard AWS names `AWS_ENDPOINT_URL` / `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY`.

3. After setting variables, **restart the backend service** on Railway.

**How it works**:
- Backend auto-detects MinIO at startup when `MINIO_ENDPOINT` is set
- Creates the bucket if it doesn't exist
- All new uploads go to MinIO; old files on local filesystem remain accessible
- Attachment preview/download is proxied through the backend (never served via public URL)
- If MinIO is unavailable, falls back to local filesystem

---

## Deployment Targets

| Environment | Backend URL | Frontend URL |
|---|---|---|
| **Production** | `https://securitysms-production.up.railway.app` | `https://taawon-sms-frontend-production.up.railway.app` |
| **Local dev** | `http://localhost:8000` | `http://localhost:8000` (backend serves frontend) |

Backend and frontend are separate repos but the backend FastAPI also serves `index.html` at `/` for local development convenience.

---

## Git Repositories

| Component | Owner | Repo |
|---|---|---|
| Backend | abkader60-sketch | `taawon-sms-OC-backend` |
| Frontend | abkader60-sketch | `taawon-sms-OC-frontend-` |
