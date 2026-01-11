# SecureDose Frontend Architecture (Non-Technical)

This is a simple overview of how the SecureDose mobile app is built, how it
connects to the backend, and how it runs on devices. It is written for
presentations and non-technical audiences.

## Overview

SecureDose is a mobile app for patients, caregivers, and family members. The app
provides schedules, QR scanning, confirmations, and notifications. It connects
to the SecureDose backend to sync data and receive updates.

## Where The App Runs

- Primary target: Android phones/tablets
- Built as a web app and packaged into a native app using Capacitor

## What The App Uses (High Level)

- React + TypeScript: the UI layer
- Vite: the build tool used during development
- Capacitor: wraps the app to run on Android devices

## Connecting to the Backend

The app talks to the backend using a configurable API URL:

- `VITE_API_URL` is set in the mobile app’s `.env` file
- If not set, it defaults to `http://localhost:8787` for local development

This allows the same app build to point to local, staging, or production
backends depending on the environment.

## Core App Capabilities

- Role-based screens (patient, caregiver, family)
- QR code scanning for dose verification
- Local notifications for reminders
- Offline-friendly storage to keep data available without internet

## How The App Works (Simple Flow)

1) A user opens the app and signs in.
2) The app loads schedules and patient data from the backend.
3) The user scans or confirms doses; the app sends results to the backend.
4) The app shows reminders and notifications as needed.

## Local Development (Optional)

Developers can run the app in a browser or on Android:

- Start the dev server: `npm run mobile:dev`
- Run on Android: `npm run mobile:android`

Detailed steps are in `docs/QUICK_START.md`.
