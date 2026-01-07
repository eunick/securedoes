# Quick Start Guide - SecureDose

This guide will get you up and running with SecureDose in 15 minutes.

## Prerequisites

- Node.js 18+ installed
- npm 9+ installed
- PostgreSQL client (psql) installed
- (Optional) Android Studio for mobile development

## Step 1: Clone and Install (2 min)

```bash
cd /path/to/SecureDose
npm install
```

This installs all dependencies for the monorepo.

## Step 2: Database Setup (5 min)

### Create Neon Database

1. Go to https://neon.tech
2. Sign up for free account
3. Create new project called "securedose"
4. Copy the connection string (looks like: `postgresql://user:pass@host.neon.tech:5432/dbname`)

### Run Migrations

```bash
# Set your database URL
export DATABASE_URL="postgresql://user:pass@host.neon.tech:5432/dbname"

# Run migration
cd packages/db-schema
psql $DATABASE_URL < migrations/001_initial_schema.sql

# Load test data
psql $DATABASE_URL < seed.sql

# Verify
psql $DATABASE_URL -c "\dt"
```

You should see 10+ tables listed.

## Step 3: Configure Environment (2 min)

```bash
# Copy example env file
cp .env.example .env

# Edit .env and add:
# - DATABASE_URL (from step 2)
# - JWT_SECRET (generate random 32+ char string)
# - QR_SIGNING_SECRET (generate random 32+ char string)
```

Generate secrets:
```bash
# On Linux/Mac
openssl rand -hex 32

# Or use this Node.js command
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

## Step 4: Start Backend (2 min)

```bash
cd apps/backend
npm run dev
```

Backend will start at `http://localhost:8787`

Test it:
```bash
curl http://localhost:8787
# Should return: {"service":"SecureDose API","version":"1.0.0",...}
```

## Step 5: Start Mobile App (2 min)

Open new terminal:

```bash
cd apps/mobile
npm run dev
```

App will open at `http://localhost:3000`

## Step 6: Test Login (2 min)

1. Open http://localhost:3000 in browser
2. Click "Caretaker"
3. Login with:
   - Email: `sarah.caretaker@test.com`
   - Password: `TestPassword123!`

**Note:** Authentication is currently mocked. Backend implementation needed.

## Step 7: (Optional) Build Android App

### Install Capacitor

```bash
cd apps/mobile
npx cap add android
```

### Build and Run

```bash
npm run build
npx cap sync android
npx cap open android
```

This opens Android Studio. Click "Run" button to install on emulator/device.

---

## Test Users

All users have password: `TestPassword123!`

| Role | Email | Name |
|------|-------|------|
| Caretaker | sarah.caretaker@test.com | Sarah Johnson |
| Patient | john.patient@test.com | John Smith |
| Family | emma.family@test.com | Emma Smith |

---

## Project Structure

```
securedose/
├── apps/
│   ├── mobile/              # React + Capacitor app
│   │   ├── src/
│   │   │   ├── screens/     # UI screens
│   │   │   ├── services/    # Business logic
│   │   │   └── store/       # State management
│   │   └── capacitor.config.ts
│   │
│   └── backend/             # Cloudflare Workers API
│       ├── src/
│       │   └── routes/      # API endpoints
│       └── wrangler.toml
│
├── packages/
│   ├── shared-types/        # TypeScript types
│   └── db-schema/           # Database migrations
│
└── docs/                    # Documentation
```

---

## Common Commands

### Development
```bash
# Start backend dev server
npm run backend:dev

# Start mobile web dev server
npm run mobile:dev

# Run mobile on Android
npm run mobile:android
```

### Database
```bash
# Connect to database
psql $DATABASE_URL

# View all tables
\dt

# View users
SELECT email, role FROM users;

# Reset database (CAUTION: deletes all data)
psql $DATABASE_URL < migrations/001_initial_schema.sql
psql $DATABASE_URL < seed.sql
```

### Building
```bash
# Build mobile app for production
cd apps/mobile
npm run build

# Deploy backend to Cloudflare
cd apps/backend
npm run deploy
```

---

## Next Steps

1. **Implement Backend Routes**
   - Start with `apps/backend/src/routes/auth.ts`
   - Add database queries using `postgres` library
   - Implement JWT token generation

2. **Connect Mobile to Backend**
   - Update `VITE_API_URL` in `.env`
   - Test API calls from mobile app

3. **Implement QR Code Features**
   - Generate QR codes in backend
   - Test scanning in mobile app

4. **Add Notifications**
   - Schedule local notifications
   - Test TTS voice reminders

See [SESSION_HANDOFF.md](../SESSION_HANDOFF.md) for detailed implementation plan.

---

## Troubleshooting

**Database connection fails:**
- Check `DATABASE_URL` is correct
- Ensure Neon database is running
- Verify IP allowlist in Neon dashboard

**TypeScript errors:**
```bash
npm run typecheck
```

**Module not found errors:**
```bash
# Reinstall dependencies
rm -rf node_modules
npm install
```

**Capacitor sync fails:**
```bash
cd apps/mobile
npx cap sync
```

---

## Resources

- [Full Documentation](../README.md)
- [Session Handoff](../SESSION_HANDOFF.md)
- [Neon Docs](https://neon.tech/docs)
- [Cloudflare Workers Docs](https://developers.cloudflare.com/workers)
- [Capacitor Docs](https://capacitorjs.com/docs)

---

**You're ready to start building! 🚀**
