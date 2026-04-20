# Home Zone Furniture — Deployment Brand Notes

Source of truth for generating `data/store-config.json`. Fill in each section
with Blake; values flow into config or `index.html` where noted.

---

## 1. Store identity

- **storeName**: `Home Zone Furniture`
- **Wordmark split** (`logo.main` / `logo.sub`): `home zone` / `furniture`
- **Welcome screen logo**: styled text (same as Bel — no template change; logo image at `logo/HOME ZONE LOGO.png` kept available but not rendered in the wordmark slot)
- **languages**: `["en"]` (English-only for v1)
- **Deployment URL**: `https://beford782.github.io/HomeZoneFurniture/`
- **Repo**: `beford782/HomeZoneFurniture`

---

## 2. Brand colors

- **storePrimary**: `#A32F2C` — warm burgundy, eyeball-matched to the red circle in the Home Zone logo
- **storePrimaryLight**: `#BA3A37` — ~8% lighter, for hover/secondary
- **storePrimaryGlow**: `rgba(163, 47, 44, 0.15)` — 15% alpha for glow effects

Derived from the logo badge. Swap in an official brand hex later if the style guide surfaces one.

---

## 3. Trust signals & regional badge

- **trustSignal**: `Texas Born. Family Owned. Since 2007.`
- **madeBadge**: `Made in Texas`
- **socialProof**: `Trusted by 500,000+ Texas families`

---

## 4. Footer & email copy

- **footer**: `© 2026 Home Zone Furniture. All rights reserved.`
- **pageTitle**: `DreamFinder — Home Zone Furniture Sleep Quiz`
- **metaDescription**: `Take the DreamFinder sleep quiz at Home Zone Furniture and get personalized mattress recommendations.`
- **ogTitle**: `DreamFinder — Home Zone Furniture`
- **emailPrivacy**: `We'll only use your email to send your results.`
- **privacyPolicyContact**: `To have your information removed, contact your local Home Zone Furniture store.`
- **inStockText**: `In Stock at Home Zone Furniture`
- **emailHeader**: `Home Zone Furniture × DreamFinder`
- **emailSubtext**: `Bring this email to your Home Zone Furniture store`

---

## 5. Discount code prefix

- **discountPrefix**: `HOME` (4-letter match to template's `DREAM` so slot-machine animation keeps its width/timing)
- **Implementation**: introduce `discountPrefix` as a new field in `store-config.json` and patch `index.html` to read it (replaces hardcoded `'DREAM'` literals at ~line 3862 and downstream). Small config-driven extension to the template, per the "new features must be config-driven" rule.

---

## 6. Locally-made brands (scoring bonus)

The scoring engine adds +25 points and a match reason when `m.locallyMade === true` in the CSV. Locally-made rule for Home Zone:

| Brand | locally-made |
|---|---|
| HZ Sleep (house brand, incl. Diamond Firm) | `yes` |
| Tempurpedic | `no` |
| Stearns & Foster | `no` |

`madeBadge` for locally-made cards: "Made in Texas" (see §3).

---

## 7. Google Apps Script (email/lead backend)

- **GOOGLE_SCRIPT_URL**: _blank (to be deployed)_
- `index.html` will ship with an empty `GOOGLE_SCRIPT_URL` constant. Blake deploys a new GAS project (using `Code.gs` from template, under the Home Zone Google account) and pastes the `/exec` URL in afterward. Until then, email delivery is disabled; the rest of the app works.

---

## 8. Accessories

10 accessories. Prices blank across the board (per app-wide rule).

### Adjustable Bases (3)

| id | name | description | image |
|---|---|---|---|
| `base-2150` | 2150 Adjustable Base | Independent head and foot adjustment with wireless remote and adjustable leg heights | `adjustable-2150.webp` |
| `base-4150` | 4150 Adjustable Base | Head/foot movement with massage, four presets, Bluetooth app control, zero clearance | `adjustable-4150.webp` |
| `base-ergo30` | Tempur-Ergo 3.0 Adjustable Base | Premium Tempurpedic base with Sleep Tracker, Snore Response, zero gravity, and app control | `adjustable-ergo30.webp` |

### Pillows (3)

| id | name | description | image |
|---|---|---|---|
| `pillow-cooling-lowloft` | Active Cooling+ Pillow Low Loft 2-Pack | Pair of low-loft pillows with active cooling technology | `pillow-cooling-lowloft.jpg` |
| `pillow-cooling-extralowloft` | Active Cooling+ Pillow Extra Low Loft 2-Pack | Pair of extra-low-loft pillows for temperature-regulating sleep | `pillow-cooling-extralowloft.jpg` |
| `pillow-cooling-mediumloft` | Active Cooling+ Pillow Medium Loft | HZ Sleep medium-loft pillow with active cooling | `pillow-cooling-mediumloft.jpg` |

### Protectors (1)

| id | name | description | image |
|---|---|---|---|
| `protector-activedry` | Active Dry Protector | HZ Sleep mattress protector with moisture-wicking technology | `protector-activedry.webp` |

### Foundations (3, universal across deployments)

| id | name | description | image |
|---|---|---|---|
| `base-foundation` | Regular Foundation | Standard-height foundation — sturdy platform support that replaces a traditional box spring | `base-foundation.webp` |
| `base-lowprofile` | Low Pro Foundation | Reduced-height foundation — ideal for taller mattresses or platform-style frames | `base-lowprofile.webp` |
| `base-bunkie` | Bunkie Board Foundation | Slim board foundation for firmer support on existing bases | `base-bunkie.webp` |
