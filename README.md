# The WJZ Nebrasters 🏌️

A live skins-game tracker for Warren, Zac & Jon — "A Tradition Unlike Any Other."

A single, self-contained HTML page that tracks scores, handicaps, and skins in
real time across everyone's phones. State is synced through [Firebase Firestore](https://firebase.google.com/products/firestore)
and the site is hosted on [Netlify](https://www.netlify.com), auto-deploying from
this repository.

## How it's wired

- **`index.html`** — the entire app (HTML, CSS, JS in one file). Netlify serves
  this as the homepage.
- **`netlify.toml`** — Netlify config. No build step; the repo root is published
  as-is and every push to the connected branch triggers a deploy.
- **`DEPLOY.md`** — full step-by-step guide for standing up Firebase and Netlify.

## Deploying

This repo is set up for **continuous deployment**: connect it to a Netlify site
once, and thereafter every push to the deploy branch publishes automatically —
no more drag-and-drop. See [`DEPLOY.md`](./DEPLOY.md) for the one-time Firebase
and Netlify setup.

### Enabling live sync

Real-time sync stays off until Firebase credentials are supplied. Open
`index.html`, find the config block near the top of the `<script>` (search for
`firebaseConfig`), and paste in the web config from your Firebase project:

```js
const firebaseConfig = {
  apiKey: 'AIzaSy...',
  authDomain: 'wjz-nasters.firebaseapp.com',
  projectId: 'wjz-nasters',
  storageBucket: 'wjz-nasters.appspot.com',
  messagingSenderId: '1234567890',
  appId: '1:1234567890:web:abcdef123456'
};
const ROOM_ID = 'wjz-nasters';   // shared room; same value for all players
```

These web config values are designed for browser use and are safe to commit —
access is constrained by the Firestore security rules in `DEPLOY.md`. Commit and
push, and Netlify redeploys with sync enabled. Until then the app runs in
**local-only** mode (state lives in the browser).

## Local preview

It's just a static file — open `index.html` in a browser, or serve the folder:

```sh
python3 -m http.server 8000   # then visit http://localhost:8000
```
