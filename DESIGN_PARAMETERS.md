# Ta'awon SMS — Design Parameters

## Architecture

### Backend
- **Single-file FastAPI** (`main.py`) — keeps deployment simple; no package or module structure
- **PostgreSQL** — portable 17.9 binary distribution, no installer needed
- **Async I/O** — asyncpg + uvicorn for connection pooling under concurrent requests
- **Session-based auth** — opaque token stored in `Sessions` table; 8-hour expiry refreshed on each request

### Frontend
- **Single-page application** — all HTML, CSS, and JS in `index.html` (no build step, no framework)
- **API base URL** set via `<meta name="sms-api-base">` — one tag change switches between local and production
- **CSS-only responsive layout** — works on desktop and mobile browsers

### Deployment
- **Railway** (free tier) — auto-deploys from GitHub `main` branch
- **Two repos**: `taawon-sms-OC-backend` and `taawon-sms-OC-frontend-`
- **No persistent local state** — all configuration lives in Railway environment variables

---

## Security Model

### Authentication
| Principal type | Login ID | Credentials |
|---|---|---|
| Staff (`App_Users`) | username | bcrypt-hashed password |
| External (`External_Submitters`) | email | bcrypt-hashed password |

- Sessions expire after 8 hours of inactivity
- Password change forced on first login (`must_change_password` flag)
- Rate-limited login: 5 failed attempts in 15 minutes triggers a 15-minute cooldown per (username, IP)

### Authorization
- **Permission-based** for staff (roles aggregate permissions like `view_all_applications`, `submit_application`, `approve_security`)
- **Org-locked** for external submitters — they can only view/submit for their assigned organization
- **Group-locked** form variant: `group_1` (Client/PMC/Main/LDC) → brief form, `group_2` (Subcontractor) → full form

### Attachment Security
- All file access requires authenticated session
- Staff must have `view_all_applications` permission, OR be the original submitter
- External submitters can only access their own attachments
- Files stored in MinIO/S3 (production) or local filesystem (development)
- Never served with public URLs; always proxied through auth-gated backend endpoint

---

## Form Design

### Two-tier form (v0.9)

| Aspect | Brief form | Full form |
|---|---|---|
| Applies to | Client / PMC / Main Contractor / LDC | Subcontractor personnel |
| Fields required | Name, Company, ID Type/Number, Language, Mobile, Email, Nationality, Designation, Site/Location | All brief fields + Employee No, Government ID, Blood Type, WhatsApp, Sponsor, Date Joined, Reports To |
| Attachments required | ID Copy only | ID Copy + Insurance (+ Legal Agreement if 3rd-party sponsor) |

### Company detection
- Company dropdown populated from `companies` lookup table (with hardcoded fallback)
- Any company whose name contains "subcontractor" (case-insensitive) triggers the full form
- `Subcontractor (other)` selection also shows a secondary subcontractor-name dropdown

### Dynamic lookup tables (v0.9)
- `companies` and `subcontractors` editable via admin panel (Lookup Tables tab)
- Values can be imported from Excel (.xlsx)
- Form dropdowns refresh on every navigation to the form view

---

## Operations

### `init-db` endpoint
- `POST /api/v1/admin/init-db` creates tables, seeds data, and runs migrations
- **Auth-protected** on populated databases (requires `manage_system_settings` permission)
- **Unauth allowed** on fresh databases (no `App_Users` table yet)
- All SQL is idempotent: `CREATE TABLE IF NOT EXISTS`, `ON CONFLICT DO NOTHING`, `ALTER TABLE ADD COLUMN IF NOT EXISTS`
- Safe to re-run — won't drop tables or overwrite existing data

### Backup & Restore
- Railway PostgreSQL add-on provides daily automatic snapshots
- Manual backup via `pg_dump`, restore via `psql`
- After restore, run `init-db` to catch schema columns added since the backup

---

## Storage

| Data | Storage | Production | Local dev |
|---|---|---|---|
| App data | PostgreSQL | Railway-managed DB | Local PostgreSQL portable |
| Attachments | MinIO/S3 / filesystem | MinIO bucket (`attachments`) | `backend/uploads/` |
| Sessions | PostgreSQL `Sessions` table | Railway-managed DB | Local PostgreSQL |

### Attachment storage decision
- **Local filesystem** used for development only
- **MinIO** (S3-compatible) on Railway for production — survives deploys, accessible across instances
- Backend auto-detects MinIO via `MINIO_ENDPOINT` env var; falls back to local filesystem if not set
- Accepts multiple env var naming conventions: `MINIO_ACCESS_KEY` / `MINIO_ROOT_USER` / `AWS_ACCESS_KEY_ID`, `MINIO_SECRET_KEY` / `MINIO_ROOT_PASSWORD` / `AWS_SECRET_ACCESS_KEY`
- Files stored with key pattern: `app_{id}/{type}_{uuid}.{ext}`

---

## Notifications

| Channel | When | From |
|---|---|---|
| Email | Status change (Approved/Rejected) | SMTP (Gmail) configured via `.env` |
| WhatsApp | Status change (Approved/Rejected) | WhatsApp Business API (configurable) |

---

## Workflow

### Modes
| Mode | States | Use case |
|---|---|---|
| `security_only` | Pending Security Clearance → Approved/Rejected | Client/PMC sites |
| `standard` | Pending HR Verification → Pending Security Clearance → Approved/Rejected | Large contractor sites |

---

## Data Model (core tables)

- **`Site_Access_ID`** — main application record (all form fields + workflow state)
- **`Attachments`** — uploaded files linked to applications
- **`App_Users`** — staff accounts (role-based)
- **`External_Submitters`** — external user accounts (org-locked)
- **`Sessions`** — active auth tokens
- **`Lookup_Tables`** / **`Lookup_Values`** — dynamic dropdown data (v0.9)
- **`Submitter_Organizations`** — organizations with submitter group assignment
- **`Workflow_History`** — state-change audit trail
- **`Notification_Log`** — sent notification records
