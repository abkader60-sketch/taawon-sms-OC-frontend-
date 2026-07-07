# Ta'awon P3 SMS — Project Handover

## Quick Access

| Item | Value |
|------|-------|
| **Backend GitHub** | `https://github.com/abkader60-sketch/taawon-sms-OC-backend.git` |
| **Frontend GitHub** | `https://github.com/abkader60-sketch/taawon-sms-OC-frontend-.git` |
| **Backend (Railway)** | `https://securitysms-production.up.railway.app` |
| **Frontend (Railway)** | `https://taawon-sms-frontend-production.up.railway.app` |
| **Backend Health** | `https://securitysms-production.up.railway.app/` |
| **FastAPI docs** | `https://securitysms-production.up.railway.app/docs` |

> The Railway projects are owned by the account that deployed them. Transfer ownership via Railway dashboard → Project Settings → Transfer.

## GitHub Access

Both repos are private, owned by `abkader60-sketch`. Add your colleague as a collaborator:
- Backend: https://github.com/abkader60-sketch/taawon-sms-OC-backend/settings/access
- Frontend: https://github.com/abkader60-sketch/taawon-sms-OC-frontend-/settings/access

## Railway Access

Two services are deployed on Railway:
1. **Backend API** — `securitysms-production` — runs `uvicorn main:app --host 0.0.0.0 --port $PORT` via Nixpacks
2. **Frontend** — `taawon-sms-frontend-production` — runs `npx serve -s . -l $PORT`

Add colleague via Railway dashboard → Project → Members.

## Environment Variables (Backend)

Set these on Railway backend service. Local dev copies are in `backend/.env`:

| Variable | Value / Notes |
|----------|--------------|
| `DATABASE_URL` | (Optional — Railway PostgreSQL plugin provides this automatically; overrides DB_* vars) |
| `DB_HOST` / `DB_PORT` / `DB_NAME` / `DB_USER` / `DB_PASSWORD` | Fallback if no `DATABASE_URL` |
| `CORS_ORIGINS` | `*` (or restrict to frontend URL) |
| `SMTP_HOST` / `SMTP_PORT` / `SMTP_USERNAME` / `SMTP_PASSWORD` / `SMTP_FROM` | Gmail app password for notifications |
| `MINIO_ENDPOINT` | `minio.railway.internal:9000` (on Railway) |
| `MINIO_ACCESS_KEY` / `MINIO_SECRET_KEY` | MinIO credentials |
| `MINIO_BUCKET` | `attachments` |

**Credentials (local `.env` file):**
- DB Password: `Omar123` (local dev only)
- SMTP Gmail: `abkader60@gmail.com` / app password: `zdosgtburpeddeih`
- Seeded default password (all users): `ChangeMe123!`

## Seeded Staff Accounts

| Full Name | Username | Role | Notes |
|-----------|----------|------|-------|
| Omar Abdulaziz Almaqhawi | `omar.almaqhawi` | Administrator | `manage_system_settings` — can do everything |
| Abdulkader Abdullatif | `abdulkader.abdullatif` | Security Clearance | Can approve/reject, view all, edit pending |
| Hind Mana Alahmari | `hind.alahmari` | Reviewer | Read-only access |
| Badriah Rizq Allah | `badriah.rizqallah` | Submitter | Can only submit applications |

All users require password change on first login. Initial password: `ChangeMe123!`

## Local Development Setup

```powershell
# Backend
cd backend
pip install -r requirements.txt
# Ensure PostgreSQL is running, create database "id_system"
uvicorn main:app --reload --host 0.0.0.0 --port 8000
# Then init the database via browser:
# GET http://localhost:8000 -> click "Re-run Init-DB" button
# OR POST http://localhost:8000/api/v1/admin/init-db

# Frontend (separate terminal)
cd frontend
npx serve -s . -l 3000
# Update the <meta name="sms-api-base"> in index.html to point to local backend
```

## Code Architecture

### Backend (`backend/main.py` ~3100 lines)
Single-file FastAPI app. No framework, no routers, no SQLAlchemy — raw SQL via `asyncpg`.

### Frontend (`frontend/index.html` ~3130 lines)
Single HTML file containing all CSS (inline `<style>`) and JS (inline `<script>`). No build step, no framework, no npm dependencies for production.

### Key Files in Repository
| File | Purpose |
|------|---------|
| `backend/main.py` | Entire FastAPI backend (schema, endpoints, auth, workflow, export, import) |
| `backend/requirements.txt` | Python dependencies |
| `backend/railway.json` | Railway deployment config |
| `frontend/index.html` | Entire single-page frontend |
| `archive/DEPLOYMENT_GUIDE.md` | Step-by-step Railway deploy guide (slightly outdated) |
| `archive/ROADMAP.md` | Version history |
| `archive/DESIGN_PARAMETERS.md` | Architecture decisions |

## Permissions Reference

| Permission Key | Description |
|---------------|-------------|
| `submit_application` | Submit new applications |
| `approve_security` | Approve/reject at Security Clearance stage |
| `approve_hr` | Approve/reject at HR Verification stage |
| `view_all_applications` | View all applications and dashboard |
| `edit_application` | Edit pending applications |
| `manage_users` | Admin staff users, roles, groups |
| `manage_external_users` | Admin external submitters |
| `manage_system_settings` | System settings, lookup tables, Non Grata, Induction WB, System Users, init-db, backlog import |

## Known Issues / Unresolved

1. **Form submission not working** — latest fix (commit `a42fb74`) changes button to `type="button"` with `onclick` handler. If still broken on Railway, must redeploy.
2. **Old attachments lost** — files uploaded before the MinIO fix (commit `6f2af66`) were on ephemeral local disk and are gone. Re-upload by editing each application.
3. **Non Grata checking** — table and import exist, but no real-time check during submission/approval. Pending feature.
4. **WhatsApp notifications** — logged as `pending_whatsapp_api` but actual sending not implemented.

## OpenCode Context

An `AGENTS.md` file has been created at the project root with detailed session context. When your colleague opens this project in OpenCode, the agent will automatically read it and have full context about the codebase, architecture, credentials, and recent changes.
