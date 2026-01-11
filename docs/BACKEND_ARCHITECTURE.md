# SecureDose Backend Architecture (Non-Technical)

This is a simple overview of how the SecureDose backend works, where it runs,
and how it connects to the database. It is written for presentations and
non-technical audiences.

## Overview

SecureDose uses a cloud-hosted backend that powers the mobile app. The backend
handles logins, medication schedules, QR codes, and notifications. All data is
stored in a secure Postgres database.

## Where It Runs (Actual Hosting Setup)

- Cloudflare Workers: hosts the backend API (serverless, auto-scaling)
- Neon Postgres: hosts the database (managed Postgres)

These are the two accounts used for hosting:

- Cloudflare account (Workers hosting + secrets storage)
- Neon account (database hosting + connection string)

## Database Connection (URL and Accounts)

The backend connects to the database using a single environment value called
`DATABASE_URL`. This value is stored as a secret in Cloudflare, and the actual
URL comes from the Neon account.

- Database provider: Neon (Postgres)
- Database URL: stored as Cloudflare secret `DATABASE_URL`
- URL format example: `postgresql://user:pass@host.neon.tech:5432/dbname`

## What The Backend Does

- Authentication and session management (logins, tokens)
- Patient and caregiver records
- Medication schedules and dose confirmation
- QR code generation and verification
- Notifications for missed doses

## How Data Flows (Simple View)

1) The mobile app talks to the Cloudflare-hosted backend.
2) The backend reads or updates data in the Neon database.
3) The backend sends results back to the app.

Note: `docs/QUICK_START.md` mentions that authentication is currently mocked
and needs a full implementation.

## Automated Background Tasks

The backend runs automated jobs on a schedule:

- Generate upcoming QR codes
- Clean up old QR codes
- Detect missed doses and create notifications

## Local Development (Optional)

Developers can run the backend locally using the same database connection and
the same hosting stack. Setup steps are in `docs/QUICK_START.md`.

## Deployment (Optional)

- The backend is deployed to Cloudflare Workers.
- The database connection and secrets are stored in Cloudflare.
