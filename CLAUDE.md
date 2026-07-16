# CLAUDE.md — Project instructions for Claude Code

## What this project is
A single-file web app ("The Scramble & Blind Shot") for the 2026 xBU GA Welcome Party on **22 Jul 2026**. Full requirements live in **SPEC.md** (the build brief) and **PLAN.md** (dated milestones + the owner's testing checklists). Read both before doing anything.

## The one rule that matters: milestone-gated delivery
- Implement **exactly one milestone (M1–M5) per instruction**, as scoped in SPEC.md §17 / PLAN.md. Never start the next milestone without the owner explicitly saying "approved" or "proceed".
- When a milestone is complete:
  1. Commit with message `M<n> complete — ready for testing` and push to `main`.
  2. Tell the owner it is deployed (Cloudflare Pages auto-deploys `main` in ~1 min).
  3. Print the heading **"YOUR TESTING CHECKLIST — M<n>"** followed by that milestone's check steps copied from PLAN.md, then STOP and wait.
- If anything in the milestone cannot be completed, say so plainly and list what's missing — do not silently descope. The agreed descope order (only if the owner invokes it) is in PLAN.md.
- **Code freeze: end of Mon 20 Jul 2026.** After freeze, refuse changes unless the owner says "show-stopper".

## Hard technical constraints (from SPEC.md — do not drift)
- **One deployed file:** everything lives in a single `index.html` — HTML + CSS + vanilla JS. No frameworks, no build step, no bundler, no extra pages. Hash-routed views: `#/team` (default), `#/board`, `#/host`.
- Libraries via CDN only: Firebase (Realtime Database), html5-qrcode, qrcode.js, canvas-confetti, SheetJS/xlsx (Excel question import), Google Fonts (Archivo Black + Plus Jakarta Sans).
- Shared state in **Firebase Realtime Database**; slot claims and card redemptions must use **transactions**; team session + device token in `localStorage`.
- Ask the owner for the Firebase web config at the start of M1 if not already in the file.
- All display copy is **theme-neutral English** (no basketball/rookie/draft language) and the event name/labels come from `/config` (admin Event tab), never hardcoded.
- Answers stored as salted hashes, not plain indices.
- Defaults: bonuses 30/20/10s · lockout 30s · Scramble 25 min · Blind Shot base 120s · team cap 5 · host PIN "1234" — all editable in the admin Rules tab.
- QR station text format: `GA-S07` (year-neutral). Printable sheets (station posters, power-up cards ID-only, Staff Master Sheet) are print-styled views inside the same page. Posters must not show the event name.

## Style tokens — "Dusk Expedition"
App background ink teal `#0E2A38` (`--bg-deep`); surfaces `#143647` (`--bg-raised`); hairlines `rgba(239,227,204,.14)` (`--line`); text sand `#EFE3CC` (`--text`, dim `rgba(239,227,204,.62)`); primary accent amber `#F5A942` (time, CTAs, highlights); secondary ember `#FF7A59` (alerts, wrong answers, destructive); Night Rush background `#081C26` (`--night`). Team accents by join order: amber `#F5A942` · coral `#FF7A59` · cyan `#45C4E0` · mint `#4ADE80` · lavender `#A78BFA`. Display type Bebas Neue (uppercase, big numbers), body Plus Jakarta Sans; every timer uses `font-variant-numeric: tabular-nums`. Radius 12–16px max, 1px hairlines over drop shadows, glows reserved for meaning (timers, beacons, live states). Board signature: the sky darkens dusk→night as the Time Rush clock runs; Night Rush = full night with amber basket beacons; results = dawn; confetti restricted to amber/ember/sand. Board designed for 1080p projection with hall-scale type. Print views (posters, cards, staff sheet) are NOT themed — white background, ink-friendly, Archivo Black display.

## Working style
- Keep the code readable and commented by section (ROUTER / TEAM VIEW / BOARD / HOST / FIREBASE / PRINT) — a non-developer owner maintains this after handover.
- Test in mobile viewport for `#/team`, desktop for `#/board` and `#/host`.
- Never commit secrets; the Firebase web config is public by design and lives in the file.
