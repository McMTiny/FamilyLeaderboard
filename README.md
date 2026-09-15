# Game Night Tracker

A single-page, mobile-first web app for tracking wins, losses, buy-ins, and
gifted pots across game nights (Poker, Mahjong, Uno, and anything else you add).

Everything lives in `index.html` — no build step, no dependencies, no server.
Data is stored locally in each visitor's browser (`localStorage`).

## Local preview

Just open `index.html` in a browser, or run a tiny local server:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Deploying — GitHub + Netlify (continuous deployment)

Once this is set up, **every time you edit `index.html` and push to GitHub,
Netlify automatically rebuilds and redeploys the live site** — no manual
redeploy step, ever.

### 1. Push this folder to GitHub

```bash
cd game-night-site
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<your-username>/<your-repo>.git
git push -u origin main
```

(If you don't have a repo yet: go to github.com → New repository → don't
initialize with a README since you already have one → copy the URL it gives
you into the `git remote add` command above.)

### 2. Connect Netlify to that repo

1. Go to [app.netlify.com](https://app.netlify.com) and log in (you can sign
   in directly with your GitHub account).
2. Click **Add new site → Import an existing project**.
3. Choose **GitHub**, authorize Netlify if prompted, and select this repo.
4. Build settings — since this is a plain HTML site, leave:
   - **Build command:** empty
   - **Publish directory:** `.` (the repo root, where `index.html` lives)
   (The included `netlify.toml` already sets these for you, so Netlify
   should detect them automatically.)
5. Click **Deploy site**. Netlify gives you a live URL right away, something
   like `random-name-123.netlify.app`.

### 3. That's it — continuous deployment is now on by default

Netlify automatically watches the branch you deployed (usually `main`).
From now on:

```bash
# make changes to index.html
git add .
git commit -m "Add a new stat card"
git push
```

...and Netlify rebuilds and redeploys within seconds, with zero extra
configuration. You'll see each deploy listed under the site's **Deploys** tab,
along with build logs if anything ever fails.

### 4. Optional: custom domain

In the site's Netlify dashboard: **Domain settings → Add a custom domain**.
Netlify provides free HTTPS automatically via Let's Encrypt.

### 5. Optional: deploy previews for pull requests

This is on by default once the repo is connected — if you ever open a pull
request instead of pushing straight to `main`, Netlify will build a preview
URL for that PR automatically, so you can check changes before merging.

## Live sync across devices (private to your group)

By default the app stores everything in `localStorage` — fine for one
device, but each person sees their own separate copy. To get a **live,
shared board that updates instantly across everyone's phones and laptops**,
without making it publicly discoverable, wire it up to a free Firebase
Realtime Database:

### 1. Create a free Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com) → **Add project** → give it any name → you can skip Google Analytics.
2. In the left sidebar: **Build → Realtime Database → Create Database**.
   - Choose any region.
   - Start in **test mode** for now (we'll lock it down in step 3).
3. In the left sidebar: **Project settings** (gear icon) → scroll to **Your apps** → click the **</> (Web)** icon → register an app (any nickname, no need for Firebase Hosting) → it will show you a `firebaseConfig` object like:

   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "your-project.firebaseapp.com",
     databaseURL: "https://your-project-default-rtdb.firebaseio.com",
     projectId: "your-project",
     ...
   };
   ```

### 2. Paste your config into `index.html`

Open `index.html`, find this block near the top of the `<script>` tag:

```js
const FIREBASE_CONFIG = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  databaseURL: "https://YOUR_PROJECT-default-rtdb.firebaseio.com",
  projectId: "YOUR_PROJECT",
};
```

Replace the four placeholder values with the ones from your `firebaseConfig`.
Commit and push — Netlify redeploys automatically (see above), and the app
now syncs live.

### 3. Lock the database down (so it's not wide open)

Back in Firebase console → **Realtime Database → Rules**, replace the
default test-mode rules with:

```json
{
  "rules": {
    "rooms": {
      "$room": {
        ".read": true,
        ".write": true
      }
    }
  }
}
```

This restricts reads/writes to the `rooms/` path only — the same shape the
app already uses. Each "board" lives under a random room code
(`rooms/<code>`), and that code only exists in the URL you share — it's
never listed or guessable, similar to how a Google Doc "anyone with the
link" share works. It's not bulletproof security, but it means nobody
stumbles onto your group's data by accident. If you want real
authentication (so only signed-in people can read/write), that's a further
step using Firebase Auth — ask if you'd like that added.

### How it works day-to-day

- The first time the app loads, it generates a short room code and appends
  it to the URL (`?room=ab12cd`).
- Open **Settings → Share & Sync** to copy the exact link — send that to
  your group. Anyone who opens it lands in the same room and sees the same
  live data.
- The green **"Live sync"** dot in the header confirms it's connected; a
  gray **"Local only"** dot means Firebase isn't configured yet (or the
  device is offline — it'll reconnect automatically).
- Every change (a new game logged, a player added) pushes instantly to
  Firebase, and every other open device receives it in real time — no
  refresh needed.

## Sharing with your group

Once deployed (and Firebase configured per above), send everyone the same
link from **Settings → Share & Sync**. On a phone:
- **iPhone:** open the link in Safari → Share → "Add to Home Screen"
- **Android:** open the link in Chrome → ⋮ menu → "Add to Home screen"

Without the Firebase setup, the app still works fine — it just keeps each
device's data local instead of syncing it live.
