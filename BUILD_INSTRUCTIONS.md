# LECS — Build Instructions (Self-Contained Version)

This project bundles a full Node.js server (your LECS backend, including
the database, sales, stock, and the automatic WiFi sync service) directly
inside the Android app. No Termux, no separate server, no other device
required — every phone/laptop runs its own complete copy.

There are two one-time steps that need an internet connection on your PC,
which is why they couldn't be done automatically for you.

## Step 1 — Install prerequisites

- **Android Studio** (developer.android.com/studio) — during setup, also
  install: **NDK (Side by side)** and **CMake**, via
  *Settings → Languages & Frameworks → Android SDK → SDK Tools tab*
  (check the boxes, they don't install by default).
- **Node.js** (nodejs.org) — only needed on your PC to run `npm install`
  once, not on the phone.

## Step 2 — Install the server's dependencies

Open a terminal in this project folder and run:

```
cd app/src/main/assets/nodejs-project
npm install --omit=dev
```

This downloads express, sql.js, and the other small libraries the server
uses, directly into `node_modules` inside that folder — which then gets
bundled into the APK. (`--omit=dev` skips nodemon, which you don't need
on the phone.)

## Step 3 — Download the Node.js mobile runtime

This is the actual Node.js engine, compiled to run inside an Android app.

1. Go to **https://github.com/nodejs-mobile/nodejs-mobile/releases**
2. Find the latest release with Android binaries (currently
   **nodejs-mobile-v18.20.4**) and download its Android zip asset.
3. Unzip it. Inside you'll find `bin/` and `include/` folders.
4. Copy `bin/` → `app/libnode/bin` in this project
   (end result: `app/libnode/bin/arm64-v8a/libnode.so`, etc.)
5. Copy `include/` → `app/libnode/include` in this project
   (end result: `app/libnode/include/node/node.h`)

## Step 4 — Open and build in Android Studio

1. Open Android Studio → **Open** → select this project's folder.
2. Let Gradle sync. The first sync will also configure the CMake/NDK
   native build — this can take a few minutes.
3. **Build → Build Bundle(s) / APK(s) → Build APK(s)**
4. Find the APK at `app/build/outputs/apk/debug/app-debug.apk`

## Step 5 — Install on each device

Copy `app-debug.apk` to each phone/laptop and install it (allow installs
from unknown sources when prompted, the first time). Each device runs its
own independent copy — first launch takes a few extra seconds while it
unpacks and starts its own local server for the first time.

**Default login on first launch:** username `admin`, password `admin123`
— you'll be asked to change it immediately.

## What happens automatically after that

- Every device works fully offline, with its own local data.
- On the same WiFi, devices automatically discover each other and push
  sales/stock changes to one another in the background — no button to
  press. See Settings → 🔄 Auto-Sync to check how many devices are
  currently visible.
- Across different networks, there's no automatic sync (see chat history
  for why) — the manual export/import API still exists under the hood at
  `/api/sync/export` and `/api/sync/import` if you ever want a "Send
  Now"-style button added for that case later.

## If the build fails

The most common issues:
- **CMake/NDK not found** — install them from the SDK Manager (see Step 1).
- **"libnode.so not found"** — double check Step 3's folder structure exactly.
- **npm install errors** — make sure you're running it inside
  `app/src/main/assets/nodejs-project`, not the project root.

Screenshot the exact error and send it over if you get stuck on any of
these — build errors are usually easy to diagnose from the message.

## Charts work fully offline

Dashboard and Reports charts no longer depend on any CDN or external
library — they use a small built-in renderer (`public/js/vendor/lecs-charts.js`)
bundled directly in the app. Nothing to download, nothing that can
break once the phone has no internet.
