# SecureDose

A medication tracking and verification mobile application for patients with cognitive or mental health challenges, featuring QR code verification and multi-role support.

## Overview

SecureDose helps improve medication adherence through:
- **Caretaker Mode**: QR code verification of correct dosage
- **Patient Mode**: Voice-assisted reminders and confirmations
- **Family Mode**: Remote monitoring and notifications

## Tech Stack

### Frontend (Mobile App)
- React 18 + TypeScript
- Vite (build tool)
- Capacitor (native wrapper)
- Tailwind CSS
- Zustand (state management)
- Dexie (offline storage)

### Backend (API)
- Cloudflare Workers (serverless)
- Hono (web framework)
- Neon Postgres (database)
- JWT authentication

### Mobile Features
- QR code scanning (`@capacitor-community/barcode-scanner`)
- Local notifications with custom sounds (`@capacitor/local-notifications`)
- Text-to-speech (`@capacitor/text-to-speech`)
- Offline support (IndexedDB)

## Project Structure

```
securedose/
├── apps/
│   ├── mobile/          # Capacitor mobile app
│   │   ├── src/
│   │   │   ├── screens/     # UI screens (by role)
│   │   │   ├── services/    # Business logic
│   │   │   ├── store/       # State management
│   │   │   └── components/  # Reusable UI
│   │   └── android/         # Android native project
│   └── backend/         # Cloudflare Workers API
│       ├── src/
│       │   ├── routes/      # API endpoints
│       │   ├── services/    # Business logic
│       │   ├── middleware/  # Auth, rate limiting
│       │   └── utils/       # Helpers
├── packages/
│   ├── shared-types/    # TypeScript types
│   └── db-schema/       # Database migrations
└── docs/                # Documentation
```

## Quick Start

### Prerequisites
- Node.js 18+
- npm 9+
- Android Studio (for mobile development)

### Installation

```bash
# Install dependencies
npm install

# Setup environment variables
cp .env.example .env
# Edit .env with your configuration

# Start development servers
npm run dev
```

### Development Commands

```bash
# Start all dev servers (via Turborepo)
npm run dev

# Mobile app only
npm run mobile:dev          # Start web dev server (port 5173)
npm run mobile:android      # Build and open in Android Studio

# Backend only
npm run backend:dev         # Start Wrangler dev server (port 8787)
npm run backend:deploy      # Deploy to Cloudflare Workers

# Testing
npm run test                # Run all tests
npm run lint                # Lint all packages
```

### Building the APK

```bash
cd apps/mobile
npm run build
npx cap sync android
```

Then either:
- Open Android Studio: `npx cap open android` and build from there
- Or use Gradle: `cd android && ./gradlew assembleDebug`

The APK will be at: `apps/mobile/android/app/build/outputs/apk/debug/app-debug.apk`

## Documentation

### Getting Started
- [Quick Start Guide](docs/QUICK_START.md) - Get up and running in 15 minutes
- [User Guide](docs/USER_GUIDE.md) - How to use the app (all roles)

### Architecture
- [Frontend Architecture](docs/FRONTEND_ARCHITECTURE.md) - Mobile app overview
- [Backend Architecture](docs/BACKEND_ARCHITECTURE.md) - API and database overview
- [CLAUDE.md](CLAUDE.md) - AI assistant guidance for this codebase

### Development
- [Implementation Checklist](IMPLEMENTATION_CHECKLIST.md) - Track development progress
- [Debugging Log](docs/DEBUGGING_LOG.md) - Fixes and debugging history
- [Session Handoff](SESSION_HANDOFF.md) - Session continuity notes

## Test Users

All users have password: `TestPassword123!`

| Role | Email | Name |
|------|-------|------|
| Caretaker | sarah.caretaker@test.com | Sarah Johnson |
| Patient | john.patient@test.com | John Smith |
| Family | emma.family@test.com | Emma Smith |

## Current Status

### Completed
- Project structure and configuration
- TypeScript types and data models
- Database schema and migrations
- Basic UI screens (all roles)
- QR scanner with camera permissions
- Custom notification sounds (patient, caregiver dose, caregiver missed)
- Backend API routes defined
- APK build configuration

### In Progress
- Backend route implementations
- API integration with mobile app
- QR code generation workflow
- Push notification triggers

## Environment Variables

### Backend (Cloudflare Workers)
```
DATABASE_URL=postgresql://...
JWT_SECRET=your-secret-key
QR_SIGNING_SECRET=your-qr-secret
FCM_SERVER_KEY=optional-for-push
```

### Mobile App
```
VITE_API_URL=http://localhost:8787
```

## License

Private - All Rights Reserved

## Contact

For questions or support, please contact the development team.
