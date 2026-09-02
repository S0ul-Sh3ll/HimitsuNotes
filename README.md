# 🔒 Himitsu Notes (秘密ノート)

### A private, offline, hardware-encrypted notes app for Android — with per-note locks, an Obsidian-style tagging drawer, a built-in cipher wheel, a calculator, reminders and version history.

**Package / Application ID:** `com.s0ulsh3ll.himitsunotes`
**SDK:** `minSdk 26` (Android 8.0) · `targetSdk 35` (Android 15) · `compileSdk 35`
**Stack:** Kotlin · Jetpack Compose (Material 3) · Room · DataStore · AndroidKeyStore · Coil · Play Billing

---

## 🛡️ Security & privacy

- **Offline by design.** Notes, encryption and every core feature work with no network. The only declared network permission is `INTERNET`, used **solely** by Google Play Billing for the one-time Pro unlock — note content never leaves the device.
- **Hardware-backed AES-256-GCM at rest.** A master key is generated in the `AndroidKeyStore` (`CryptoEngine`). Titles, content, blocks, tables, code blocks, labels, checklists, attachments metadata and version history are all encrypted before they touch the database.
- **Two lock layers.**
  - **Global app lock** — a 6-digit master PIN (PBKDF2-HMAC-SHA256, `PinManager`) with optional biometric unlock and a configurable auto-lock timeout (`AppLockManager`).
  - **Per-note lock** — an individual 4-digit PIN on any single note.

---

## ✨ Features

### 📝 Block-based note editor
- Mixed **document blocks** on one note: rich text, interactive checkboxes, bullet lists, tables, code blocks, and inline images.
- **Rich text formatting:** bold, italic, underline, font size, font color, highlight, and links (span-based, `RichTextHelper` / `RichTextVisualTransformation`).
- **Interactive to-do checkboxes** — tap the checkbox to toggle done/undone with strikethrough; press Enter at the end of a checklist line to start the next item.
- **Tables** and **syntax-styled code blocks** as first-class blocks.
- **VS Code-style line-number gutter** (toggleable per note).
- **File attachments** and **inline images** (with a fullscreen image viewer).

### 🏷️ Labels & tags
- Obsidian-style `#tag` chips on each note, a left-drawer tag filter, and automatic clean-up of tags no longer used by any note.

### 🔑 Cipher Wheel
Transform note text with classic and modern ciphers: **ROT13/Caesar, Vigenère, Atbash, Base64, XOR, Morse, Reverse,** and **AES-256-GCM password encryption** (PBKDF2). Live preview, copy, and one-tap “apply to note”.

### 🧮 Built-in calculator
A safe expression evaluator (`CalculatorEvaluator`) reachable from the drawer and the editor — insert the result straight into a note.

### 🕒 Version history
Automatic snapshots of the last **5** revisions per note, with timestamps and one-tap **Restore** / **Copy**.

### ⏰ Reminders
Offline `AlarmManager` reminders with quick presets (later today, tomorrow morning/evening, next week), repeat cycles, notifications, and a dedicated Reminders screen.

### 🗂️ Organisation
- **Sort** by date modified, date created, title, or color.
- **Grid or list** home layout, pinned notes, multi-select batch actions (pin, color, trash).
- **Trash** with restore and permanent delete.
- **12 note colors** and **custom wallpapers** (home screen + per-note background, with a dim slider).

### ➕ Quick-create
A speed-dial FAB for the two note types: **Text / Markdown** and **Checklist**.

### 📤 Export & share
Export or share a note as **Markdown**, **Plain text**, **HTML**, or a **JSON backup**.

---

## 💎 Himitsu Pro (one-time unlock)

The free tier allows up to **10 active notes**. Creating the 11th prompts a one-time **Pro** unlock (₹511) that removes the cap forever.

- Powered by **Google Play Billing** (a non-consumable, managed in-app product). Because the entitlement is tied to the buyer’s Google account, Pro **restores automatically after a reinstall or on a new phone** — there’s also a manual **Restore purchase** button.
- A cached flag lets Pro keep working offline once Play has confirmed the purchase at least once.

> **To make Pro live, configure it in the Play Console** (see `PLAY_CONSOLE_SETUP.md`):
> create a Managed one-time product with ID `himitsu_pro_unlock`, price it at ₹511.00, and activate it. Product ID and the free limit live in `billing/ProManager.kt`.

---

## 🏗️ Building

```bash
export JAVA_HOME=/home/kali/jdk17
cd /home/kali/Projects/KeepNotes

# Unit tests (crypto, ciphers, markdown, checkbox toggle, version history, exporter)
./gradlew testDebugUnitTest

# Debug APK
./gradlew assembleDebug           # -> app/build/outputs/apk/debug/app-debug.apk

# Release App Bundle for Play (requires keystore.properties — see PLAY_CONSOLE_SETUP.md)
./gradlew bundleRelease           # -> app/build/outputs/bundle/release/app-release.aab
```

**Release signing:** if a `keystore.properties` is present at the project root, `bundleRelease`/`assembleRelease` sign with that upload key; otherwise they fall back to the debug key (for local builds). Never commit the keystore or `keystore.properties`.

---

## 📁 Project layout

```
app/src/main/java/com/s0ulsh3ll/himitsunotes/
├── billing/        ProManager — Play Billing Pro unlock
├── calculator/     Expression evaluator + dialog
├── cipher/         Cipher wheel engine + UI
├── crypto/         CryptoEngine, PinManager, AppLockManager
├── data/           Room entities/DAOs, NoteRepository (encrypt/decrypt), PreferencesRepository
├── export/         Markdown / plain / HTML / JSON exporters + share
├── markdown/       Rich-text spans, parser, visual transformations
├── media/          Media/file storage helper
├── model/          Note, NoteBlock, RichSpan, AppPreferences, …
├── reminder/       AlarmManager scheduling + receiver
└── ui/             Compose screens, components, view-models, theme, navigation
```

---

## 🔐 Permissions

| Permission | Why |
|---|---|
| `INTERNET` | Google Play Billing (Pro unlock) only |
| `USE_BIOMETRIC` | Optional biometric app unlock |
| `POST_NOTIFICATIONS` | Reminder notifications |
| `SCHEDULE_EXACT_ALARM` | Exact-time reminders |
| `VIBRATE` | Reminder haptics |
