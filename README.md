# Code to QR

Single-screen Android app (Kotlin + Jetpack Compose):
scan one printed code with the camera, on-device OCR fills an editable text field, and a QR code is generated from that exact text. Manual entry works without the camera. Save the QR as a PNG.

- **Camera**: CameraX
- **OCR**: Google ML Kit Text Recognition (bundled Latin model, works offline)
- **QR**: ZXing core (black-on-white, error correction M, quiet zone included)
- **Saving**: MediaStore, to `Pictures/QRCodes/` (no storage permission needed)
- **Permissions**: `CAMERA` only. No internet permission, no accounts, nothing is uploaded.
- **Requires**: Android 10 (API 29) or newer

## Build the APK

The first build must be online (Gradle downloads the Android libraries). After that, the app itself needs no internet.

### Option A: Android Studio (easiest)
1. Install Android Studio (Koala or newer).
2. **File > Open** and choose this `CodeToQR` folder. Let Gradle sync finish.
3. **Build > Build Bundle(s) / APK(s) > Build APK(s)**.
4. APK: `app/build/outputs/apk/debug/app-debug.apk`

### Option B: Command line
Needs JDK 17+, the Android SDK (API 34 platform + build-tools), and Gradle 8.9+.
```
# point Gradle at your SDK once
echo "sdk.dir=/path/to/Android/Sdk" > local.properties

gradle assembleDebug
```
APK: `app/build/outputs/apk/debug/app-debug.apk`

(No Gradle wrapper is included. Android Studio doesn't need one.)

### Option C: GitHub Actions (no local setup)
1. Create a new GitHub repo and push this folder to it.
2. Open the **Actions** tab, run **Build APK** (it also runs on every push).
3. Download `CodeToQR-debug-apk` from the finished run and unzip it.

## Install on your phone
Copy `app-debug.apk` to the phone and open it (allow "install unknown apps" for your file manager or browser when asked), or with USB debugging on: `adb install app-debug.apk`.
This is a debug-signed APK, which is fine for personal use.

## How it behaves
- Point the white frame at one code. If the same best line is read twice in a row, it fills the text field and pauses scanning.
- Several lines visible? They show as chips under the camera, nearest to the frame centre first. Tap one to use it. Lines are never merged.
- While you focus or edit the field, scanning never overwrites your text. Tap **Scan again** to resume.
- The QR always matches the exact text in the field (nothing is trimmed, uppercased or reformatted) and updates as you type. **Generate QR** re-confirms it and reports empty or too-long input.
- **Save QR** writes a PNG to `Pictures/QRCodes/` and shows a success or error message.
- OCR only ignores obvious noise in the detected-line chips (fewer than 2 characters, or no letter/digit). There is no format, length or character restriction on the text itself.

## Project layout
```
app/src/main/java/com/example/codetoqr/
  MainActivity.kt    theme + entry point
  MainScreen.kt      the single screen, permission flow, scan/edit rules
  CameraScanner.kt   CameraX preview + ML Kit OCR analyzer
  QrGenerator.kt     text -> QR bitmap (ZXing)
  QrSaver.kt         bitmap -> PNG via MediaStore
```
