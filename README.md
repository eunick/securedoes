# SecureDose

A medication tracking and verification mobile application for patients with cognitive or mental health challenges. Features QR code dose verification, a 28-day eMAR chart, PDF export, and four user roles.

**Live API:** `https://securedose-api.securedose.workers.dev`

---

## Overview

SecureDose improves medication adherence through a verified, role-based workflow:

- **Admin** — manages all patients, caretakers, family members, and medications
- **Caretaker** — verifies doses via QR scan, views eMAR chart, generates printable QR codes
- **Patient** — receives reminders, confirms doses in-app
- **Family** — monitors dose activity and notifications remotely

---

## Tech Stack

### Frontend (Mobile App)
- React 18 + TypeScript
- Vite (build tool)
- Capacitor 5 (Android native wrapper)
- Tailwind CSS
- Zustand (state management)
- Dexie (offline storage / IndexedDB)
- jsPDF + jspdf-autotable (MAR chart PDF export)

### Backend (API)
- Cloudflare Workers (serverless, auto-scaling)
- Hono (web framework)
- Neon Postgres (managed database)
- JWT authentication (bcryptjs password hashing)
- Zod (request validation)

### Native Features
- QR code scanning (`@capacitor-community/barcode-scanner`)
- Local notifications (`@capacitor/local-notifications`)
- File system access (`@capacitor/filesystem`) — PDF export on Android

---

## Project Structure

```
SecureDose/
├── apps/
│   ├── mobile/                  # React + Vite + Capacitor app
│   │   ├── src/
│   │   │   ├── screens/
│   │   │   │   ├── admin/       # Admin role screens
│   │   │   │   ├── caretaker/   # Caretaker role screens
│   │   │   │   ├── patient/     # Patient role screens
│   │   │   │   └── family/      # Family role screens
│   │   │   ├── services/        # api.ts, notifications, qrScanner
│   │   │   ├── store/           # authStore (Zustand)
│   │   │   ├── components/      # Button, Card, LoadingSpinner
│   │   │   └── utils/           # generateMarPdf.ts
│   │   └── android/             # Native Android project
│   │
│   └── backend/                 # Cloudflare Workers API
│       ├── src/
│       │   ├── routes/          # auth, patients, schedules, verify, admin, ...
│       │   ├── middleware/      # JWT auth, rate limiting
│       │   └── utils/           # db, jwt, qr signing, validation
│       └── wrangler.toml
│
├── packages/
│   ├── shared-types/            # TypeScript types (shared)
│   └── db-schema/               # Postgres migrations + seed data
│
└── docs/                        # Documentation
```

---

## Quick Start

### Prerequisites
- Node.js 18+
- npm 9+
- PostgreSQL client (`psql`) for database setup
- (Android builds only) Java 17 + Android Studio — Windows only

### Install

```bash
npm install
```

### Database Setup

```bash
export DATABASE_URL="postgresql://user:pass@host.neon.tech:5432/dbname"

cd packages/db-schema
psql $DATABASE_URL < migrations/001_initial_schema.sql
psql $DATABASE_URL < migrations/002_notification_read_at.sql
psql $DATABASE_URL < migrations/003_refresh_tokens.sql
psql $DATABASE_URL < migrations/004_admin_role.sql
psql $DATABASE_URL < migrations/005_patient_fields.sql
psql $DATABASE_URL < seed.sql
```

### Start Development

```bash
# Backend (http://localhost:8787)
npm run backend:dev

# Mobile app in browser (http://localhost:5173)
npm run mobile:dev

# Or start everything at once
npm run dev
```

### Build the Android APK

Run from **Windows CMD** (not WSL — build tools are Windows `.exe` binaries):

```cmd
cd C:\path\to\SecureDose

REM Build web assets
npm run build --workspace=apps/mobile

REM Sync to Android project
cd apps\mobile
npx cap sync android

REM Build APK
set "JAVA_HOME=C:\Users\eunick\.jdks\ms-17.0.17"
cd android
gradlew.bat assembleDebug
```

Output: `apps/mobile/android/app/build/outputs/apk/debug/app-debug.apk`

### Deploy Backend

From **Windows CMD** at the monorepo root:

```cmd
npm run backend:deploy
```

---

## Test Users

All accounts use password: `TestPassword123!`

| Role | Email | Name |
|------|-------|------|
| Admin | admin@test.com | System Admin |
| Caretaker | sarah.caretaker@test.com | Sarah Johnson |
| Caretaker | mike.caretaker@test.com | Mike Davis |
| Patient | john.patient@test.com | John Smith |
| Family | emma.family@test.com | Emma Smith |
| Family | robert.family@test.com | Robert Smith |

> Admin accounts cannot be created through the app — use the database or the Admin > Manage Users screen to create caretaker/family accounts.

---

## Current Status

### Completed
- Four user roles: Admin, Caretaker, Patient, Family
- Full JWT authentication with refresh tokens (bcrypt password hashing)
- Database schema with 5 migrations applied to production
- Admin dashboard with live stats
- Admin patient management: create, edit, deactivate, caretaker transfer
- Admin medication management: add (with schedule), remove
- Admin user creation: caretaker and family via in-app modal
- Admin 5-tab Patient Detail: Profile, Medications, eMAR, Calendar, Prescriptions
- Admin caretaker and family member management, link request approval
- Caretaker QR code scanning and dose verification
- Caretaker QR code generation (printable)
- Caretaker + Admin eMAR chart: 28-day grid with time-of-day sections
- PDF export of MAR chart (web: browser download; Android: saves to Documents)
- Patient dose confirmation and reminder notifications
- Family activity log with history and calendar tabs
- Family compliance monitoring with progress bar
- Offline storage (IndexedDB via Dexie)
- Cloudflare Workers backend fully deployed
- Android APK build pipeline

### Known Limitations
- Patient deactivation is a soft UI remove only (no `active` column on `patients` table yet)
- No self-service password reset (contact admin)
- Android PDF saves to Documents folder — no in-app PDF viewer
- Compliance calculation uses UTC timezone (no per-patient timezone support)

---

## Environment Variables

### Backend (`apps/backend/.dev.vars` for local / Cloudflare secrets for production)
```
DATABASE_URL=postgresql://...
JWT_SECRET=<32+ char random string>
QR_SIGNING_SECRET=<32+ char random string>
FCM_SERVER_KEY=<optional, for push notifications>
ALLOWED_ORIGINS=<optional, comma-separated>
```

### Mobile (`apps/mobile/.env`)
```
VITE_API_URL=https://securedose-api.securedose.workers.dev
# For local dev:
# VITE_API_URL=http://localhost:8787
```

---

## Documentation

| File | Description |
|------|-------------|
| [docs/QUICK_START.md](docs/QUICK_START.md) | Full setup and build guide |
| [docs/USER_GUIDE.md](docs/USER_GUIDE.md) | How to use each role's features |
| [docs/FRONTEND_ARCHITECTURE.md](docs/FRONTEND_ARCHITECTURE.md) | Screen structure, routing, components |
| [docs/BACKEND_ARCHITECTURE.md](docs/BACKEND_ARCHITECTURE.md) | API routes, database schema, deployment |
| [docs/DEBUGGING_LOG.md](docs/DEBUGGING_LOG.md) | Bug fixes and debugging history |
| [docs/SESSION_LOG_2026-02-19.md](docs/SESSION_LOG_2026-02-19.md) | Session log — initial full build |
| [docs/SESSION_LOG_2026-02-20.md](docs/SESSION_LOG_2026-02-20.md) | Session log — bug fixes + caretaker dropdown |
| [CLAUDE.md](CLAUDE.md) | Guidance for Claude Code AI assistant |

---

## License

Private — All Rights Reserved
