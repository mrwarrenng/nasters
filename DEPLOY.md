# The WJZ Nebrasters — Deployment Guide

Deploy your skins game tracker so Warren, Zac, and Jon can all see live updates on their own phones during the trip.

Two services, both free tier, ~20 minutes total.

---

## Part 1 — Firebase Firestore (the database)

### 1.1 Create the project

1. Go to **https://console.firebase.google.com** and sign in with a Google account.
2. Click **Add project** (or **Create a project**).
3. Name it `wjz-nasters` (or whatever you like), then **Continue**.
4. **Google Analytics** is optional — you can turn it off; it's not needed here.
5. Click **Create project** and wait ~30 seconds for it to provision, then **Continue**.

### 1.2 Register a Web app and grab the config

1. On the project overview page, click the **web icon** (`</>`) under "Get started by adding Firebase to your app."
2. Give it a nickname like `wjz-nasters-web`. **Do not** check "Firebase Hosting" (we use Netlify). Click **Register app**.
3. Firebase shows a `firebaseConfig` object. Copy it into a notes app for Part 2 — it looks like:

```js
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "wjz-nasters.firebaseapp.com",
  projectId: "wjz-nasters",
  storageBucket: "wjz-nasters.appspot.com",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abcdef123456"
};
```

> These web config values are **safe to put in client-side code** — Firebase is designed this way. Access is controlled by the Firestore security rules you set in the next step, not by hiding these values.

### 1.3 Create the Firestore database

1. In the left sidebar, click **Build → Firestore Database**, then **Create database**.
2. Pick a location closest to South Dakota (e.g. **nam5 (United States)** or **us-central1**). This can't be changed later.
3. Start in **production mode** (we'll paste explicit rules next). Click **Create**.

### 1.4 Set the security rules

1. In Firestore, open the **Rules** tab and replace everything with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // The Nebrasters: a single shared game document per room.
    // Anyone who has the app URL + web config can read/write.
    // The obscure Netlify URL is the practical secret.
    match /nasters_state/{roomId} {
      allow read, write: if true;
    }
  }
}
```

2. Click **Publish**.

> This mirrors the app's original "anyone with the link can play" model. Want it locked down harder? See the note at the end of Troubleshooting.

---

## Part 2 — Connect the app

### 2.1 Easiest: paste it in the app (no code)

1. Open the app and tap **Share / Sync** in the header.
2. In the **⚡ Live Sync** box, paste the whole `firebaseConfig` you copied in Part 1.2 and tap **Connect Live Sync**.
3. The app reloads and shows **● Connected**. Done — your current game is pushed up automatically.

This saves the config on your device. It's the quickest way to get syncing.

### 2.2 To make the shared link work for everyone (recommended)

If you want Zac and Jon to just open the URL and be connected — without each pasting the config — bake it into the site once:

Open `index.html`, search for `BAKED_FIREBASE_CONFIG`, and fill in the six values:

```js
const BAKED_FIREBASE_CONFIG = {
  apiKey: 'AIzaSy...',
  authDomain: 'wjz-nasters.firebaseapp.com',
  projectId: 'wjz-nasters',
  storageBucket: 'wjz-nasters.appspot.com',
  messagingSenderId: '1234567890',
  appId: '1:1234567890:web:abcdef123456'
};
```

Only `apiKey` and `projectId` are strictly required, but paste all six. Commit and push — Netlify redeploys and everyone on the link is synced. (Or just send the config to your coding assistant and have it wired in for you.)

> **Tip:** Want a separate game for a future trip (Costa Rica 2027, etc)? Change `ROOM_ID` (search for it in `index.html`) to a different string. All three of you must use the same `ROOM_ID` to share the same game.

---

## Part 3 — Netlify (the hosting)

This repo is set up for **continuous deployment from GitHub** — connect it once and every push publishes automatically. The app already lives at `index.html` (Netlify's expected entry point) and `netlify.toml` tells Netlify to serve the repo root with no build step.

### 3.1 Connect the repo

1. Go to **https://app.netlify.com** and sign in (use GitHub for the smoothest link).
2. Click **Add new site** → **Import an existing project**.
3. Choose **GitHub**, authorize Netlify if prompted, then pick the **`nasters`** repository.
4. Build settings auto-populate from `netlify.toml` (publish directory `.`, no build command). Set the **branch to deploy** to the branch you push to.
5. Click **Deploy site**. Netlify builds and gives you a random URL like `https://merry-dolphin-abc123.netlify.app`.
6. Click **Site configuration** → **Change site name** to give it a memorable URL (e.g., `wjz-nasters`). Now it's `https://wjz-nasters.netlify.app`.

> Every push to the connected branch now redeploys automatically — no manual uploads.

### 3.2 Test it

1. Open the URL on your phone.
2. The masthead should show **● Synced** in the top-right (green-ish).
3. Add a test score; open the URL on another device and confirm it appears within a second.

---

## Part 4 — Share with Zac and Jon

Text them the Netlify URL. That's it. They open it, the app loads the current state from Firestore, and any score they enter shows up on your phone within a second.

If you want a vanity URL like `wjznasters.com`, buy it from Cloudflare/Namecheap (~$10/year) and point it at Netlify per their docs.

---

## Updating the app later

Made a code change? Commit it and push to the connected branch — Netlify rebuilds and publishes within seconds. Same URL, new code. No database disruption.

---

## Troubleshooting

**Status shows "● Error" or stays on "○ Local only":**
- Check that `apiKey` and `projectId` are pasted correctly with no extra quotes or spaces.
- Open browser console (Cmd-Opt-J on Mac Chrome, Cmd-Opt-I on Safari → Console) and look for red errors. A `permission-denied` error means the security rules from Part 1.4 weren't published.
- Confirm the Firestore database exists (Firebase console → **Build → Firestore Database**).

**Realtime not working but writes are:**
- This is almost always the security rules. Firestore's live listener needs `read` access — confirm the Part 1.4 rules are published and include `allow read, write: if true;` for `nasters_state`.
- Check the browser console for a `Missing or insufficient permissions` message on the listener.

**Want to wipe everything and start over:**
In the Firebase console: **Build → Firestore Database → Data** tab, open the `nasters_state` collection, and delete the `wjz-nasters` document. Then reload the app.

**Want to back up a trip permanently:**
In **Firestore Database → Data**, open `nasters_state / wjz-nasters` and copy the `state` field (it's the full game as JSON text). Save it somewhere. You can paste it back later via the app's Share/Sync → Import flow.

**Want to lock it down harder than "anyone with the link":**
The rules in Part 1.4 let anyone with the URL read/write — fine for a private trip URL you only share with two friends. To restrict it, enable **Firebase Authentication** (e.g. anonymous or Google sign-in) and change the rule to `allow read, write: if request.auth != null;`. That's a larger change and requires adding sign-in to the app — ask if you want it.

---

## Cost reality check

- **Firebase (Spark) free tier:** 1 GiB stored, 50K document reads + 20K writes + 20K deletes **per day**. Your game is one small document and three phones; you'll never come close.
- **Netlify free tier:** 100 GB bandwidth/month, unlimited sites. A ~90 KB HTML page served to three phones is rounding error.

You're well under the free limits unless something goes very strange.
