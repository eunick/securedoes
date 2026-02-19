# SecureDose Backend Architecture

This document describes how the SecureDose backend works, where it runs, and how it connects to the database. Written for both technical and non-technical audiences.

---

## Overview

SecureDose uses a cloud-hosted backend that powers the mobile app. The backend handles logins, medication schedules, QR code generation and verification, notifications, and all admin management. All data is stored in a secure Postgres database hosted on Neon.

---

## Where It Runs

| Service | Provider | Purpose |
|---------|----------|---------|
| Backend API | Cloudflare Workers | Serverless, auto-scaling API |
| Database | Neon (Postgres) | Managed Postgres — stores all app data |

**Live API URL:** `https://securedose-api.securedose.workers.dev`

---

## Database Connection

The backend connects using a single environment variable `DATABASE_URL`, stored as a Cloudflare Worker secret.

- Provider: Neon (Postgres)
- Format: `postgresql://user:pass@host.neon.tech:5432/dbname`
- Set in Cloudflare: `wrangler secret put DATABASE_URL`

Other secrets required:
- `JWT_SECRET` — signs authentication tokens
- `QR_SIGNING_SECRET` — signs QR code payloads

---

## API Routes

All routes are under the base URL. Auth is via Bearer JWT token.

### Authentication (`/auth/*`)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/auth/login` | Login — returns accessToken + refreshToken |
| POST | `/auth/register` | Register new caretaker or family user |
| POST | `/auth/refresh` | Refresh an expired access token |
| POST | `/auth/logout` | Invalidate refresh token |

> Admin users cannot self-register. They must be created directly in the database.

### Patients (`/patients/*`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/patients` | List patients (caretaker sees own; admin sees all) |
| GET | `/patients/:id` | Get single patient |
| GET | `/patients/:id/compliance` | Adherence rate for current week |

### Schedules (`/schedules/*`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/schedules/:patientId` | List medication schedules for a patient |

### Verification

| Method | Path | Description |
|--------|------|-------------|
| POST | `/verify` | Verify a scanned QR code |
| POST | `/confirm` | Patient confirms taking a dose |
| GET | `/events/:patientId` | Verification event history (adherence calendar) |

### Notifications

| Method | Path | Description |
|--------|------|-------------|
| GET | `/notifications` | List notifications (admin: all; family: own) |
| POST | `/notifications/:id/read` | Mark notification as read |

### Admin (`/admin/*`) — Requires admin role

| Method | Path | Description |
|--------|------|-------------|
| GET | `/admin/stats` | Dashboard counts (patients, caretakers, family, missed doses) |
| GET | `/admin/patients` | All patients with assigned caretaker name |
| POST | `/admin/patients` | Create a new patient |
| PUT | `/admin/patients/:id` | Update patient profile |
| POST | `/admin/patients/:id/deactivate` | Deactivate a patient |
| POST | `/admin/patients/:id/medications` | Add medication + schedule to a patient |
| DELETE | `/admin/medications/:id` | Remove (deactivate) a medication |
| GET | `/admin/caretakers` | All caretakers with patient counts |
| GET | `/admin/family` | All family members with patient links |
| POST | `/admin/users` | Create a new caretaker or family user |
| GET | `/admin/link-requests` | Pending family link requests |
| POST | `/admin/link-requests/:id/approve` | Approve a family link request |
| POST | `/admin/link-requests/:id/reject` | Reject a family link request |
| GET | `/admin/patients/:id/prescriptions` | List prescriptions for a patient |
| POST | `/admin/patients/:id/prescriptions` | Upload a prescription image |
| GET | `/admin/patients/:id/emar` | Patient + schedules + recent verification events |

---

## Database Schema (Key Tables)

| Table | Purpose |
|-------|---------|
| `users` | All users — role: admin, caretaker, patient, family |
| `patients` | Patient records (full_name, dob, mrn, address, allergies, condition) |
| `medications` | Medications per patient (name, dosage, instructions, route) |
| `schedules` | When medications are taken (scheduled_time, days_of_week) |
| `verification_events` | QR scan + confirmation records |
| `notification_log` | Missed dose and verification notifications |
| `patient_family_links` | Links between family users and patients |
| `audit_log` | Audit trail; also stores pending link requests (action='link_request') |
| `prescriptions` | Uploaded prescription images per patient |
| `refresh_tokens` | Active refresh token store |

### Important Notes on Schema

- `users.role` is a `VARCHAR` with a `CHECK` constraint (not a PostgreSQL enum). Adding new roles requires dropping and re-adding the constraint — see `migrations/004_admin_role.sql`.
- Family link requests are stored in `audit_log` with `action = 'link_request'`. There is no separate `link_requests` table.
- `medications` and `schedules` are separate tables. A medication must have a corresponding schedule to appear in the mobile app.

---

## Migrations

Apply in order against the Neon database:

```bash
psql $DATABASE_URL < migrations/001_initial_schema.sql
psql $DATABASE_URL < migrations/002_notification_read_at.sql
psql $DATABASE_URL < migrations/003_refresh_tokens.sql
psql $DATABASE_URL < migrations/004_admin_role.sql
psql $DATABASE_URL < migrations/005_patient_fields.sql
```

All five migrations have been applied to the production Neon database.

---

## Authentication Flow

1. Client POSTs credentials to `/auth/login`.
2. Backend verifies password hash with bcrypt.
3. Backend returns `accessToken` (15 min TTL) and `refreshToken` (7 day TTL).
4. Client includes `Authorization: Bearer <accessToken>` on all subsequent requests.
5. When `accessToken` expires, client POSTs `refreshToken` to `/auth/refresh` to get a new pair.

---

## Automated Background Tasks

The backend runs scheduled jobs via Cloudflare Workers cron triggers:

- Generate upcoming QR codes for the next 24 hours
- Clean up expired QR codes
- Detect missed doses and create notification records

---

## Deployment

Deploy the backend from the monorepo root using Windows CMD (not WSL — Wrangler uses Windows binaries):

```cmd
cd C:\Users\eunick\Documents\SecureDose
npm run backend:deploy
```

> Do NOT run `wrangler deploy` from WSL. The `workerd` binary is a Windows `.exe` and will fail in WSL.

### Setting Secrets in Cloudflare

```cmd
cd apps/backend
npx wrangler secret put DATABASE_URL
npx wrangler secret put JWT_SECRET
npx wrangler secret put QR_SIGNING_SECRET
```

---

## Local Development

```bash
# From monorepo root
npm run backend:dev
# Backend starts at http://localhost:8787
```

The local backend reads secrets from `apps/backend/.dev.vars` (not committed to git).

---

## Pre-existing TypeScript Errors

All backend route files have type errors from Hono's generic typing (`c.get('user')`, `c.env?.DATABASE_URL`). These are pre-existing and do not affect runtime. Wrangler deploys without enforcing strict TypeScript. Do not attempt to fix these globally.
