# SecureDose User Guide

This guide explains how to use the SecureDose app. There are four roles: **Admin**, **Caretaker**, **Patient**, and **Family**.

---

## Why SecureDose

SecureDose helps families and care teams prevent missed doses and confirm medication intake:

- **Verified confirmations** — caretakers scan a QR code per medication for reliable tracking.
- **Family visibility** — family members see dose activity and notifications without calling anyone.
- **Admin control** — admins manage all patients, staff, and medications from one place.
- **eMAR chart** — a 28-day Medication Administration Record is always available to view or export as PDF.

---

## Test Accounts

All accounts use password: `TestPassword123!`

| Role | Email |
|------|-------|
| Admin | admin@test.com |
| Caretaker | sarah.caretaker@test.com |
| Caretaker | mike.caretaker@test.com |
| Patient | john.patient@test.com |
| Family | emma.family@test.com |
| Family | robert.family@test.com |

---

## Login

1. Open the app.
2. Enter your email and password.
3. Tap **Log In**.

> Roles are detected automatically from your account. You do not select a role at login.

**Sign Up** is available for Caretakers and Family members only. Patients and Admins are created by an Admin.

**Forgot password?** Contact your system administrator — there is no self-service password reset.

---

## Admin Workflow

The Admin role manages the entire system: patients, staff, medications, and users.

### Dashboard

Shows live counts:
- Total patients
- Total caretakers
- Total family members
- Total missed dosages
- Pending prescriptions

Bottom tab bar: **Dashboard**, **Patients**, **Caretakers**, **Family**, **More**

---

### Managing Patients

**Patients tab** → lists all patients with their assigned caretaker.

**Create a new patient:**
1. Tap **+ New**.
2. Fill in: Full Name (required), Date of Birth (required), Assign Caretaker (dropdown, required).
3. Optional: MRN, Address, Allergies, Condition.
4. Tap **Create Patient**.

**View/edit a patient:**
Tap **Edit Patient Info** on any patient card to open the 5-tab Patient Detail screen.

**Deactivate a patient:**
Tap **Deactivate Patient** → confirm. The patient is removed from the list.

---

### Patient Detail — 5 Tabs

#### Profile Tab
- View and edit: Full Name, Address, Allergies, Condition.
- Date of Birth is read-only.
- **Transfer Caretaker:** check "Transfer Caretaker" → select a new caretaker from the dropdown → Save Changes.

#### Medications Tab
- Lists all active medications with name, dosage, scheduled time, and route.
- **Add Medicine:** Tap **+ Add Medicine** → fill in Drug Name, Dosage, Route, Instructions, and Scheduled Time → Tap **Add**.
- **Remove:** Tap **Remove** on any medication card.

#### eMAR Tab (Medication Administration Record)
- Shows a 28-day grid with medications grouped by time of day.
- **X** = dose verified/administered.
- **Blue column** = today.
- Tap **Save as PDF** to export the chart. On Android the PDF is saved to your Documents folder.

#### Calendar Tab
- Monthly calendar view.
- Tap any date to see the scheduled medications for that day.

#### Prescriptions Tab
- Tap **Upload Prescription Image** to attach a photo of a prescription.
- All uploaded images are listed with upload date.

---

### Managing Caretakers

**Caretakers tab** → lists all caretakers with:
- Number of assigned patients
- Expandable patient list

---

### Managing Family Members

**Family tab** → lists all family members with their linked patient and relationship.

**Pending link requests:** Family members who requested access appear at the top.
- Tap **Approve** to grant access.
- Tap **Reject** to deny.

---

### Managing Users (Create Caretaker / Family Accounts)

**More tab → Manage Users**

1. Tap **+ Create User** (top right).
2. Select role: **Caretaker** or **Family**.
3. Fill in: Full Name, Email, Password (min 8 characters), Phone (optional).
4. Tap **Create User**.

The new user can immediately log in with the email and password you set.

> Admin accounts cannot be created through the app. Use the database directly.

---

## Caretaker Workflow

Caretakers manage assigned patients and verify medication doses.

### Dashboard

Shows all assigned patients with:
- Weekly compliance progress bar
- **Scan QR** — open QR scanner
- **View MAR** — open the 28-day eMAR chart for that patient

### Scan and Verify a Dose

1. Tap **Scan QR** for a patient.
2. Point the camera at the printed medication QR code.
3. The app confirms the dose with the drug name, dosage, route, and scheduled time.

Result: A verification event is recorded.

### eMAR Chart

Tap **View MAR** for any patient to open the 28-day Medication Administration Record.

- Medications are grouped by time of day (Pre-Morning, Morning, Noon, Afternoon, Evening).
- X marks show verified doses.
- Tap **Print / PDF** to export or print.

### Generate a QR Code

1. Tap **Generate QR**.
2. Select a patient and medication schedule.
3. Tap **Generate QR Code**.
4. Tap **Save QR** to store it.

Print the saved QR image and attach it to the medication container.

### Link a Family Member

1. On the dashboard, tap a patient.
2. Enter the family member's email and relationship.
3. Confirm and submit.

The family member can see dose activity after an Admin approves the request.

---

## Patient Workflow

### Home Screen

Shows the next scheduled dose:
- Medication name and dosage
- Scheduled time

Actions:
- **I Took My Medication** — confirms the dose.
- **Remind Me in 5 Minutes** — schedules a short reminder notification.

### Schedule

Tap **My Schedule** to see today's full medication list with times.

---

## Family Workflow

### Dashboard

Shows linked patients with:
- Weekly compliance progress bar
- Recent notification summary
- **View Activity** — open the activity log for a patient

### Activity Screen

Two tabs:

**History tab** — lists verification events:
- Medication name and dosage
- Caretaker who verified
- Date and time

**Calendar tab** — monthly calendar:
- Tap any date to see verification events for that day.

### Notifications

Notifications show:
- Patient name
- Medication and dosage
- Verified or missed status
- Timestamp

Tap a notification to mark it as read.

---

## QR Codes

QR codes are generated per medication schedule and are used for caretaker dose verification.

Recommended workflow:
1. Admin or Caretaker generates a QR code for each medication schedule.
2. Print and attach QR codes to medication containers or a daily chart.
3. Caretaker scans during each dose check.

This creates a reliable audit trail without extra manual steps.

---

## Troubleshooting

**"Failed to fetch" error:**
- Check your internet connection.
- Confirm the API URL is reachable: `https://securedose-api.securedose.workers.dev`

**Medications not showing after adding:**
- Each medication must have a scheduled time. If you added a medication without a time, remove it and re-add with a scheduled time.

**PDF not saving on Android:**
- The PDF is saved to your device's **Documents** folder.
- Use a file manager app to find `MAR_Chart_<PatientName>.pdf`.

**Login fails after creating a user:**
- Confirm the email has no typos.
- Password must be at least 8 characters.
- Contact Admin if the account was recently created and login still fails.

**QR scan camera not opening:**
- Grant camera permission to the SecureDose app in Android Settings → Apps → SecureDose → Permissions.
