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

## Sharing with your group

Once deployed, send everyone the Netlify URL. On a phone:
- **iPhone:** open the link in Safari → Share → "Add to Home Screen"
- **Android:** open the link in Chrome → ⋮ menu → "Add to Home screen"

Note: because data is stored per-browser (`localStorage`), each person will
see their own local copy, not a shared live leaderboard. If you want
everyone to see the same live data, the next step is wiring the app up to a
small hosted database (e.g. Supabase or Firebase) — ask if you'd like that
built in.
