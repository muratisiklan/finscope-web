# Landing page + waitlist

Pre-launch marketing site for the rebuilt app. Static, zero-infra (host on GitHub
Pages, Cloudflare Pages, Netlify, or `python -m http.server`). Signups land in the
`waitlist` table in your existing Supabase project.

## Files
- `index.html` — the page (premium UI + SEO head + waitlist form)
- `privacy.html`, `terms.html` — legal (carried over; review before launch)
- `robots.txt`, `sitemap.xml` — SEO
- `assets/` — screenshots. **These are the OLD app's screenshots — swap with new
  beta captures** (keep the same filenames or update the `<img src>` in `index.html`).

## How the waitlist works
The form POSTs to the `join_waitlist()` RPC (see
`../supabase/migrations/0004_waitlist.sql`) using the **publishable** Supabase key
(safe to ship publicly). The RPC is `security definer`, so it writes to a table the
public key otherwise can't touch — nobody can read or scrape the list, and
duplicate/existing emails are deduped silently (no "email already exists" leak).

**Before the form works, apply the migration** to your Supabase project:
```bash
supabase db push          # or run 0004_waitlist.sql in the SQL editor
```

### Export your signups
The table is service-role-only, so read it from the Supabase dashboard (Table
editor → `waitlist`) or SQL editor:
```sql
select email, source, created_at from waitlist order by created_at desc;
```
At launch, export the `email` column → send your announcement.

## Rebrand (currently neutral default "Portfoyo")
The displayed name lives in one place: the `BRAND` const in the `<script>` at the
bottom of `index.html` drives every `[data-brand]` slot. The SEO `<meta>` tags and
visible copy use the literal word "Portfoyo" — to fully rebrand (e.g. to FinScope):
```bash
# from landing/
sed -i '' 's/Portfoyo/FinScope/g' index.html
```
Then review the `<title>`/`og:` text and update the `mark` letter ("P") in the nav.

## Deploy (GitHub Pages → portfoyo.app)
1. Push this `landing/` content to a new repo (or a `gh-pages` branch).
2. Settings → Pages → deploy from that branch.
3. Add a `CNAME` file containing `www.portfoyo.app` (and configure DNS).
4. Set the canonical/OG/sitemap URLs in `index.html` + `sitemap.xml` if the final
   domain differs.

## Pre-launch checklist
- [ ] Apply `0004_waitlist.sql` and submit a test email; confirm the row appears.
- [ ] Replace `assets/*.jpg` with new beta screenshots.
- [ ] Decide the final brand and run the rebrand step if needed.
- [ ] Review `privacy.html` / `terms.html` for the connected (brokerage-sync) product.
- [ ] Confirm `og:image` renders (share the URL in a Slack/iMessage to preview).
