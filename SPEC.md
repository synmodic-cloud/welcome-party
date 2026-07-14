# 2026 xBU GA Welcome Party — Game App Specification
**"The Scramble & Blind Shot" companion app · v1.2 — 14 Jul 2026**
Event: 22 Jul 2026 · 14:00–17:40 · Sports hall · **30 GAs in 5 teams of 6**

**v1.2 changes:** configurable event branding (name / subtitle / challenge labels set in admin, displayed across all views) · theme-neutral UI copy — the basketball metaphor lives in facilitation, not the app · year-neutral QR namespace (`GA-S07`) so laminated posters are reusable across cohorts · continuity & reuse model (§1.1) · dated milestone plan (§17 + companion Development Plan doc).
**v1.1 changes:** 30 pax · name-only registration · one-device anti-cheat · power-ups claimed via staff mission (Staff Master Sheet) · Silence card · lockout 30 s · host PIN 1234 · English only · Cloudflare Pages.

---

## 1. Purpose & scope

One lightweight web app that runs the two linked team challenges: **The Scramble** (QR quiz treasure hunt that banks shooting time and collects power-up cards) and **Blind Shot** (blindfolded shooting where the banked time is spent). It serves three audiences from a single URL: the 5 team devices, the projected main-screen dashboard, and the L&D host console.

It is an **event tool**: no user accounts, no personal data, results exportable. The Emoji Passport (check-in / grouping) and the 2-year journey tracking (Padlet + form) are **separate tools and out of scope** — this app shares only the visual brand system.

**1.1 Continuity & reuse.** The app is designed to be reused for future cohorts and events. The model: one live game at a time → after the event, **export results (CSV) → reset game → rename the event in admin → swap the questions → reuse the same laminated posters.** Everything cohort-specific lives in the database (event name, labels, questions, codes); nothing cohort-specific is printed on the physical station posters or baked into the code. Admin's **Event tab** (§9) sets the event name, subtitle, and challenge labels, which display live on the team view header, board header, and results screen.

---

## 2. One page, three views

A single `index.html`. The view is chosen by the URL hash — no separate HTML files, no page swapping. All three views display the admin-set event name.

| View | URL | Device | Used by |
|---|---|---|---|
| **Team** | `.../#/team` (default) | 5 team phones | One nominated "team device" per team |
| **Board** | `.../#/board` | Laptop → projector | Whole room (main screen) |
| **Host** | `.../#/host` (PIN-gated, default **1234**) | L&D phone or laptop | Host / admin |

Printable materials (station QR posters, power-up cards, Staff Master Sheet) are also generated **inside the same page** via print-styled views — so the deployed artifact is exactly one file.

**URL hygiene:** the team-view QR/link is handed **only to the 5 nominated team keepers** at team formation — never projected to the room. This is part of the fair-play design (§4.6).

---

## 3. Game phases

The host moves the game through four phases; the current phase drives what every view displays.

| Phase | Team view shows | Board shows |
|---|---|---|
| **Setup** | Team registration (name) | Event-name splash + "teams joining" list |
| **Scramble** | Scan button · wallet · code entry | Live leaderboard + master countdown + event ticker |
| **Blind Shot** | "Wallet locked" summary + team's own result | Giant per-team countdown, basket tally, cards played, standings |
| **Results** | Final team result | Final ranking + confetti |

---

## 4. Team view (mobile)

**4.1 Registration (Setup phase).** Enter a team name, tap Join. No emoji pick; each team is auto-assigned one of the 5 brand accent colours by join order. Team count is capped (admin-set, default **5**). On creation, the team is **bound to that device** (§4.6); the app stores the identity in `localStorage`, so a refresh or phone lock does not lose the session.

**4.2 Game screen (Scramble phase).** Header shows the event name. Three elements, thumb-sized: a big **SCAN STATION** button (opens in-app camera scanner), the **team wallet** (banked seconds, power-up cards held, stations cleared count), and an **Enter Card Code** field for power-up cards. A small ticker mirrors the board feed.

**4.3 Scan → answer flow.** Scan a station QR → the app looks up that station's question → shows the question + 4 options → team taps an answer → **instant in-app marking**:
- Correct, slot available → "✅ Correct — 2nd to solve! +20s banked." Wallet updates live.
- Correct but 3 teams already cleared it → "Station closed — 3 teams got here first. Keep hunting!"
- Wrong → "❌ Not quite — this station is locked for you for 30s." (countdown shown; retry allowed after)
- Already cleared by this team → "You've already cleared this station."

**Camera fallback:** if camera permission fails, a manual "enter station number" input does the same lookup — the game never dies on a broken camera.

**4.4 Power-up code entry.** Typing a valid, unredeemed card code (revealed by staff after the mission — §6) adds that card to the wallet ("🎴 CLOSER LINE collected!"). Each code is **single-use** — first team to enter it owns it; a second attempt shows "already claimed."

**4.5 End of Scramble.** When the host ends the phase (or the clock hits zero), submissions lock and the team sees their final wallet: total banked time + cards, carried into Blind Shot.

**4.6 Fair play — one device per team.** Design goal: prevent members from multiplying coverage by all scanning in parallel on personal phones.
- **Inert QRs:** station QRs encode plain text (`GA-S07`), not a link — a personal camera app gets a meaningless string. Only the in-app scanner on a registered team device turns it into a question.
- **Device binding:** on team creation the app generates a random device token, stored in `localStorage` and on the team record. Every submission carries it. A second device attempting to join or act as that team is rejected: "This team is already active on another device."
- **Team cap:** with the cap at 5, extra devices can't register ghost teams either.
- **Host-controlled transfer:** if a team phone dies, the host taps **Release team device** in Live Control; only then can a replacement phone claim the team. No self-service rejoin — so recovery can't be abused as a backdoor.
- **Honest scope note:** console-level tampering by a determined developer is out of scope for a party game; the ticker and host dashboard make anomalies visible, and Live Control manual adjustments are the human override.

---

## 5. Scramble rules engine

| Rule | Behaviour | Default (admin-editable) |
|---|---|---|
| Questions | Multiple-choice, 4 options, 1 correct; each question bound 1:1 to a station QR | 15–20 questions |
| Podium slots | First **three** teams to answer a station correctly earn tiered bonuses, then the station closes | 3 slots |
| Tiered bonus | 1st / 2nd / 3rd correct team banks descending seconds | **30 / 20 / 10 s** |
| Wrong answer | Per-team lockout on that station; retry allowed after; does **not** consume a slot | **30 s** |
| One clear per team | A team can only earn from each station once | — |
| Team cap | Maximum registered teams | 5 |
| Master clock | Host starts the phase; board shows countdown; submissions auto-lock at zero (host can end early) | 25 min |
| Ticker | Board announces events ("Team Nova took 1st on Station 7!" / "Station 12 — one slot left!") | on |

**Simultaneous answers** are resolved atomically by database transaction — two near-simultaneous correct answers get slots in server order; there is exactly one 1st, 2nd, 3rd.

---

## 6. Power-up cards — find, earn, claim

Physical cards hidden around the hall. **The printed card does not carry the code.** The claim flow has a human checkpoint:

1. **Find** — team discovers a hidden card (card face: "✨ POWER-UP CARD ★ 3 — bring this to a referee to unlock").
2. **Earn** — a staff referee gives the team a quick micro-mission (≈20 seconds, from the Staff Master Sheet — e.g. whole-team synchronized 5 jumping jacks, a 10-second team cheer, spell "HKT" with bodies, human tunnel, 6-person high-five chain, group freeze-frame "victory pose", hum a song until someone guesses it, 10 passes without dropping).
3. **Claim** — on completion, the referee reveals that card's code; the team types it into the app → power-up lands in the wallet.

This adds a fun social beat, stops one lucky team hoovering cards without earning them, and needs **zero app changes** beyond the existing code entry — it's a printing + staffing change.

| Card | Effect | In-app logic | Suggested count |
|---|---|---|---|
| **Time Boost** | +30 s shooting time | Adds to Blind Shot total when armed | 2 |
| **Closer Line** | Shoot from a nearer line for one turn | Badge shown on board; physical effect | 2 |
| **Double Points** | Next basket counts ×2 | Host console gets a "+2" button while armed | 2 |
| **Silence** | Other teams may not distract during the next shot | Full-screen board banner "🤫 SILENCE — no distractions for this shot"; host referees the quiet | 2 |

Total set: **8 cards, 2 of each** (adjustable — confirm mix). Codes look like `GOLD-K7M3` — 4 characters from an unambiguous alphabet (no 0/O, 1/I/L) so they're easy to type under pressure.

---

## 7. Blind Shot module

**Per-team total time = base + banked Scramble time + Time Boost cards armed.** (Default base **120 s**, admin-set.)

**Host console flow:** select the shooting team → the app shows their computed total and cards held → host **arms** any cards the team chooses to play → **Start**. Controls: Start / Pause / Resume / End turn, a large **+1 basket** button, a **+2** button while Double Points is armed, and **Undo last**. On End turn, the result saves and the board updates standings.

**Card behaviour during a turn:** Time Boost adds to the clock before start; Closer Line and Silence show as prominent board badges/banners (physical/social effects, host-refereed); Double Points arms the +2 button for the next basket.

**Board during a turn:** giant countdown, team name in team colour, live basket tally, badges for cards in play — and the full-screen Silence banner when armed, so the whole room knows the rule is active.

**Ranking:** by baskets scored; proposed tiebreak = total bonus time earned in the Scramble (rewards the hunt), then a sudden-death free throw for drama. *(Please confirm the tiebreak.)*

---

## 8. Board view (projector)

Designed for 1080p projection: huge type, high contrast, readable from the far end of a sports hall. Header shows the admin-set event name. Scramble mode = leaderboard (rank, team name in team colour, stations cleared, banked time, card icons) + master countdown + ticker. Blind Shot mode = the timer stage above. Results mode = final ranking with a confetti burst. Optional buzzer/horn sound effects on major events *(admin toggle; confirm default — venue AV dependent)*.

---

## 9. Host / admin view

PIN-gated (**default 1234**, changeable in config). Seven tabs:

| Tab | Functions |
|---|---|
| **Event** | Event name (default "2026 xBU GA Welcome Party"), subtitle/date line, editable labels for the two challenges (defaults "The Scramble" / "Blind Shot"), sound-effects toggle. Propagates live to team, board, and results views. |
| **Questions** | Add / edit / reorder / toggle the 15–20 questions (text, 4 options, correct answer) |
| **Rules** | Edit every default in §5–§7: slot bonuses & order, slots per station, lockout, Scramble length, Blind Shot base time, card values, team cap |
| **Codes & printing** | Generate station QRs + card set; render print-ready sheets: station posters, power-up cards (ID only), **Staff Master Sheet** (card ID → code → power-up type → suggested mission) |
| **Live control** | Start / end phases; switch board mode; manual adjustments per team (add/remove seconds or cards — the dispute-resolution lever); **Release team device** (§4.6); reset game (with confirm) for rehearsal |
| **Blind Shot** | The console described in §7 |
| **Data** | Raw state view; export results as CSV |

---

## 10. QR codes, power-up cards & staff sheet — generation & printing

**Generated inside the admin tab — no external tools.** This guarantees the printed materials exactly match the app's database and makes reprints one click.

- **Station QRs:** each QR encodes plain text like `GA-S07` — a **year-neutral** namespace prefix + station number, meaningless outside the app. Admin → "Print station posters" renders one A5 poster per station: giant station number + QR + one-line instruction. Posters deliberately carry **no event name or year**, so you **laminate once and reuse every cohort** — next year you swap the questions and rename the event in admin; the physical posters don't change.
- **Power-up cards:** print sheet renders each card with its **ID and instruction only** ("bring to a referee to unlock") — **no code on the card**. Print, fold/laminate, hide.
- **Staff Master Sheet:** one printable table for referees — card ID, its secret code, the power-up type, and a suggested micro-mission. One copy per referee.
- **Team QR:** a QR of the site URL, printed 5× — handed to the 5 team keepers at team formation (not projected).

---

## 11. Architecture & tech stack

| Layer | Choice | Why |
|---|---|---|
| Frontend | **Single `index.html`** — HTML + CSS + vanilla JS, hash-routed views | One file to maintain & deploy; no build step; Claude Code edits it directly |
| Libraries (CDN) | Firebase SDK · `html5-qrcode` (camera scanning) · `qrcode.js` (QR generation) · `canvas-confetti` · Google Fonts | All free, no installs |
| Shared state | **Firebase Realtime Database** (free Spark tier) | The one genuinely required backend: live sync across 7 devices (5 teams + board + host); transactions give atomic slot/code claiming; offline queue survives wifi blips |
| Session | `localStorage` for team identity + device token | Refresh-proof; enforces one-device binding |

**Data model (Realtime Database):**

```
/config      eventName · eventSubtitle · labels{challenge1, challenge2} · sounds ·
             bonuses [30,20,10] · lockoutSec 30 · scrambleMins 25 · blindBaseSec 120 ·
             teamCap 5 · phase · scrambleEndsAt · hostPin "1234"
/questions/{S01..S20}   text · options[4] · answerHash · active
/stations/{S01..S20}    slots: [ {teamId, ts} ]        (max 3, transaction-claimed)
/teams/{teamId}         name · colour · deviceToken · bonusSec · cleared{} · lockouts{} · powerups{}
/cards/{code}           type · value · redeemedBy      (transaction-claimed)
/blindshot/{teamId}     totalSec · baskets · playedCards[] · status
/feed                   ticker events
```

**Security, stated honestly:** party-grade and proportionate — unguessable URL, host PIN (1234) checked client-side, single-use codes, device-token binding, salted answer hashes (so a tech-savvy GA inspecting the database can't read the answer key). Not bank-grade; doesn't need to be. **Firebase keys in the code are fine** — they're public identifiers by design; access control lives in database rules, not key secrecy.

**Language:** English only (single-language UI strings; no i18n layer).

---

## 12. Hosting & deployment workflow (GitHub + Claude Code)

**Recommended: Cloudflare Pages** — the same host as your colleague's `ai-learnmap.pages.dev` (the `.pages.dev` domain is Cloudflare Pages' default), so there's in-house experience one desk away.

1. Create a GitHub repo. **Public is fine** — the repo contains no sensitive content, because questions, answers, and codes all live in Firebase, not in the code.
2. One-time: create a Firebase project (free) → enable Realtime Database → paste the web config into `index.html`.
3. One-time: in Cloudflare Pages, **connect the GitHub repo** (framework preset: none — it's a static file). You get `https://<project>.pages.dev` — pick something reusable like `xbu-ga-games.pages.dev` (year-neutral, per §1.1). HTTPS is automatic — **required for camera access**.
4. **Claude Code workflow:** clone the repo locally → drop this spec in as `SPEC.md` → tell Claude Code to implement it → iterate → `git push`. **Every push to the production branch auto-deploys in ~30–60 seconds.** Identical to your colleague's workflow.
5. Safer iteration near the event: work on a `dev` branch (Cloudflare gives each branch its own preview URL), merge to production only when happy; **code freeze end of Mon 20 Jul** and rehearse on the frozen build.

**Alternatives:** GitHub Pages or Netlify — all free, all push-to-production; Cloudflare Pages wins here on team familiarity.

**How many HTML files? One.** All three views, plus the printable sheets, live in the single hash-routed page (§2). Nothing to swap or keep in sync.

---

## 13. Visual style

Match the established GA brand system — modern, young, energetic, consistent with the party materials:

- **Palette:** coral, violet, sky blue, sunny yellow on warm off-white *(exact hexes to be lifted from the GA deck — placeholder tokens: coral `#FF6B5E`, violet `#8B5CF6`, sky `#38BDF8`, sunny `#FFD166`, off-white `#FFF9F2`)*. Each team is auto-assigned one accent colour on registration.
- **Type:** Archivo Black for display/headings, Plus Jakarta Sans for body.
- **Shape language:** rounded cards, soft shadows, chunky buttons; **no dark/cyber/glassmorphism**.
- **Tone:** theme-neutral copy — energy comes from colour, motion, and pace, not sports metaphor. No rookie/draft/season language anywhere in the UI; the basketball framing belongs to the MC and facilitators in the room.
- **Energy:** count-up animations on the wallet, ticker slide-ins, confetti on 1st-place claims and final results; board type sized for a sports hall.

---

## 14. Edge cases & resilience

| Scenario | Handling |
|---|---|
| Team phone refresh / crash / lock | `localStorage` session restore on the same device |
| Team phone dies mid-game | Host taps **Release team device** → replacement phone claims the team |
| Second device tries to join a team | Rejected — device-token mismatch (§4.6) |
| Venue wifi blip | Firebase offline queue syncs on reconnect; run team devices on **mobile data** as the recommended default |
| Two teams answer at once | Database transaction — slots claimed atomically in server order |
| Camera permission denied / broken | Manual station-number entry fallback |
| Dispute ("we found the card first!") | Host live-control manual adjustment |
| Host mis-tap (ends phase early) | Confirm dialogs on all phase changes; phases reversible from admin |
| Rehearsal | "Reset game (keep questions & cards)" |
| Projector legibility | Board designed at 1080p with hall-scale type |

---

## 15. Out of scope (v1)

User accounts · data retention beyond a CSV export · native app · Emoji Passport integration · long-term goal tracking (handled by Padlet + form per the 2-year plan) · live photo features · bilingual UI · multi-event history (one live game; export → reset per §1.1).

---

## 16. Decisions

**Resolved in v1.2:** configurable event branding (Event tab) · theme-neutral UI copy, default title "2026 xBU GA Welcome Party" · year-neutral QR namespace + event-name-free posters · continuity model (§1.1).
**Resolved in v1.1:** 30 pax, 5 teams of 6 · name-only registration · one-device anti-cheat (§4.6) · staff-mission power-up claim with Staff Master Sheet · Silence card · no difficulty multiplier · lockout 30 s · host PIN 1234 · English only · Cloudflare Pages · Blind Shot base 120 s.

**Remaining to confirm (the M0 gate — see §17):**

| # | Decision | Recommendation |
|---|---|---|
| 1 | Question format: multiple-choice only | **Yes** — reliable instant auto-marking |
| 2 | Tier bonuses 30/20/10 s · Scramble 25 min | Confirm or adjust |
| 3 | Card mix: 2 × Time Boost / 2 × Closer Line / 2 × Double Points / 2 × Silence | Confirm counts |
| 4 | Tiebreak: baskets → Scramble bonus earned → sudden-death free throw | Confirm |
| 5 | Board sound effects default on/off | Venue AV dependent |
| 6 | Exact brand hexes | To lift from the GA deck before M5 styling |

---

## 17. Development milestones (condensed — see companion Development Plan for checks & prep)

| Milestone | Date | Build scope | Gate |
|---|---|---|---|
| **M0 — Kick-off** | Tue 14 Jul | Spec v1.2 approved · §16 decisions confirmed · GitHub + Firebase + Cloudflare Pages set up · repo created with `SPEC.md` | You green-light the build |
| **M1 — Foundation** | Wed 15 Jul | Routing, Firebase wiring, Event tab (name propagation), team registration + device binding, board splash | 15-min check on live URL |
| **M2 — Scramble engine** | Thu 16 Jul | Scanner → question → slots/tiers → wallet · lockouts · code redemption · leaderboard + ticker + master clock | 2-phone mini-hunt |
| **M3 — Admin & printing** | Fri 17 Jul | Question manager, rules editor, QR/code generation, 3 print sheets, live control (release/reset), CSV export | Load real questions; print-preview all materials |
| **M4 — Blind Shot & results** | Sat 18 – Mon 20 am | Console, timer stage, card arming incl. Silence banner, +1/+2/undo, standings, results + confetti | Mock two team turns |
| **M5 — Polish & freeze** | Mon 20 Jul | Brand hexes, animations, sounds (if on), full game-cycle dry run + reset · **CODE FREEZE EOD** | Full run-through sign-off |
| **Rehearsal** | Tue 21 Jul | Venue walk: wifi test, station placement, projector legibility, referee briefing · print & laminate | No code changes |
| **Event** | Wed 22 Jul | Run the game · export CSV · reset for next use | 🎉 |

---

*Confirm the six §16 decisions (the M0 gate) and this spec is ready to hand to Claude Code as the build brief.*
