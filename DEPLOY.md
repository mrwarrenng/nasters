# The WJZ Nasters — Deployment Guide

Deploy your skins game tracker so Warren, Zac, and Jon can all see live updates on their own phones during the trip.

Two services, both free tier, ~20 minutes total.

---

## Part 1 — Supabase (the database)

### 1.1 Create the project

1. Go to **https://supabase.com** and sign up (GitHub or email).
2. Click **New project**.
3. Name it `wjz-nasters` (or whatever you like).
4. Pick a strong database password (save it somewhere — you won't need it often, but losing it locks you out).
5. Region: pick **West US (N. California)** or whichever is closest to South Dakota.
6. Plan: **Free**.
7. Click **Create new project**. Wait ~2 minutes for it to provision.

### 1.2 Create the table

When the project's ready, click the **SQL Editor** icon in the left sidebar, then **New query**, and paste this:

```sql
-- The Nasters: single-document game state, real-time enabled

create table if not exists nasters_state (
  id text primary key,
  state jsonb not null,
  updated_at timestamptz default now()
);

alter table nasters_state enable row level security;

-- Anyone with the anon key can read/write. The Netlify URL is the secret.
create policy "public read"   on nasters_state for select using (true);
create policy "public insert" on nasters_state for insert with check (true);
create policy "public update" on nasters_state for update using (true);

-- Enable real-time broadcasts on this table
alter publication supabase_realtime add table nasters_state;
```

Click **Run** (or `Ctrl+Enter`). You should see "Success. No rows returned."

### 1.3 Grab the credentials

1. In the left sidebar, click the **gear icon** (Project Settings) → **API**.
2. Copy two things into a notes app for the next step:
   - **Project URL** (looks like `https://abcdefghij.supabase.co`)
   - **anon public** key (the long string starting with `eyJ...`)

> The anon key is safe to put in client-side code. It's designed for browser use and only allows the operations our policies permit.

---

## Part 2 — Configure the app

### 2.1 Paste credentials into the HTML

Open `index.html` in a text editor (VS Code, TextEdit, Notepad, anything). Search for `SUPABASE_URL` — you'll find this block near the top of the JavaScript:

```js
const SUPABASE_URL = '';         // e.g. 'https://abcdefgh.supabase.co'
const SUPABASE_ANON_KEY = '';    // e.g. 'eyJ...'  (the "anon public" key)
const ROOM_ID = 'wjz-nasters';   // shared room ID — change if you want a separate game
```

Paste your values between the quotes:

```js
const SUPABASE_URL = 'https://YOUR-PROJECT.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIs...';
const ROOM_ID = 'wjz-nasters';
```

Save the file.

> **Tip:** Want a separate game for a future trip (Costa Rica 2027, etc)? Change `ROOM_ID` to a different string. All three of you must use the same `ROOM_ID` to share the same game.

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

Text them the Netlify URL. That's it. They open it, the app loads the current state from Supabase, and any score they enter shows up on your phone within a second.

If you want a vanity URL like `wjznasters.com`, buy it from Cloudflare/Namecheap (~$10/year) and point it at Netlify per their docs.

---

## Updating the app later

Made a code change? Commit it and push to the connected branch — Netlify rebuilds and publishes within seconds. Same URL, new code. No database disruption.

---

## Troubleshooting

**Status shows "● Error" or stays on "○ Local only":**
- Check that `SUPABASE_URL` and `SUPABASE_ANON_KEY` are pasted correctly with no extra quotes or spaces.
- Open browser console (Cmd-Opt-J on Mac Chrome, Cmd-Opt-I on Safari → Console) and look for red errors.
- Confirm the SQL ran successfully in Supabase (check **Table Editor** — `nasters_state` should exist).

**Realtime not working but writes are:**
- Confirm the `alter publication supabase_realtime add table nasters_state;` line ran without error.
- In Supabase: **Database → Replication** — `nasters_state` should be listed under the realtime publication.

**Want to wipe everything and start over:**
In SQL Editor: `delete from nasters_state where id = 'wjz-nasters';` Then reload the app.

**Want to back up a trip permanently:**
In SQL Editor: `select state from nasters_state where id = 'wjz-nasters';` Copy the JSON and save it somewhere. You can paste it back in later via the Share/Sync → Import flow.

---

## Cost reality check

- **Supabase free tier:** 500 MB database, 50K monthly active users, 2 GB egress. Your game state is a few KB. You'll never come close.
- **Netlify free tier:** 100 GB bandwidth/month, unlimited sites. A 70 KB HTML page served to three phones is rounding error.

You're well under the free limits unless something goes very strange.
