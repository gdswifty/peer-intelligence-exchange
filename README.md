# Peer Intelligence Exchange — RAISE Paris 2026

WEKA booth activation for RAISE Summit (July 8–9 2026, Le Carrousel du Louvre, Paris).
A peer-sentiment survey that produces a live "read of the room" on the booth screen and a
post-event Peer Intelligence Report emailed to participants (which doubles as lead capture).

Aggregate responses are anonymous. Contact details are captured separately and are never
readable by the public anon key — only through the password-gated admin panel.


## Pages

| File | Role | URL |
|------|------|-----|
| `index.html` | iPad **kiosk** — the survey visitors take | https://gdswifty.github.io/peer-intelligence-exchange/ |
| `screen.html` | 55" TV **consensus board** — live read of the room | https://gdswifty.github.io/peer-intelligence-exchange/screen.html |
| `admin.html` | Password-gated **admin panel** — entries, funnel, CSV export | https://gdswifty.github.io/peer-intelligence-exchange/admin.html |

The booth **promo loop** (screen-recorded to video, not hosted here) lives in the working
folder as `raise-booth-loop-promo.html`.

## Flow (kiosk)

attract → how it works → 4 questions → contact details → gift ask (yes/no) →
gift select + shipping address + consent → thank-you (live room read)

- Each question: pick one option, optional texture chips, optional open text.
- Data is written to Supabase progressively via a single `save_response` RPC (answers →
  contact → finalize), so a partial completion still lands as a row.
- Kiosk resets to the attract screen after 90s idle.

## Backend — Supabase

- Project URL: `https://olbswfxmskkcshcirnxu.supabase.co`
- Table: `public.raise_responses` (one row per participant, client-generated `id`)
- **All writes go through the `save_response(payload jsonb)` RPC** (SECURITY DEFINER upsert).
  The anon role has INSERT/UPDATE but **no SELECT** — so PII is unreadable with the public key.
  A direct anon PATCH silently matches zero rows without a SELECT policy, which is why every
  write is funnelled through the RPC instead.
- Board reads the `room_read` view (counts only — no PII, excludes custom "other" answers and
  admin-excluded rows).
- Admin reads via `admin_dump(pw)` and can soft-exclude (`admin_set_excluded`) or hard-delete
  (`admin_delete`) rows. Password is checked **server-side inside the functions** — it is not
  in the page source. It is held in memory only for the session.
- Full schema, RLS, views, and functions are in `supabase-setup.sql`.

Timestamps are stored UTC; the admin panel and CSV display them in **Europe/Paris** time.

## Deploy

GitHub Pages from `main` / root. To publish: replace `index.html`, `screen.html`,
`admin.html` (and `supabase-setup.sql` / this README if changed), commit, wait for the Pages
build (amber dot ≈ 1–2 min), then hard-refresh (Cmd/Ctrl+Shift+R) to clear cache.

## Post-event

Freeze writes by revoking anon execute on the RPC (commented block at the bottom of
`supabase-setup.sql`), then export the final entries from the admin panel.
