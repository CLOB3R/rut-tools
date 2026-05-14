# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

RUT.tools is a static Chilean RUT (Rol Único Tributario) utility site. It validates and generates RUTs using the Módulo 11 algorithm. There is no build system — all files are plain HTML with inline CSS and JavaScript.

- **Hosting:** Vercel — `rut-tools.vercel.app` (auto-deploy on push to `main`, ~30s)
- **Repo:** github.com/CLOB3R/rut-tools
- **Domain:** `rutificado.com` (Namecheap → Vercel nameservers, DNS propagating)
- **DB:** Supabase — `jcuhqhlbzgubnhhuqiqo.supabase.co`

## Development

No build step. Open any `.html` file directly in a browser, or serve locally:

```bash
python3 -m http.server 8080
# or
npx serve .
```

Deploying: push to `main` → Vercel auto-deploys.

## Architecture

All logic is self-contained within each HTML file — no separate `.js` or `.css` files. CSS variables define the design system (`--bg`, `--surface`, `--accent`, `--green`, etc.) and are duplicated across pages since there is no shared stylesheet.

**Pages:**
- `index.html` — Main tool: RUT validator + generator, pricing, user login, SEO content
- `otras-herramientas.html` — Bulk Excel RUT processor (SheetJS CDN), company RUT validator, RUT formatter
- `panel.html` — Admin dashboard at `/panel.html` (Supabase auth + PIN-gated)
- `contacto.html` — Contact page (opens pre-filled WhatsApp)
- `privacidad.html` — Privacy policy

**External dependencies (CDN only):**
- Supabase JS v2 — auth and database
- SheetJS `xlsx.full.min.js` — Excel import/export in `otras-herramientas.html`
- Google Fonts — Sora, DM Mono, Share Tech Mono
- Google Analytics — `G-HY2JWNDTDX`

## Supabase schema

| Table | Description | RLS |
|-------|-------------|-----|
| `codigos` | Access codes. Cols: `id`, `codigo`, `plan`, `val_limit`, `gen_limit`, `activo`, `fecha_creacion`, `fecha_expiracion`, `veces_activado` | SELECT public, rest requires auth |
| `activaciones` | Activation log. Cols: `id`, `codigo`, `fecha` | Auth only |
| `admin_pin` | Admin PIN. Cols: `user_id`, `pin` | Owner only |
| `usuario_planes` | Per-user plans. Cols: `id`, `user_id`, `plan`, `val_limit`, `gen_limit`, `activo` | RLS disabled |

## Business logic

**Freemium model — free tier (client-side only):**
- Validator: 10 uses/day tracked in `localStorage` (`_vu` count, `_vd` date)
- Generator: 3 phases of 10 RUTs/day (`_gp` phase 0–2, `_gd` date); phases 1 and 2 require watching an ad modal

**Plan activation — two sources:**
1. **Access codes** (format: `PRO-XXXX`, `DEV-XXXX`) — entered manually by the user, validated against `codigos` table, cached in `localStorage` under `_cd_<code>`
2. **User login** — Supabase auth user; plan loaded from `usuario_planes` table, cached under `_cd_user_<UUID>`

`activeCodes` is a global JS array of currently active code strings. On page load, each code is re-verified against Supabase — if deleted/disabled, access is immediately revoked.

When a plan is active:
- Validator switches from daily-limit bar to plan's `val_limit`
- Generator switches from the 3-phase ad-gated flow (`gen-free-ui`) to a free-input UI (`gen-plan-ui`) with the plan's `gen_limit`
- Excel upload zones unlock

**RUT validation** runs entirely client-side. Módulo 11: multiply digits right-to-left by `2,3,4,5,6,7` (repeating), sum products, compute `11 - (sum % 11)`. Result `10` → `K`, result `11` → `0`.

## User login (index.html)

- Nav icon: red (logged out) → green with initial (logged in)
- If `email === 'lisandrodvicente2000@gmail.com'` → shows "Panel de creador" button
- Plan loaded from `usuario_planes` on login
- Plan can also be activated independently with an access code

## Admin panel (panel.html)

Two-step auth:
1. Supabase email + password (clean UI)
2. 4-digit PIN (red terminal-style UI, max 3 attempts)

Functions: dashboard stats, create/toggle/delete codes, view activation log, manage user plans, read beta comments.

## Payment flow (manual)

1. Customer contacts via WhatsApp (+56 9 9314 9165)
2. Admin creates a code in `panel.html`
3. Customer enters the code on the site → plan activates instantly
