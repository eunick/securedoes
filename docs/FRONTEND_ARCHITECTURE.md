# SecureDose Frontend Architecture

This document describes how the SecureDose mobile app is built, its screen structure, how it connects to the backend, and how it runs on devices.

---

## Overview

SecureDose is a mobile app for four user roles: Admin, Caretaker, Patient, and Family. It is built as a React web app, then wrapped into a native Android app using Capacitor.

**Live backend:** `https://securedose-api.securedose.workers.dev` (configured via `VITE_API_URL` in `.env`)

---

## Technology Stack

| Layer | Technology |
|-------|-----------|
| UI Framework | React 18 + TypeScript |
| Build Tool | Vite |
| Native Wrapper | Capacitor 5 |
| Styling | Tailwind CSS |
| State Management | Zustand (`useAuthStore`) |
| Routing | React Router v6 |
| Offline Storage | Dexie (IndexedDB) |
| PDF Generation | jsPDF + jspdf-autotable |

---

## Project Structure

```
apps/mobile/src/
├── components/          # Reusable UI: Button, Card, LoadingSpinner
├── screens/
│   ├── LoginScreen.tsx
│   ├── admin/           # Admin role screens
│   ├── caretaker/       # Caretaker role screens
│   ├── patient/         # Patient role screens
│   └── family/          # Family role screens
├── services/
│   ├── api.ts           # All API calls (singleton)
│   ├── notifications.ts # Local notifications (Capacitor)
│   ├── qrScanner.ts     # QR barcode scanning (Capacitor)
│   └── offlineStorage.ts# Dexie IndexedDB
├── store/
│   └── authStore.ts     # Zustand: user, token, role flags
├── utils/
│   └── generateMarPdf.ts# jsPDF MAR chart PDF export
└── App.tsx              # Route definitions + role guards
```

---

## Role-Based Routing

Each role has its own set of screens and default route. Route guards in `App.tsx` redirect unauthenticated users to `/login` and wrong-role users to their own dashboard.

| Role | Default Route | Key Screens |
|------|--------------|-------------|
| `admin` | `/admin/dashboard` | Dashboard, Patients, PatientDetail, Caretakers, Family, ManageUsers, More |
| `caretaker` | `/caretaker/dashboard` | Dashboard, QR Scanner, QR Generator, MAR Chart |
| `patient` | `/patient/home` | Home, Schedule |
| `family` | `/family/dashboard` | Dashboard, Activity |

### Route Guard Pattern

```tsx
// Accessible to admin OR caretaker:
element={(isCaretaker || isAdmin) ? <CaretakerMARChartScreen /> : <Navigate to="/login" replace />}
```

---

## Screen Reference

### Admin Screens (`screens/admin/`)

| Screen | Path | Description |
|--------|------|-------------|
| `AdminDashboard` | `/admin/dashboard` | Stats cards + tab bar |
| `AdminPatientsScreen` | `/admin/patients` | Patient list, search, create patient (with caretaker dropdown) |
| `AdminPatientDetailScreen` | `/admin/patients/:id` | 5 tabs: Profile, Medications, eMAR, Calendar, Prescriptions |
| `AdminCaretakersScreen` | `/admin/caretakers` | Caretaker list with patient counts |
| `AdminFamilyScreen` | `/admin/family` | Family members + link request approval |
| `AdminManageUsersScreen` | `/admin/manage-users` | Create caretaker/family users via modal |
| `AdminMoreScreen` | `/admin/more` | Profile, settings, manage users link, logout |

**AdminPatientDetailScreen tabs:**
- **Profile** — Edit name, address, allergies, condition; Transfer Caretaker dropdown
- **Medications** — List active meds; Add Medicine form (name, dosage, route, instructions, scheduled time)
- **eMAR** — 28-day MAR chart grid; time-of-day sections; X marks for verified doses; Print/PDF button
- **Calendar** — Monthly calendar with medication schedule overlay
- **Prescriptions** — Upload and view prescription images

### Caretaker Screens (`screens/caretaker/`)

| Screen | Path | Description |
|--------|------|-------------|
| `CaretakerDashboard` | `/caretaker/dashboard` | Patient list, compliance bars, quick actions |
| `CaretakerQRScannerScreen` | `/caretaker/scan` | QR code scanner for dose verification |
| `CaretakerQRCodeScreen` | `/caretaker/qr` | Generate printable QR codes |
| `CaretakerMARChartScreen` | `/caretaker/mar-chart/:patientId` | 28-day eMAR chart + PDF export |

### Patient Screens (`screens/patient/`)

| Screen | Path | Description |
|--------|------|-------------|
| `PatientHomeScreen` | `/patient/home` | Next dose, confirm button, snooze |
| `PatientScheduleScreen` | `/patient/schedule` | Full day schedule |

### Family Screens (`screens/family/`)

| Screen | Path | Description |
|--------|------|-------------|
| `FamilyDashboard` | `/family/dashboard` | Linked patients, compliance, notifications |
| `FamilyActivityScreen` | `/family/activity/:patientId` | History + Calendar tabs |

---

## API Service (`services/api.ts`)

A singleton `api` object wraps all backend calls. It automatically attaches the JWT token from `useAuthStore`.

Key method groups:
- **Auth:** `login()`, `register()`, `refreshToken()`, `logout()`
- **Patients:** `getPatients()`, `getPatient()`, `getPatientCompliance()`
- **Schedules:** `getSchedules()`
- **Verification:** `verifyQR()`, `confirmDose()`, `getAdherenceHistory()`
- **Notifications:** `getNotifications()`, `markNotificationRead()`
- **Admin:** `getAdminStats()`, `getAdminPatients()`, `createAdminPatient()`, `updateAdminPatient()`, `deactivateAdminPatient()`, `addAdminMedication()`, `removeAdminMedication()`, `getAdminCaretakers()`, `getAdminFamily()`, `createAdminUser()`, `getAdminLinkRequests()`, `approveAdminLinkRequest()`, `rejectAdminLinkRequest()`, `getAdminPrescriptions()`, `uploadAdminPrescription()`

---

## Auth Store (`store/authStore.ts`)

Zustand store persisted to `localStorage`. Key fields:

```ts
{
  user: User | null,
  token: string | null,
  refreshToken: string | null,
  isAdmin: boolean,
  isCaretaker: boolean,
  isPatient: boolean,
  isFamily: boolean,
}
```

---

## MAR Chart & PDF Export

The eMAR (Medication Administration Record) chart is available in:
1. `AdminPatientDetailScreen` — inline in the eMAR tab (loads on first tab open)
2. `CaretakerMARChartScreen` — dedicated full-screen view

**Chart features:**
- 28-day date columns with day-of-week headers
- Time-of-day sections: PRE-MORNING, MORNING, NOON, AFTERNOON, EVENING (color-coded)
- X marks for verified/administered doses
- Today's column highlighted in blue
- Patient info header (name, DOB, address, allergies in red, MRN, condition)

**PDF export (`utils/generateMarPdf.ts`):**
- Uses `jspdf` + `jspdf-autotable`
- On web: triggers browser download via `doc.save()`
- On Android: writes base64 PDF to `Documents/` folder via `@capacitor/filesystem`; shows alert with filename

---

## Connecting to the Backend

The API URL is set in `apps/mobile/.env`:

```
VITE_API_URL=https://securedose-api.securedose.workers.dev
```

If not set, defaults to `http://localhost:8787` for local development.

---

## Building the Android APK

Run these steps from **Windows CMD** (not WSL — Android build tools are Windows `.exe` binaries):

```cmd
# Step 1: Build web assets
cd C:\Users\eunick\Documents\SecureDose
npm run build --workspace=apps/mobile

# Step 2: Sync to Android project
cd apps\mobile
npx cap sync android

# Step 3: Build APK
set "JAVA_HOME=C:\Users\eunick\.jdks\ms-17.0.17"
cd android
gradlew.bat assembleDebug
```

Output APK: `apps/mobile/android/app/build/outputs/apk/debug/app-debug.apk`

---

## Capacitor Native Features

| Plugin | Use |
|--------|-----|
| `@capacitor-community/barcode-scanner` | QR code scanning (opens native camera) |
| `@capacitor/local-notifications` | Medication reminder alerts |
| `@capacitor/filesystem` | Write PDF files to Android Documents folder |

> `@capacitor/share` is **not installed**. Do not import it — it will crash the entire Vite bundle.

---

## Local Development

```bash
# Start mobile web dev server (browser)
npm run mobile:dev
# Opens at http://localhost:5173

# Start with backend dev
npm run dev  # starts all apps via Turborepo
```
