# Ta'awon P3 SMS — Project Context for OpenCode

## Architecture
- **Backend**: Single-file FastAPI (`backend/main.py` ~3100 lines) — raw asyncpg SQL, no ORM, no routers
- **Frontend**: Single HTML file (`frontend/index.html` ~3130 lines) — inline CSS + JS, no build step, no framework
- **Deployment**: Railway (Nixpacks for backend, `npx serve` for frontend)
- **Database**: PostgreSQL (Railway plugin in prod, local Postgres 17 for dev)

## Two separate Git repos
1. Backend: `https://github.com/abkader60-sketch/taawon-sms-OC-backend.git` (local: `D:\Opencode\sms-project\backend`)
2. Frontend: `https://github.com/abkader60-sketch/taawon-sms-OC-frontend-.git` (local: `D:\Opencode\sms-project\frontend`)

## Railway URLs
- Backend: `https://securitysms-production.up.railway.app`
- Frontend: `https://taawon-sms-frontend-production.up.railway.app`

## Seeded Users (initial password: `ChangeMe123!`)
- `omar.almaqhawi` — Administrator (full access, `manage_system_settings`)
- `abdulkader.abdullatif` — Security Clearance (approve/reject, view all, edit)
- `hind.alahmari` — Reviewer (read-only)
- `badriah.rizqallah` — Submitter (submit only)
- `omar.almaqhawi`'s password set to `@Khobar123` (no forced change)
- All others force password change on first login

## Frontend API URL
Set via `<meta name="sms-api-base">` in `frontend/index.html:12`. Production points to Railway backend. Fallback to `http://127.0.0.1:8000`.

## Key Conventions
- **ID fields always use camelCase in HTML** (`fullName`, `companyName`, `iqamaNumber`, `dateOfBirth`, `mobileNo`, `whatsappNo`, etc.)
- **Backend uses snake_case** for DB columns and Form fields (`full_name`, `gov_id_number`, `date_of_birth`)
- **Form submission**: button is `type="button"` with `onclick="doSubmitApplication()"`, form has `onsubmit="doSubmitApplication(); return false;"`
- **Variant system**: `.full-only` shown for subcontractors, hidden for Client/PMC/Main/LDC. `.brief-only` is the reverse (currently unused in HTML but logic exists). `.subcontractor-only` shown only when "Subcontractor" selected.
- **Required fields**: managed dynamically by `setRequiredAttribute()` and `handleSponsorChange()`/`handleWhatsAppChange()`. Never hardcode `required` in HTML except for fields always required (idCopyUpload, photoUpload).
- **Permissions**: checked via `userCan(permKey)` function. Available keys: `submit_application`, `approve_security`, `approve_hr`, `view_all_applications`, `edit_application`, `manage_users`, `manage_external_users`, `manage_system_settings`.

## Form Fields (brief vs full)
| Field | ID | Brief | Full |
|-------|-----|-------|------|
| Project | `location` | ✓ | ✓ |
| Company | `companyName` | ✓ | ✓ |
| Subcontractor Name | `subcontractorName` | hidden | shown |
| ID Type | `govIdType` (radio) | ✓ | ✓ |
| Full Name | `fullName` | ✓ | ✓ |
| Gender | `gender` | ✓ | ✓ |
| Nationality | `nationality` | ✓ | ✓ |
| Date of Birth | `dateOfBirth` | ✓ | ✓ |
| Iqama/ID Number | `iqamaNumber` | ✓ | ✓ |
| Iqama Expiry | `iqamaExpiry` | ✓ | ✓ |
| Passport Number | `passportNo` | optional | optional |
| Passport Expiry | `passportExpiry` | optional | optional |
| Blood Type | `bloodType` | ✓ | ✓ |
| Employee Number | `employeeNo` | optional | optional |
| Language | `language` | ✓ | ✓ |
| Safety Induction Date | `safetyInductionDate` | optional | optional |
| Date Joined | `dateJoined` | ✓ | ✓ |
| Reports To | `reportsTo` | optional | optional |
| Mobile Number | `mobileNo` | ✓ | ✓ |
| Same as WhatsApp | `sameAsWhatsapp` | ✓ | ✓ |
| WhatsApp Number | `whatsappNo` | shown/unchecked | shown/unchecked |
| Email | `email` | ✓ | ✓ |
| Trade (Iqama) | `tradeIqama` | — | ✓ |
| Same Trade checkbox | `sameTradeAsIqama` | — | ✓ |
| Trade (Project) | `tradeProject` | — | ✓ |
| ID Copy | `idCopyUpload` | required | required |
| Insurance | `insuranceUpload` | — | required |
| Insurance Expiry | `insuranceExpiry` | — | required |
| Personal Photo | `photoUpload` | required | required |
| 3rd Party Sponsor checkbox | `sponsorIsThirdParty` | — | shown |
| Sponsor Name | `sponsorName` | — | shown when checkbox checked |
| Legal Agreement | `legalAgreementUpload` | — | shown when checkbox checked |

## Key Business Rules
1. **Company containing "subcontractor" (case-insensitive)** → Full form. Client/PMC/Main/LDC → Brief form.
2. **External submitters**: form variant locked by `submitter_group` (group_1=brief, group_2=full). Can only access `/submit`, `/organizations`, `/auth/*`.
3. **Workflow modes**: `security_only` (default, single-stage) vs `standard` (HR → Security two-stage).
4. **Edit allowed** only while application is in a pending state (not Approved/Rejected).
5. **Export respects search/status filters** — passes `?status=&search=` query params.

## Recent Fixes (know these before making changes)
- **MinIO init** (Jun 2026): was failing on subsequent deploys because `create_bucket()` throws `BucketAlreadyExists`. Fixed with `head_bucket` check first.
- **Form submit** (Jun 2026): Changed from `type="submit"` + `addEventListener('submit')` to `type="button"` + `onclick` + `onsubmit`. Must keep both handlers for Enter key and button click.
- **Date parsing** (May 2026): `parse_date_or_none` in backend handles 9+ formats plus raw datetime/date objects.
- **Search** (May 2026): Server-side ILIKE across 9 fields (full_name, organization, subcontractor, gov_id_number, nationality, mobile_number, email, employee_number, application_id).
- **Export download** (May 2026): Uses XMLHttpRequest with `responseType='blob'` instead of fetch + blob() to avoid "Failed to fetch" on large files.
- **Photo preview** (May 2026): Shows "Photo not available" instead of stuck "Loading photo..." when 410 returned.
- **Non Grata** (May 2026): Moved from admin sub-tab to standalone sidebar menu item.

## Key Files
- `backend/main.py` — everything backend
- `frontend/index.html` — everything frontend
- `frontend/DESIGN_PARAMETERS.md` — architecture doc
- `frontend/ROADMAP.md` — version history
- `Security_SMS_Archive/DEPLOYMENT_GUIDE.md` — Railway deploy steps
- `User Guide.md` — comprehensive user manual
- `HANDBOOK.md` — handover doc with credentials & links

## Admin Tabs (in order)
`users` → `roles` → `groups` → `external` → `lookups` → `settings` → `import` → `nongrata` → `induction` → `sysusers`

When adding a new admin tab, add it to `switchAdminTab()` and the tab bar HTML, and add the tab ID to the admin tabs JS.

## Useful Commands
```powershell
# Backend dev
cd backend; uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Frontend dev (when running standalone)
cd frontend; npx serve -s . -l 3000

# Git push both
cd backend; git add -A; git commit -m "msg"; git push
cd frontend; git add -A; git commit -m "msg"; git push

# Init DB (run after backend is running, e.g. via browser at backend URL)
# - Requires manage_system_settings if DB is not fresh
# - Idempotent — safe to run repeatedly
```
