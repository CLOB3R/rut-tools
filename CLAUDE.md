# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Stack

Static HTML/CSS/JS — no build system. All logic inline per file, no shared stylesheet (CSS vars duplicated across pages).

- **Deploy:** push to `main` → Vercel auto-deploys in ~30s
- **Domain:** `rutificado.com` ✓ funcionando (Namecheap A → 76.76.21.21, CNAME www → cname.vercel-dns.com)
- **DB:** Supabase `jcuhqhlbzgubnhhuqiqo.supabase.co`
- **Analytics:** `G-HY2JWNDTDX` (propiedad GA4, renombrar a "rutificado.com" en Google Analytics)
- **Search Console:** verificado, sitemap enviado (`https://rutificado.com/sitemap.xml`)
- **CDN deps:** Supabase JS v2, SheetJS `xlsx.full.min.js`, Google Fonts (Sora, DM Mono, Share Tech Mono)

## Pages

- `index.html` — página principal: validador + generador + pricing + login + textos SEO
- `otras-herramientas.html` — hub que enlaza a las páginas dedicadas
- `organizar-rut-excel.html` — organizador masivo de RUTs desde Excel
- `validar-rut-empresa.html` — validador de RUT de empresas
- `panel.html` — admin dashboard (Supabase auth + PIN 4 dígitos, máx 3 intentos)
- `contacto.html` / `privacidad.html` — páginas estáticas

## SEO actual

**Keyword principal:** "rutificador" (~5M búsquedas/mes en Chile). Implementada en:
- `<title>`: "Rutificador Chile Gratis – Valida y Genera RUTs Online | rutificado.com"
- `<h1>`: "Rutificador Chile / Gratis y Online"
- Meta description, og:title, schema name, textos del cuerpo

**Keywords secundarias activas:** rutificador chileno, rutificador gratis, rutificador online, validar RUT Chile, generador RUT Chile.

**NO apuntar a:** "buscar RUT por nombre" / "rutificador por nombre" — requieren base de datos del Registro Civil, el sitio no tiene esa funcionalidad.

## Supabase tables

| Table | Key columns |
|-------|-------------|
| `codigos` | `codigo`, `plan`, `val_limit`, `gen_limit`, `activo`, `fecha_expiracion`, `veces_activado` |
| `activaciones` | `codigo`, `fecha` |
| `admin_pin` | `user_id`, `pin` |
| `usuario_planes` | `user_id`, `plan`, `val_limit`, `gen_limit`, `activo` |

## Business logic

**Free tier (localStorage only):**
- Validator: 10/day → `_vu` (count), `_vd` (date)
- Generator: 3 phases × 10 RUTs/day → `_gp` (phase 0–2), `_gd` (date); phases 1–2 require ad modal

**Plan activation (two sources):**
1. Access code (`PRO-XXXX` / `DEV-XXXX`) → validated against `codigos`, cached as `_cd_<code>`
2. Login → plan from `usuario_planes`, cached as `_cd_user_<UUID>`

`activeCodes` global array re-verified on page load — deleted/disabled codes revoke access immediately.

**With active plan:** validator uses `val_limit`, generator uses `gen-plan-ui` with `gen_limit`, Excel zones unlock.

**RUT algorithm (Módulo 11):** multiply digits right-to-left by `2,3,4,5,6,7` (repeat), sum, `11-(sum%11)`. `10`→`K`, `11`→`0`.

**Admin:** `email === 'lisandrodvicente2000@gmail.com'` → shows "Panel de creador" link.

**Payment flow:** WhatsApp (+56 9 9314 9165) → admin creates code in `panel.html` → user enters code → plan activates.
