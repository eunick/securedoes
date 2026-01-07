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
- Tailwind CSS + shadcn/ui
- Zustand (state management)
- Dexie (offline storage)

### Backend (API)
- Cloudflare Workers (serverless)
- Hono (web framework)
- Neon Postgres (database)
- JWT authentication

### Mobile Features
- QR code scanning (`@capacitor-community/barcode-scanner`)
- Local notifications (`@capacitor/local-notifications`)
- Text-to-speech (`@capacitor/text-to-speech`)
- Offline support (IndexedDB)

## Project Structure

```
securedose/
├── apps/
│   ├── mobile/          # Capacitor mobile app
│   └── backend/         # Cloudflare Workers API
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

# Run database migrations
cd packages/db-schema
npm run migrate

# Start development servers
npm run dev
```

### Mobile App Development

```bash
# Start web dev server
npm run mobile:dev

# Build and sync to Android
npm run mobile:android
```

### Backend Development

```bash
# Start local Cloudflare Workers dev server
npm run backend:dev

# Deploy to Cloudflare
npm run backend:deploy
```

## Documentation

- **[SESSION_HANDOFF.md](SESSION_HANDOFF.md)** - ⚠️ **READ THIS FIRST** - Critical continuity documentation
- [Quick Start Guide](docs/QUICK_START.md) - Get up and running in 15 minutes
- [Implementation Checklist](IMPLEMENTATION_CHECKLIST.md) - Track your progress

## 📝 Important: Session Continuity

**If you're working on this project across multiple sessions:**

1. **ALWAYS** read `SESSION_HANDOFF.md` at the start of each session
2. **ALWAYS** update `SESSION_HANDOFF.md` before ending your session (especially if you might run out of tokens)
3. Document what you completed, what's blocked, and what's next

This ensures you can pick up exactly where you left off, even if your session ends unexpectedly.

## Current Status

This is a scaffolded project with:
- ✅ Project structure and configuration
- ✅ TypeScript types and data models
- ✅ Database schema and migrations
- ✅ Basic UI screens (all roles)
- ✅ Service modules (QR scanner, notifications, storage)
- ✅ Backend API skeleton (routes defined)
- ⏳ Backend implementation (TODO)
- ⏳ API integration (TODO)
- ⏳ QR code generation (TODO)
- ⏳ Notification triggers (TODO)

See [SESSION_HANDOFF.md](SESSION_HANDOFF.md) for next steps.

## License

Private - All Rights Reserved

## Contact

For questions or support, please contact the development team.
