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

### Known Issues
- **Attachment persistence on Railway**: Attachments stored on ephemeral filesystem are lost on deploy. MinIO/S3 integration in progress.
- **Existing attachments broken on Railway**: Previously uploaded files (now deleted from filesystem) show "File missing on disk" — requires re-upload after MinIO is set up.
- **Lookup table data drift**: Database on Railway may have different lookup values than code expects (e.g., "Subcontractors" vs "Subcontractor (other)"). Case-insensitive matching mitigates this.

---

## Upcoming

### 🔧 MinIO / S3 Attachment Storage
- Integrate Railway MinIO add-on for persistent attachment storage
- Backend auto-detects MinIO via env vars; falls back to local filesystem
- Zero-downtime migration path: new uploads go to MinIO, old files remain accessible from disk

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
