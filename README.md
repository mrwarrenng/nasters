# The WJZ Nasters 🏌️

A live skins-game tracker for Warren, Zac & Jon — "A Tradition Unlike Any Other."

A single, self-contained HTML page that tracks scores, handicaps, and skins in
real time across everyone's phones. State is synced through [Supabase](https://supabase.com)
and the site is hosted on [Netlify](https://www.netlify.com), auto-deploying from
this repository.

## How it's wired

- **`index.html`** — the entire app (HTML, CSS, JS in one file). Netlify serves
  this as the homepage.
- **`netlify.toml`** — Netlify config. No build step; the repo root is published
  as-is and every push to the connected branch triggers a deploy.
- **`DEPLOY.md`** — full step-by-step guide for standing up Supabase and Netlify.

## Deploying

This repo is set up for **continuous deployment**: connect it to a Netlify site
once, and thereafter every push to the deploy branch publishes automatically —
no more drag-and-drop. See [`DEPLOY.md`](./DEPLOY.md) for the one-time Supabase
and Netlify setup.

### Enabling live sync

Real-time sync stays off until Supabase credentials are supplied. Open
`index.html`, find the config block near the top of the `<script>` (search for
`SUPABASE_URL`), and paste in your Project URL and anon public key:

```js
const SUPABASE_URL = 'https://YOUR-PROJECT.supabase.co';
const SUPABASE_ANON_KEY = 'eyJ...';   // the "anon public" key
const ROOM_ID = 'wjz-nasters';        // shared room; same value for all players
```

The anon key is designed for browser use and is safe to commit — access is
constrained by the row-level-security policies in `DEPLOY.md`. Commit and push,
and Netlify redeploys with sync enabled. Until then the app runs in **local-only**
mode (state lives in the browser).

## Local preview

It's just a static file — open `index.html` in a browser, or serve the folder:

```sh
python3 -m http.server 8000   # then visit http://localhost:8000
```
