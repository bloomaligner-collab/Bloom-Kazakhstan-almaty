# Bloom Raffle · KazDentExpo Eurasia 2026

A two-page web app for capturing leads at the booth via QR code:

- **`index.html`** — Public landing page. Visitors scan the QR, fill in name / email / specialization / clinic / country / phone, and get a raffle number (BL-001, BL-002, …).
- **`admin.html`** — Password-protected dashboard. Search, see live stats, draw a random winner, export everything to Excel.
- **`schema.sql`** — Supabase table + Row Level Security policies. Run once.

Tech stack: vanilla HTML + Supabase (Postgres + Auth) + SheetJS for the Excel export. No framework, no build step. Same pattern as the Bassoul-Heneine and payment-follow-up apps.

---

## Setup — 10 minutes total

### 1. Supabase (3 min)

1. Use an existing Bloom Supabase project, or create a new one at https://supabase.com.
2. Open **SQL Editor**, paste all of `schema.sql`, click **Run**. This creates the `raffle_entries` table and the security policies.
3. Open **Authentication > Users > Add user > Create new user**. Set the admin email + password. Tick **Auto Confirm User**. This is what you'll use to log into `admin.html`.
4. In **Project Settings > API**, copy:
   - `Project URL` (looks like `https://xxx.supabase.co`)
   - `anon public` key (a long string starting with `eyJ...`)

The anon key is safe to put in the public HTML because RLS limits it to inserting only.

### 2. Plug the Supabase config into the two HTML files (1 min)

Open `index.html` and `admin.html` and replace these two lines in each (top of the `<script>` block):

```js
const SUPABASE_URL = "https://YOUR-PROJECT.supabase.co";
const SUPABASE_ANON_KEY = "YOUR-ANON-KEY";
```

Same values in both files.

### 3. Deploy to GitHub Pages (3 min)

```bash
mkdir bloom-raffle && cd bloom-raffle
# copy index.html, admin.html, schema.sql, README.md here
git init
git add .
git commit -m "Bloom raffle — KazDentExpo"
git remote add origin git@github.com:IantradingSAL/bloom-raffle.git
git push -u origin main
```

Then on GitHub: **Settings > Pages > Source: main / root**. After a minute the public URL is live, e.g.:

- Public form: `https://iantradingsal.github.io/bloom-raffle/`
- Admin: `https://iantradingsal.github.io/bloom-raffle/admin.html`

### 4. Update the QR code on the back-wall artwork (1 min)

Send me the public URL once it's live and I'll regenerate the QR pointing at it, then re-export the back-wall PDF / SVG. Until then the QR on the printed artwork still points at `bloomaligner.fr`.

### 5. Test (2 min)

- Open the public URL on your phone, fill in the form, submit. You should see `BL-001`.
- Open `admin.html`, sign in with the admin email + password from step 1.3. You should see your test entry. Click **Export to Excel** — it should download `Bloom-Raffle-KazDentExpo-2026-XX-XX.xlsx`.
- Click **🎲 Draw a winner** to test the random pick.

You're done. The whole thing runs in the browser, costs nothing on Supabase's free tier (well under the 50k row / 500MB limit even with thousands of entries), and works for the full two days of the expo.

---

## What admin.html shows

- **Total entries · Countries · Today's entries · Specializations**
- **Search box** — instant filter across name, email, country, clinic, specialization, raffle number
- **📥 Export to Excel** — every row, all columns, ISO timestamps, named with today's date
- **🎲 Draw a winner** — picks one random entry uniformly at random, shows a modal with their raffle number, name, email, specialization. "Draw again" lets you re-roll if you want.
- **Refresh** — pulls latest entries (use this at the end of day 2 before drawing)

---

## After the expo

- Hit **📥 Export to Excel** one final time for your records.
- The data stays in Supabase — you can keep using it for follow-up campaigns. If you want it gone: Supabase Dashboard > Table Editor > raffle_entries > delete rows, or just drop the table.

---

## Notes / future tweaks (optional)

- **Prevent the same email entering twice**: already enforced — the `email` column has a unique index. If someone re-submits, the form looks up their existing number and shows it (so they don't lose their original entry).
- **Add more questions**: add the column in `raffle_entries` (Supabase Table Editor) AND the input in `index.html` AND the column in `admin.html`'s table + Excel mapping. Three small edits.
- **Custom domain**: GitHub Pages supports a custom domain like `raffle.bloomaligner.fr` — Settings > Pages > Custom domain.
- **Send "thanks" email automatically**: Supabase Edge Function on insert + your transactional email provider. Not wired up here, but easy to add.
