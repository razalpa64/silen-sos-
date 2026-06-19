# Deploy Silent SOS+ to a permanent URL (Vercel + Neon or Supabase)

This turns the sandbox preview into a stable, always-on web app with its own
database. Total time: ~15 minutes. No credit card required for the free tiers.

You need three things:

1. A **GitHub** account (to host the code)
2. A **database** (Neon _or_ Supabase — pick one)
3. A **Vercel** account (to host and run the app)

---

## Step 1 — Put the code on GitHub

From this project folder on your computer:

```bash
git init
git add .
git commit -m "Silent SOS+ web app"
```

Then create an empty repo at https://github.com/new (e.g. `silent-sos-plus`)
and push:

```bash
git remote add origin https://github.com/YOUR_USERNAME/silent-sos-plus.git
git branch -M main
git push -u origin main
```

> If you're editing only inside this sandbox, download/export the project first,
> then run the commands above locally.

---

## Step 2 — Create a database

### Option A: Neon (recommended, simplest)

1. Go to https://neon.tech and sign up (free).
2. Create a new project. Pick a region close to you.
3. On the project dashboard, click **Connect** and copy the
   **Pooled connection** string. It looks like:
   ```
   postgresql://USER:PASSWORD@ep-xxxx-pooler.REGION.aws.neon.tech/DBNAME?sslmode=require
   ```
4. Keep this string — it's your `DATABASE_URL`.

### Option B: Supabase

1. Go to https://supabase.com and create a project (free). Save the DB password.
2. Go to **Project Settings → Database → Connection string → URI**.
3. Use the **Transaction pooler** connection (port `6543`). It looks like:
   ```
   postgresql://postgres.PROJECTREF:PASSWORD@aws-0-REGION.pooler.supabase.com:6543/postgres?sslmode=require
   ```
4. Replace `[YOUR-PASSWORD]` with the DB password you saved. This is your `DATABASE_URL`.

> The app auto-enables SSL for any non-local host, so the pooled strings above
> work as-is.

---

## Step 3 — Create the database tables

You have two easy ways. Do this once.

### Way 1 — Run the migration from your computer

```bash
# Put your production connection string in your shell, then push the schema:
DATABASE_URL="YOUR_PRODUCTION_CONNECTION_STRING" npx drizzle-kit push
```

### Way 2 — Paste SQL into the provider's SQL editor

Open `drizzle/0000_silent_sos_init.sql` in this project, copy its contents, and
paste it into:

- **Neon:** dashboard → **SQL Editor** → Run
- **Supabase:** dashboard → **SQL Editor** → New query → Run

Either way, you should now have 5 tables: `trusted_contacts`, `sos_alerts`,
`alert_deliveries`, `location_updates`, `app_settings`.

---

## Step 4 — Deploy on Vercel

1. Go to https://vercel.com and sign in with GitHub.
2. Click **Add New… → Project**, then **Import** your `silent-sos-plus` repo.
3. Vercel auto-detects Next.js — leave the build settings as default.
4. Before deploying, open **Environment Variables** and add:
   - **Name:** `DATABASE_URL`
   - **Value:** your production connection string from Step 2
   - Apply it to **Production**, **Preview**, and **Development**.
5. Click **Deploy**.

When the build finishes you'll get a permanent URL like
`https://silent-sos-plus.vercel.app`. That's your stable link — open it on your
phone and **Add to Home Screen** for an app-like experience.

---

## Step 5 — Verify it's live

- Visit `https://YOUR-APP.vercel.app/api/health` → should return `{"ok":true}`.
- Open the home page, add 2 trusted contacts, and fire a **Manual SOS** to
  confirm alerts log to your new database.

---

## Updating the app later

Any push to your GitHub `main` branch triggers an automatic redeploy on Vercel.
If you change the database schema (`src/db/schema.ts`), regenerate and re-apply:

```bash
npx drizzle-kit generate
DATABASE_URL="YOUR_PRODUCTION_CONNECTION_STRING" npx drizzle-kit push
```

---

## Reminder about limitations (web vs. native Android)

This is the **web** version. A browser cannot send real carrier SMS, run a true
always-on background service, intercept hardware buttons, or do offline
wake-word detection. SMS/push delivery is **simulated and logged**; shake
detection, the cancel window, geolocation, the alert log, and acknowledgement
links are fully functional. For real SMS and background behavior you need the
native Kotlin Android app built in Android Studio.
