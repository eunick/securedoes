# SecureDose User Guide

This guide explains how to use the SecureDose app UI and what each role gains from the system.

## Why SecureDose

SecureDose helps families and care teams prevent missed doses and confirm medication intake with a simple, fast workflow:

- Verified confirmations: caregivers scan a QR code and patients confirm in-app for reliable tracking.
- Family visibility: family members get timely notifications without calling or guessing.
- Simple roles: each user sees only what they need to act quickly.

## Roles at a Glance

- Caretaker: manages patients, verifies doses with QR scans, links family members.
- Patient: sees medication schedule and confirms taking a dose.
- Family: monitors dose activity and notifications.

## Login and Role Selection

1. Open the app.
2. Select your role: Caretaker, Patient, or Family.
3. Log in (or register if enabled).

## Caretaker Workflow

### Dashboard

The Caretaker Dashboard lists all assigned patients and quick actions.

- Scan QR: opens the scanner to verify a medication dose.
- Link Family: connect a family member to a patient by email.

### Scan and Verify a Dose

1. Tap "Scan QR".
2. Point the camera at the printed medication QR code.
3. Wait for “Verified!” confirmation.

Result: The verification event is recorded for the dose.

### Link a Family Member

1. Choose a patient.
2. Enter the family member’s email and relationship.
3. Confirm consent and submit.

Result: The family member can now see dose activity for that patient.

## Patient Workflow

### Home Screen

The Patient Home screen shows the next dose with time, medication name, and dosage.

- I Took My Medication: confirms the dose and notifies the system.
- Remind Me in 5 Minutes: schedules a short reminder notification.

### Schedule

Tap “My Schedule” (or “Progress”) to view today’s medication times.

## Family Workflow

### Dashboard

The Family Dashboard shows linked patients and quick actions:

- Notifications: view verified doses and missed dose alerts.
- Reports: placeholder for future analytics.

### Notifications

Each notification shows:

- Patient name
- Medication and dosage
- Verified by caretaker or missed dose status
- Timestamp

Tap any notification to mark it read.

## QR Codes

QR codes are generated per schedule and used for verification. The intended flow is:

1. Print the QR code for a medication schedule.
2. Caretaker scans during a dose check.
3. Patient confirms in the app if required.

This creates a reliable audit trail without extra manual steps.

## Tips

- Keep the backend running (or use the hosted API URL) so the app can log in and sync.
- For testing, use demo accounts and the sample QR code output script.

## Support

If the app says “Failed to fetch”:

- Confirm the API URL is reachable.
- Ensure JWT and database secrets are configured on the backend.
