# Home Zone Furniture — DreamFinder Deployment Guide

## Working With Blake

If Blake asks you to do something and you know of a better approach — whether
it's a cleaner implementation, a simpler workflow, a tool that would make the
task easier, or a potential pitfall with the current approach — **speak up
before acting**. Briefly explain the alternative and let him decide. Don't just
silently execute what was asked if you can see a better path.

---

## What This Repo Is

This is the **Home Zone Furniture** white-label deployment of DreamFinder — a
store-agnostic single-page tablet kiosk app for mattress showroom floors.
Customers take a 12-question sleep quiz, get personalized mattress recommendations
across Gold/Silver/Bronze tiers, browse accessories, and receive results + a
discount code by email.

Deployed at: `https://beford782.github.io/HomeZoneFurniture/`
Repo: `https://github.com/beford782/HomeZoneFurniture`
Local path: `C:\Users\BlakeFord\Documents\GitHub\HomeZoneFurniture`

**Template source:** DreamFinder canonical white-label template lives at
`../DreamFinder/` (a sibling folder, also Blake's). When the template gets
improvements, they can be cherry-picked into this repo. Do NOT push
Home Zone-specific changes back into the DreamFinder template without explicit
cleanup — the template must remain store-agnostic.

**Other retailer deployments (separate repos, all white-label siblings):**
- Bel Furniture — `beford782/DreamFinder` (active, also serves as template)
- The Furniture Market — `beford782/TheFurnitureMarket` (active)
- Star Furniture — planned
- Lacks Furniture — planned

Never push Home Zone changes to another retailer's repo. Never copy
`data/mattresses.csv` or `data/store-config.json` between retailer repos —
every store has its own lineup and branding.

---

## This Store's Identity

- **Brand colors** (in `data/store-config.json`): primary `#A32F2C` (warm
  burgundy — matched to the Home Zone logo badge), light `#BA3A37`, glow
  `rgba(163, 47, 44, 0.15)`.
- **Wordmark split**: `home zone` / `furniture`.
- **Trust signal**: "Texas Born. Family Owned. Since 2007."
- **Social proof**: "Trusted by 500,000+ Texas families".
- **Locally-made badge**: "Made in Texas".
- **Discount prefix**: `HOME` (e.g. `HOME472`). Template default is `DREAM`;
  Home Zone uses `HOME` via the `discountPrefix` field in `store-config.json`
  (a config-driven extension added during HomeZone scaffolding — see
  `applyDiscountPrefix()` in `index.html` and note below).
- **Languages**: English only for v1 (`languages: ["en"]`). Spanish is a
  follow-up once English is validated in production.

---

## White-Label Architecture — Critical Rules

### The store-agnostic boundary

`index.html` must contain zero store-specific content. No retailer names,
logos, colors, mattress models, or discount codes hardcoded in the HTML.

All store identity lives in two files only:
- `data/store-config.json` — branding, store name, colors, languages, discount
  prefix, trust/footer/email text blocks
- `data/mattresses.csv` / `data/mattresses.json` — this store's mattress lineup

### New features must be config-driven

Any feature that could vary by store (colors, copy, quiz questions, tier names,
email templates, discount prefix) must be driven by `store-config.json`, not
hardcoded. If you find yourself writing a store name or brand color into the
HTML, stop — it belongs in config.

**Example from HomeZone scaffolding:** The discount prefix was hardcoded as
`'DREAM'` at ~line 3862 and 5390 of the template's `index.html`, plus
hardcoded D/R/E/A/M letters in the slot-machine HTML at line 3614. For
HomeZone's `HOME` prefix, we added `discountPrefix` to `store-config.json`
and added an `applyDiscountPrefix()` function in `index.html` that populates
the slot-machine letters from config and hides unused slots. This change is
backward-compatible: retailers without a `discountPrefix` in their config
fall back to `DREAM`.

### Quiz questions are currently hardcoded — known limitation

The 12 quiz questions and their answer options live in the HTML, not in
config. This is a known gap shared with Bel's template. Do not add more
hardcoded store-specific question logic. Flag any question customization
requests as requiring a config migration first.

---

## App Architecture — Read Before Touching Anything

### Single-file HTML

`index.html` is the entire app. No separate JS or CSS files. Do not split it.

### Domain Lock

A domain lock at ~line 3842 of `<script>` restricts where the app runs.
Allowed domains: `beford782.github.io`, `localhost`, `127.0.0.1`. Home Zone
deploys to `beford782.github.io` (same host as Bel), so no change required.
If deploying to a new host, add it to the `allowed` array.

### Data files
- `data/mattresses.csv` — source of truth for mattress lineup. Edit this;
  regenerate the JSON below.
- `data/mattresses.json` — **generated file**, never edit directly.
- `data/store-config.json` — all store-specific configuration.
- `data/dict-en.json` — English UI dictionary (shared across all retailers).
- `data/dict-es.json` — Spanish UI dictionary (shared across all retailers,
  kept even though HomeZone is English-only for template consistency).

No `data/mattresses-es.csv` — HomeZone v1 is English-only, so the Spanish
product translations layer is skipped. Re-introducing it is a follow-up.

### Build script

```
.\build-data.ps1
```

Run from repo root. Converts `data\mattresses.csv` → `data\mattresses.json`.
If `data\mattresses-es.csv` exists (not currently), merges Spanish
translations as `tags_es`, `highlight_es`, `reasons_es` into the JSON.
Always run this before committing if the CSV was changed. Never commit CSV
changes without also committing the regenerated JSON.

### CSV schema — follow the build script, not the template docs

The CLAUDE.md in Bel's repo has an incorrect column reference table that
swaps the meaning of `quizTags` and `features`. **Source of truth is
`build-data.ps1`**, which does the following per CSV row:

| CSV column | What it's actually used for |
|---|---|
| `tier` | JSON bucket (gold / silver / bronze) |
| `id` | Unique per tier (`g#` / `s#` / `b#`), never reused |
| `name` | Display name + image filename lookup (lowercased) |
| `brand`, `subBrand` | Display |
| `firmnessScore`, `firmnessLabel` | Scoring engine + display |
| `price` | **Not displayed** — leave blank |
| `quizTags` | **Dead column — unread by build script. Ignore.** |
| `displayBadges` | Pipe-delimited chips shown on the card (`tags` in JSON) |
| `highlight` | Short hero line (~10 words) on the card |
| `locally-made` | `yes` / `no` — drives +25 scoring bonus |
| `features` | **Scoring engine input** — pipe-delimited lowercase-kebab tags (build script converts to camelCase for matching) |
| `reason_*` | Match reasons shown under mattress card after quiz |

**Valid scoring-tag vocabulary** (features column — tags the engine actually
scores on via `opt.scores` calls):
`plush · pressureRelief · soft · support · medium · zoned · firm · responsive · motionIsolation · hybrid · cooling · memory · durability · comfort · hypoallergenic · adjustable · quality`

In the CSV write kebab-case (`pressure-relief`, `motion-isolation`). Build
script lowercases and converts to camelCase.

Additional tags in use but NOT scored (they feed display badges via
substring matching at ~line 5165–5195): `copper`, `anti-viral`,
`hand-tufting`, `back-supporter`. And many descriptive tags (`latex`, `wool`,
`tencel`, `serene-foam`, etc.) are in CSV rows but currently unused —
**keep them**. They're real product attributes; the gap is that no quiz
question scores on material preference. That's a template enhancement
deferred to the DreamFinder template cleanup pass.

---

## Scoring Engine — How Recommendations Work

Located in `index.html` around line 4040. Three scoring passes:

**1. Firmness (most important, max +50)**
Linear sliding scale: `firmScore = max(0, 50 - diff * 10)` where
`diff = |customerFirmness - mattressFirmness|`. So: diff 0 = +50, diff 1 =
+40, diff 2 = +30, diff 3 = +20, diff 4 = +10, diff 5+ = 0. Additionally,
if `diff ≥ 4` an extra **-20** penalty is applied — diff of 4 nets -10 and
diff of 5+ nets -20.

**2. Feature matching**
Quiz answers map to feature tags via `opt.scores`. Each matching tag adds
points (with a per-tag cap). Tags are stored in the JSON `features` array
(mapped from kebab-case `features` column in CSV).

**3. Locally made bonus (+25)**
If `m.locallyMade === true`, adds 25 points and appends a match reason.

**For Home Zone:** `locally-made: yes` on all HZ Sleep products (including
Diamond V Firm). `no` on Tempurpedic and Stearns & Foster.

Qualified results = top 3 models scoring ≥ 60% of the top score.

**IMPORTANT:** Do not modify scoring weights or logic without confirming with
Blake. This area has had significant prior tuning shared with Bel's deployment.

---

## Backend — Google Apps Script (blank until deployed)

Email delivery and lead logging use a Google Apps Script (GAS) web app. The
GAS endpoint URL lives in `index.html` at ~line 5538 (`GOOGLE_SCRIPT_URL`).

**Current state: blank.** Email/lead logging is disabled until Blake deploys
the GAS script under Home Zone's Google account and pastes the `/exec` URL.
The rest of the app works without it.

To deploy: open `Code.gs` in a new Google Apps Script project, deploy as a
web app, copy the `/exec` URL into the `GOOGLE_SCRIPT_URL` constant in
`index.html`, commit, push.

Do NOT reuse Bel's GAS endpoint — each retailer needs its own GAS project
and Google Sheet for lead logging.

If GAS needs redeployment: Manage Deployments → pencil icon → New version.

---

## iPad / Touch Event Rules

The app runs on iPads in showrooms. Preserve in all changes:
- `touch-action: manipulation` on all interactive elements
- Dynamic elements (mattress cards, buttons) need both `touchend` and
  `pointerdown` listeners
- `event.preventDefault()` on touchend handlers to prevent ghost clicks
- `location.reload()` must never be used — always call `window.startOver()`
  to reset

**IMPORTANT:** Do not change touch handling without confirming with Blake —
this required significant debugging to get right on the template.

---

## Bilingual / i18n Architecture (inactive in v1)

The app ships with full English + Spanish infrastructure. Home Zone's
`languages: ["en"]` in `store-config.json` hides the toggle and keeps the
app English. The Spanish dictionary and mattress translation path still
exist and work; Blake can flip Spanish on later by adding `"es"` to
`languages` and providing `data/mattresses-es.csv` + `text_es` blocks in
`store-config.json`.

**Rules for new features** (still apply even while Spanish is off, so the
template stays shippable for other retailers):
- Any new user-facing string must be bilingual. Use `t('key')` for dict
  lookups or `{en: "...", es: "..."}` with `L()` for inline data.
- Never hardcode English-only display text in JavaScript template literals.
- Retailer-specific copy (store name, taglines, footer) goes in
  `store-config.json` `text` and `text_es` blocks — never in the dict files.

---

## Accessories

Home Zone's accessory lineup (in `images/accessories/`, declared in the
`accessoryData` JS array around line 4251 of `index.html`):

**Adjustable Bases (3):** 2150 Adjustable Base, 4150 Adjustable Base,
Tempur-Ergo 3.0 Adjustable Base.

**Foundations (3, universal template items):** Regular Foundation, Low Pro
Foundation, Bunkie Board Foundation. These 3 are shared across all
retailers — images currently sourced from Bel's repo.

**Pillows (3):** Active Cooling+ Low Loft 2-Pack, Extra Low Loft 2-Pack,
Medium Loft.

**Protectors (1):** Active Dry Protector.

Prices are NOT displayed in the app for any product (mattresses or
accessories). Leave all `price` fields blank / unused.

---

## Key App Flows (Don't Break These)

- **Quiz → Results**: 9 questions → scoring engine → Gold/Silver/Bronze tier
  tabs → top pick badge.
- **Mattress drawer**: Opens on card tap. Prev/next navigation between
  results. Firmness bar, match reasons, features.
- **Accessories / Sleep System**: Framed as "Build Your Sleep System" (not
  add-ons). Conditional adjustable base hero (shown when quiz flags snoring,
  reflux, or back pain) with animated SVG and personalized benefit cards.
  Featured top pillow with "Matched to Your Profile" badge. "Did You Know?"
  educational callout for protectors. Sticky cart bar. Cart persists to
  handoff screen.
- **Discount reveal**: Dramatic animation — `HOME` + 3-digit code. 10rem
  gold glow font. Letters populated at runtime from
  `STORE_CONFIG.discountPrefix`.
- **Handoff screen**: Customer marks "I'm Interested" on mattresses/
  accessories. Salesperson sees saved picks.
- **Idle timeout**: Inactivity triggers reset flow back to start screen.
  Uses `window.startOver()`.

---

## Deployment

```
git add .
git commit -m "description"
git push origin main --force
```

Force push is intentional — always used for this repo (matches the template's
convention). GitHub Pages updates within 1–2 minutes after push.

### IMPORTANT: Claude Code on the Web creates feature branches automatically

If this session is running in Claude Code on the web (claude.ai/code),
pushes default to a `claude/<name>-<id>` branch — NOT to main. GitHub Pages
only deploys from main, so those pushes do not go live.

**At the start of every session and before any commit/push, Claude MUST:**
1. Check the current branch with `git branch --show-current`.
2. If the branch is anything other than `main`, warn Blake clearly that
   pushes on this branch will NOT deploy, and offer to either:
   - Push to main directly: `git push origin HEAD:main --force`, or
   - Use the `git ship` alias (configured locally in Blake's `DreamFinder`
     repo; may need to be configured here too) which does the same.

Never assume a push to a `claude/*` branch is a successful deployment. Always
confirm main was updated before reporting that a change is live.

---

## Division of Labour — Regular Claude vs Claude Code

**Regular Claude (claude.ai)** — design, logic, new features, producing
updated HTML and scripts. Works across sessions without repo access.

**Claude Code** — applying file changes, running build scripts, committing,
pushing.

### Handoff workflow

When Regular Claude produces a new `index.html` or `build-data.ps1`:
1. Replace the existing file in the repo.
2. If CSV was also updated, run `.\build-data.ps1` to regenerate the JSON.
3. Verify no regressions in touch events, scoring, or GAS integration.
4. Commit and push.

### What Claude Code should never do without checking with Blake first

- Modify scoring weights or logic in the quiz engine.
- Change touch event handling.
- Hardcode any store-specific content into `index.html` (discount prefix is
  already config-driven via `storeConfig.discountPrefix` — don't revert to a
  hardcoded literal).
- Copy mattress data or config between retailer repos.
- Edit `data/mattresses.json` directly (always regenerate from CSV).
- Add store names, colors, or branding anywhere except `store-config.json`.
- Push Home Zone changes to the `DreamFinder` template repo, or vice versa.
