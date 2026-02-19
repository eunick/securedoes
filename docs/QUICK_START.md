# SecureDose Quick Start Guide

Get SecureDose running in 15 minutes.

---

## Prerequisites

- Node.js 18+
- npm 9+
- PostgreSQL client (`psql`) for database setup
- (Optional) Android Studio for building the Android APK
- (Optional) Java 17 for APK builds — `C:\Users\eunick\.jdks\ms-17.0.17`

---

## Step 1 — Install Dependencies

```bash
cd /path/to/SecureDose
npm install
```

---

## Step 2 — Database Setup

### Create a Neon Database

1. Go to [https://neon.tech](https://neon.tech) and sign up for a free account.
2. Create a new project named "securedose".
3. Copy the connection string (format: `postgresql://user:pass@host.neon.tech:5432/dbname`).

### Run All Migrations

```bash
export DATABASE_URL="postgresql://user:pass@host.neon.tech:5432/dbname"

cd packages/db-schema
psql $DATABASE_URL < migrations/001_initial_schema.sql
psql $DATABASE_URL < migrations/002_notification_read_at.sql
psql $DATABASE_URL < migrations/003_refresh_tokens.sql
psql $DATABASE_URL < migrations/004_admin_role.sql
psql $DATABASE_URL < migrations/005_patient_fields.sql

# Load test data
psql $DATABASE_URL < seed.sql

# Verify — should show 10+ tables
psql $DATABASE_URL -c "\dt"
```

---

## Step 3 — Configure Environment

### Backend secrets

```bash
cd apps/backend

# For local development — create .dev.vars (not committed to git)
cat > .dev.vars <<EOF
DATABASE_URL=postgresql://user:pass@host.neon.tech:5432/dbname
JWT_SECRET=<your-32-char-random-string>
QR_SIGNING_SECRET=<your-32-char-random-string>
EOF
```

Generate secrets:
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

### Mobile API URL

```bash
# apps/mobile/.env
VITE_API_URL=https://securedose-api.securedose.workers.dev
# Or for local backend:
# VITE_API_URL=http://localhost:8787
```

---

## Step 4 — Start the Backend (Local Dev)

```bash
npm run backend:dev
# Starts at http://localhost:8787
```

Test:
```bash
curl http://localhost:8787
# {"service":"SecureDose API","version":"1.0.0",...}
```

---

## Step 5 — Start the Mobile App (Browser)

```bash
npm run mobile:dev
# Opens at http://localhost:5173
```

---

## Step 6 — Test Login

Open `http://localhost:5173` in a browser.

| Role | Email | Password |
|------|-------|----------|
| Admin | admin@test.com | TestPassword123! |
| Caretaker | sarah.caretaker@test.com | TestPassword123! |
| Patient | john.patient@test.com | TestPassword123! |
| Family | emma.family@test.com | TestPassword123! |

---

## Step 7 — Deploy Backend to Cloudflare (Production)

Run from **Windows CMD** (not WSL — Wrangler uses Windows binaries):

```cmd
cd C:\Users\eunick\Documents\SecureDose
npm run backend:deploy
```

Set production secrets in Cloudflare (one-time):
```cmd
cd apps/backend
npx wrangler secret put DATABASE_URL
npx wrangler secret put JWT_SECRET
npx wrangler secret put QR_SIGNING_SECRET
```

---

## Step 8 — Build the Android APK

Run all steps from **Windows CMD**:

```cmd
cd C:\Users\eunick\Documents\SecureDose

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

APK output: `apps\mobile\android\app\build\outputs\apk\debug\app-debug.apk`

> Android SDK build tools are Windows-only `.exe` binaries. This step must run in Windows CMD or PowerShell — it will fail in WSL.

---

## Project Structure

```
SecureDose/
├── apps/
│   ├── mobile/              # React + Vite + Capacitor app
│   │   ├── src/
│   │   │   ├── screens/     # UI screens by role
│   │   │   ├── services/    # api.ts, notifications, qrScanner
│   │   │   ├── store/       # authStore (Zustand)
│   │   │   ├── components/  # Button, Card, LoadingSpinner
│   │   │   └── utils/       # generateMarPdf.ts
│   │   └── android/         # Native Android project
│   │
│   └── backend/             # Cloudflare Workers API (Hono)
│       ├── src/
│       │   ├── routes/      # auth, patients, schedules, verify, admin, ...
│       │   ├── middleware/   # auth JWT, rate limiting
│       │   └── utils/       # db, jwt, qr signing
│       └── wrangler.toml
│
├── packages/
│   ├── shared-types/        # TypeScript types (shared)
│   └── db-schema/           # Postgres migrations + seed
│
└── docs/                    # Documentation
```

---

## Common Commands

```bash
# Start everything (Turborepo)
npm run dev

# Start backend only
npm run backend:dev

# Start mobile only
npm run mobile:dev

# Deploy backend (from Windows CMD, monorepo root)
npm run backend:deploy

# Lint all
npm run lint

# Run tests (Vitest)
npm run test
npm run test --workspace=apps/backend
```

---

## Troubleshooting

**PowerShell error: "cannot be loaded because running scripts is disabled"**
```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

**`@rollup/rollup-win32-x64-msvc` missing:**
```cmd
npm install @rollup/rollup-win32-x64-msvc
```

**Database connection fails:**
- Check `DATABASE_URL` is correct.
- Ensure Neon database is active (it may auto-suspend on free tier after inactivity).
- Verify IP allowlist in Neon dashboard.

**TypeScript errors in backend routes:**
These are pre-existing Hono generic typing issues. They do not affect deployment. Do not attempt to fix globally.

**Capacitor sync fails:**
```bash
cd apps/mobile
npx cap sync android
```

**Module not found errors:**
```bash
rm -rf node_modules
npm install
```

---

## Documentation

| File | Contents |
|------|----------|
| `docs/BACKEND_ARCHITECTURE.md` | API routes, database schema, deployment |
| `docs/FRONTEND_ARCHITECTURE.md` | Screen structure, routing, components |
| `docs/USER_GUIDE.md` | How to use each role's features |
| `docs/DEBUGGING_LOG.md` | Known bugs and fixes |
| `docs/SESSION_LOG_2026-02-19.md` | Session log — initial build |
| `docs/SESSION_LOG_2026-02-20.md` | Session log — bug fixes + caretaker dropdown |
| `CLAUDE.md` | Project guidance for Claude Code |
