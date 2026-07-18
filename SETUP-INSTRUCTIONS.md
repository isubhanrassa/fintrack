# Turning Finance Tracker into a synced app — setup guide

This turns your single HTML file into a real "app" that installs on your phone and laptop, with data that syncs between them automatically. Everything below is free (Firebase's Spark plan, Google's free tier — no credit card required).

You have 4 files now: `finance-tracker.html`, `manifest.json`, `sw.js`, `icon-192.png`, `icon-512.png`. They all need to stay in the same folder.

## 1. Create a free Firebase project

1. Go to https://console.firebase.google.com and sign in with your Google account.
2. Click **Add project**, give it any name (e.g. "my-finance-tracker"), skip Google Analytics (not needed).
3. Once created, click the **</> (web)** icon to register a web app. Give it a nickname, skip hosting setup for now.
4. Firebase will show you a `firebaseConfig` object that looks like this:
   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "my-finance-tracker.firebaseapp.com",
     projectId: "my-finance-tracker",
     storageBucket: "my-finance-tracker.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abc123"
   };
   ```
   Copy this whole block.

## 2. Paste your config into the app

1. Open `finance-tracker.html` in a text editor.
2. Find this section near the top of the first `<script type="module">` block (search for `YOUR_API_KEY`):
   ```js
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     ...
   };
   ```
3. Replace it with the config you copied in step 1.
4. Save the file.

## 3. Turn on Authentication and Firestore

In the Firebase console, in the left sidebar:

1. **Build > Authentication** → click **Get started** → enable **Google** as a sign-in provider → Save.
2. **Build > Firestore Database** → click **Create database** → choose a region close to you → start in **production mode**.
3. Go to the **Rules** tab of Firestore and replace the rules with this, so each user can only read/write their own data:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /financeTrackers/{userId} {
         allow read, write: if request.auth != null && request.auth.uid == userId;
       }
     }
   }
   ```
   Click **Publish**.

## 4. Host it (so it has a real URL — required for install + login to work properly)

Easiest free option is **Firebase Hosting**:

1. Install the Firebase CLI (needs Node.js installed): `npm install -g firebase-tools`
2. In the folder with your 5 files, run: `firebase login`, then `firebase init hosting`
   - Select your project.
   - Public directory: `.` (current folder)
   - Configure as single-page app: **No**
   - Don't overwrite `finance-tracker.html` if asked.
3. Rename `finance-tracker.html` to `index.html` (or set it as your hosting entry point) so it loads at the root URL.
4. Run: `firebase deploy`
5. You'll get a URL like `https://my-finance-tracker.web.app` — that's your app's permanent address.

*(Alternative: GitHub Pages works too, and is also free — push these files to a public repo and enable Pages in repo settings.)*

## 5. Install it as an app

**On your phone (Android/iOS):**
- Open the URL in Chrome/Safari.
- Sign in with Google once.
- Tap the browser menu → **Add to Home Screen** (iOS) or you'll see an **Install app** banner (Android/Chrome).
- It now opens full-screen like a native app, with its own icon.

**On your laptop:**
- Open the URL in Chrome or Edge.
- Sign in with Google once.
- Click the **install icon** in the address bar (or menu → **Install Finance Tracker**).
- It opens in its own window, pinned to your taskbar/dock — no browser bar.

## 6. Using it

- Sign in with the **same Google account** on both devices.
- Any change you make (add a transaction, pay a bill, etc.) saves locally instantly and syncs to the cloud in the background.
- Open the app on the other device — it updates automatically, no manual export/import needed anymore.
- The old "Download Backup / Restore Backup" buttons are still there in Settings as an extra manual safety net, but you don't need them for day-to-day syncing anymore.

## Notes on cost

Firebase's free Spark plan includes 50,000 document reads/day and 20,000 writes/day on Firestore, and unlimited free Google sign-ins. A personal finance tracker used by 1–2 people won't come close to those limits — this will stay free indefinitely under normal use.
