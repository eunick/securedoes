# Debugging Log - SecureDose

This document tracks debugging cycles and fixes applied during development.

---

## Summary

| # | Issue | Status | Date |
|---|-------|--------|------|
| 1 | QR Scanner camera not working on Android | Fixed | 2026-02-04 |
| 2 | Notification sounds (.mov files) not playing | Fixed | 2026-02-04 |

**Total Debugging Cycles: 2**

---

## Cycle 1: QR Scanner Camera Not Working

### Problem
When pressing "Start Scan" button on the QR Scanner screen, the camera would not open on actual Android devices.

### Root Causes
1. **Missing CAMERA permission** - AndroidManifest.xml did not declare the camera permission
2. **Missing camera feature declaration** - Android requires `uses-feature` declaration
3. **Missing CSS for transparent WebView** - The `@capacitor-community/barcode-scanner` plugin works by making the WebView transparent to show the native camera behind it. The CSS styles were missing.
4. **Incomplete class toggling** - The scanner service only added classes to `body`, not `html` element

### Files Modified
- `apps/mobile/android/app/src/main/AndroidManifest.xml`
- `apps/mobile/src/index.css`
- `apps/mobile/src/services/qrScanner.ts`

### Fixes Applied

#### 1. Added Camera Permission (AndroidManifest.xml)
```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-feature android:name="android.hardware.camera" android:required="true" />
```

#### 2. Added Scanner CSS (index.css)
```css
body.scanner-active {
  background: transparent !important;
  --ion-background-color: transparent !important;
}

body.scanner-active #root {
  display: none !important;
}

html.scanner-active,
html.scanner-active body {
  background: transparent !important;
}
```

#### 3. Updated Scanner Service (qrScanner.ts)
Changed `prepareScanner()` and `stopScanner()` to toggle classes on both `html` and `body`:
```typescript
// prepareScanner()
document.documentElement.classList.add('scanner-active');
document.body.classList.add('scanner-active');

// stopScanner()
document.documentElement.classList.remove('scanner-active');
document.body.classList.remove('scanner-active');
```

### Verification Steps
1. Rebuild app: `npm run build && npx cap sync android`
2. Install on device from Android Studio
3. Grant camera permission when prompted (or manually in Settings)
4. Press "Start Scan" - camera should now open

---

## Cycle 2: Notification Sounds Not Playing

### Problem
User wanted to use custom notification sounds from `.mov` files:
- `PATIENT.mov` - Patient medication reminder
- `CAREGIVER1.mov` - Caregiver dose time reminder
- `CAREGIVER2.mov` - Caregiver missed dose alert

### Root Causes
1. **Wrong file format** - `.mov` is a video container format; Android notifications require audio formats (MP3, WAV, OGG)
2. **Missing raw resources directory** - Android sound files must be in `res/raw/`
3. **No differentiated sounds** - Notification service used single sound for all notification types

### Files Modified
- `apps/mobile/android/app/src/main/res/raw/` (created directory)
- `apps/mobile/src/services/notifications.ts`
- `apps/mobile/capacitor.config.ts`

### Fixes Applied

#### 1. Converted MOV to MP3
Used ffmpeg to extract audio from video files:
```bash
ffmpeg -i PATIENT.mov -vn -acodec libmp3lame -q:a 2 patient_reminder.mp3
ffmpeg -i CAREGIVER1.mov -vn -acodec libmp3lame -q:a 2 caregiver_dose.mp3
ffmpeg -i CAREGIVER2.mov -vn -acodec libmp3lame -q:a 2 caregiver_missed.mp3
```

#### 2. Placed Files in Android Resources
Files placed in: `apps/mobile/android/app/src/main/res/raw/`
- `patient_reminder.mp3` (58KB, 4.4s)
- `caregiver_dose.mp3` (81KB, 5.8s)
- `caregiver_missed.mp3` (99KB, 7.2s)

#### 3. Updated Notification Service
Added sound constants and type-specific functions:
```typescript
export const NotificationSounds = {
  PATIENT_REMINDER: 'patient_reminder.mp3',
  CAREGIVER_DOSE: 'caregiver_dose.mp3',
  CAREGIVER_MISSED: 'caregiver_missed.mp3',
} as const;

export type NotificationType = 'patient' | 'caregiver_dose' | 'caregiver_missed';
```

Added new functions:
- `sendCaregiverDoseReminder()` - For caregiver dose time alerts
- `sendCaregiverMissedDoseAlert()` - For missed medication alerts
- Updated all existing functions to accept `notificationType` parameter

#### 4. Updated Capacitor Config
Changed default sound in `capacitor.config.ts`:
```typescript
LocalNotifications: {
  sound: 'patient_reminder.mp3',
}
```

### Verification Steps
1. Rebuild app: `npm run build && npx cap sync android`
2. Install on device
3. Trigger each notification type to verify correct sound plays

---

## Rebuild Instructions

After any fixes, always rebuild and reinstall:

```bash
cd apps/mobile
npm run build
npx cap sync android
# Then run from Android Studio
```

---

## Lessons Learned

1. **Android permissions must be explicit** - Always check AndroidManifest.xml when native features don't work
2. **Capacitor barcode scanner requires transparent WebView** - CSS must hide the web content for camera to show through
3. **Audio format matters** - Android only supports specific audio formats for notifications (MP3, WAV, OGG - not MOV/MP4)
4. **Resource naming conventions** - Android resource files must be lowercase with underscores only
