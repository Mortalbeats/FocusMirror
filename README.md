# FocusMirror

AI-powered focus tracking, study techniques, brain training and a shared leaderboard — all in the browser.

```
focusmirror/
├── index.html          markup
├── css/styles.css      all styling
├── js/
│   ├── config.js       ← Supabase URL + anon key
│   ├── app.js          background, routing, XP, camera tracking, techniques
│   ├── braingym.js     hand-tracked finger exercises
│   ├── games.js        Relax Bar (Sudoku, Memory Match)
│   ├── landing.js      landing-page helpers
│   ├── auth.js         Supabase sign in / sign up
│   └── sync.js         cloud sync + real leaderboard
├── schema.sql          run this in Supabase once
└── serve.py            local dev server
```

---

## 1. Set up the database (once, ~3 min)

1. Open your project → **SQL Editor** → **New query**
2. Paste the whole of `schema.sql` → **Run**
3. Confirm it worked:

```sql
select tablename, rowsecurity from pg_tables
where schemaname='public' and tablename in ('profiles','user_stats','sessions');
```

All three rows must show `rowsecurity = true`.

### Email confirmation

Your project currently has **"Confirm email" ON**, so new users must click a link
before they can sign in. For testing, turn it off:

**Authentication → Sign In / Providers → Email → Confirm email → OFF**

Turn it back on before real users sign up.

---

## 2. Run locally

```bash
python3 serve.py
```

Opens `http://localhost:8000`. **Don't** open `index.html` directly — browsers
block camera access on `file://` URLs.

---

## 3. Deploy

Any static host works. Easiest:

**Netlify** — drag this folder onto [app.netlify.com/drop](https://app.netlify.com/drop). Live HTTPS URL in ~30 seconds.

**Vercel** — `npx vercel` in this folder, or connect a GitHub repo.

**Cloudflare Pages / GitHub Pages** — point them at this folder; there's no build step.

### After deploying

Add your live URL in Supabase → **Authentication → URL Configuration**:

- **Site URL**: `https://your-site.netlify.app`
- **Redirect URLs**: `https://your-site.netlify.app/**`

Otherwise confirmation emails will redirect to the wrong place.

---

## Is it safe to commit `config.js`?

**Yes.** The Supabase *anon* key is designed to be public — it ships in every
Supabase web app. Your data is protected by Row Level Security policies, not by
hiding the key.

**Never commit the `service_role` key.** It bypasses RLS entirely. It should
only ever live in server-side code, never in this folder.

---

## How syncing works

Local-first. `localStorage` stays the source of truth for the UI, so the app
never blocks on the network and works fully offline.

- **Signed out** → everything stays on the device (as before)
- **Signing in** → local and cloud progress merge, taking the *higher* of each
  stat, so no progress is lost either way
- **While signed in** → XP changes push up (debounced ~4s), completed sessions
  are logged individually, and a final flush runs on page close

The leaderboard reads a `leaderboard` view joining `profiles` and `user_stats`.

---

## Security note on the leaderboard

RLS guarantees a user can only write to **their own** row. It does not guarantee
the *value* is honest — the browser is untrusted, so someone could POST an
inflated XP for themselves. `schema.sql` includes a trigger that makes XP
monotonic and caps any single write at +500.

That's a deterrent, not real security. For a competitively trustworthy
leaderboard (school use, prizes), XP must be computed **server-side** from the
`sessions` table, with `user_stats` made read-only to clients.
"# FocusMirror" 
