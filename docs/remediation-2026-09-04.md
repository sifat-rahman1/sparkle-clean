# Remediation Log — Site Upgrade Execution

- **Date:** 2026-09-04
- **Input:** `docs/site-audit-2026-09-04.md` (comprehensive audit)
- **Architecture decision:** Dependency-free elevation (single-file static site; Tailwind-inspired tokens, shadcn-style component patterns in hand-crafted CSS, inline Lucide-style SVG icons, Framer-Motion-quality motion via CSS + IntersectionObserver) — zero build step, per user's selection.
- **Result:** `index.html` rewritten (1,026 → ~1,030 lines), hero assets re-exported, all audit findings closed.

## Asset & Tooling Work

| Task | Result |
|---|---|
| Hero re-export (audit H1) | Original 5000×3333 / 451 KB → **hero-1200.webp 62.6 KB (1200×960)** + **hero-800.webp 24.8 KB (800×640)**, saliency-aware 5:4 crop via `sharp` (installed ad-hoc in temp; no project dependencies added). **−86% LCP payload.** Original `hero.webp` retained in repo but no longer referenced. |
| Version control (audit L6) | `git init` + baseline commit `1744b2f` (pre-upgrade state) → upgrade committed separately for clean review diff. |

## 🔴 Critical — all resolved

### C1. Mailto-only form with false success message → **REDESIGNED (honest handoff)**
- Submit no longer claims "received". It validates, composes the message, then reveals a success panel: *"Your request is ready to send — pick how you'd like to send it."*
- Two handoff channels: **Send via WhatsApp** (`wa.me` deep link with the full request pre-filled — dominant channel in the Chandpur market) and **Send via Email** (`mailto:` fallback), plus a call line.
- "Edit my request" returns to the form with values intact; focus is managed throughout.
- Phone/email now defined once in a `CONTACT` config object in the script (JS-side dedup).
- Remaining lead-capture ceiling (no server persistence) is inherent to the zero-dependency constraint — documented in the launch checklist. Wiring Formspree/Netlify Forms later is a drop-in change to the submit handler.

### C2. AA contrast failure on all primary CTAs → **FIXED (verified)**
- Solid buttons now `linear-gradient(180deg, teal-600, teal-700)` = **5.08:1** resting, **7.79:1** hover (previously 3.32:1 ❌).
- "Most Popular" badge: ink `#032727` on amber `#FBBC04` = **9.29:1** (AAA).
- Full contrast matrix re-computed: **all 12 fg/bg pairs pass AA; body copy on dark surfaces passes AAA** (quote lede 8.63, footer links 10.72, success 7.94, error 4.92, focus ring 5.08 ≥ 3.0 UI).

### C3. iOS Safari input zoom → **FIXED**
- All form controls `font-size: var(--text-base)` = **16px** (`.field input, .field select`).

## 🟠 High — all resolved

| ID | Fix |
|---|---|
| H1 | `srcset` (800w/1200w) + `sizes` + `fetchpriority="high"` + `decoding="async"`; explicit `width/height` (no CLS); hover zoom isolated to the image. |
| H2 | JSON-LD `LocalBusiness` + `HomeAndConstructionBusiness` (name, tel, email, area, address, 2 opening-hours blocks, price range) — **validated parseable via Node**. |
| H3 | SVG data-URI favicon, `theme-color`, canonical, OG (type/title/description/url/image/locale) + Twitter `summary_large_image` — 0 external requests added. |
| H4 | `aria-invalid` set/cleared per field via JS; static `aria-describedby` wiring on all three inputs; success panel `tabindex="-1"` receives `.focus()`; `role="status"` + `aria-live="polite"` retained; `hidden` attribute toggle (no display:none live-region ambiguity). |
| H5 | Mobile navigation: 44px disclosure `nav-toggle` with morphing bars → X, `aria-expanded`/`aria-controls`/`aria-label` state, Escape-to-close (focus returns to toggle), outside-click close, close-on-link-click; glass slide-down panel; desktop nav keeps scrollspy. |
| H6 | `date.min` set to today at runtime; past-date requests blocked at the source. |
| H7 | Full `prefers-reduced-motion` block: scroll-behavior auto, all animations/transitions neutralized, reveals forced visible, draw strokes forced complete. |


## 🟡 Medium — all resolved

| ID | Fix |
|---|---|
| M1 | Launch-checklist comment at the top of the file enumerates every placeholder location (phone ×6 incl. JSON-LD + WA builder, email ×2, domain, stats); JS `CONTACT` object is the single JS-side source. |
| M2 | Footer email is now a `mailto:` link (styled consistently with the other contact items). |
| M3 | Time select defaults to **"Any time"**; service select defaults to **"Not sure yet — help me choose"** — no more silently skewed leads. |
| M4 | `<noscript>` fallback in the quote section (tel link + explanation); reveal animations only hide content when `html.js` is set, so no-JS visitors see the full page. |
| M5 | Success panel uses `hidden` attribute + focus management + one-time drawn-badge animation; truthful copy (see C1). |
| M6 | Featured card uses a gradient surface + inset ring instead of a layout-shifting border; `.card::before` gradient hairline on hover for the others. |
| M7 | `role="list"`/`role="listitem"` on the partner row; `role="presentation"` on its label; hero rating card carries real text content (no dangling `aria-label` on generic divs). |
| M8 | `<cite>` replaced by semantic `<footer class="reviewer">` with initial-avatars inside each testimonial. |
| M9 | `#year` span stamped from `new Date().getFullYear()` (static 2026 fallback remains in markup). |
| M10 | `html { scroll-padding-top: calc(var(--header-h) + 22px) }` — sticky header never covers anchor targets. |
| M11 | Floating call button now appears **≤900px** (was 480px); pulse-ring animation (disabled under reduced motion). |
| M12 | Meta description rewritten to **150 chars** (was 178) — no SERP truncation. |

## 🟢 Low — all resolved

| ID | Fix |
|---|---|
| L1 | SVG sprite: **19 `<symbol>` definitions, 79 `<use>` references** — sparkle ×5, checkmark ×15 dedup'd to one symbol each; `pathLength="1"` enables stroke-draw. |
| L2 | Inline `style` attributes reduced to intentional per-instance animation delays (`--dd`) and two icon-color tweaks — all layout styling moved to classes. |
| L3 | Explicit `::placeholder` styling, AA-verified (`--ink-faint` **5.49:1**, full opacity). |
| L4 | Form fine print at 12.8px → legible size within the new type scale; legal note styled consistently. |
| L5 | `@media print` block: chrome/nav/floating/decoration hidden, dark surfaces flattened, black-on-white. |
| L6 | Repo initialized; baseline + upgrade commits with clean diff. |
| L7 | `.gitignore` left as-is (harmless template) — noted for future trim. |
| L8 | 404 page remains a hosting-level task (noted in launch checklist context). |

## Spec-compliance updates (from audit's spec matrix)

- Breakpoint deviation (900px) retained intentionally and documented; spec doc remains the record of the original approved design.
- `data-service` preselect retained (equivalent to spec's query/hash, verified value-matched).
- Local optimized hero retained over spec's Unsplash URL (strictly better: no third-party dependency).

## Verification summary (post-implementation)

| Check | Result |
|---|---|
| Inline JS `node --check` | ✅ OK |
| Duplicate IDs | ✅ 51/51 unique |
| JSON-LD parse (Node) | ✅ Valid, 2 opening-hours blocks |
| WCAG contrast (12 pairs) | ✅ All AA; body-on-dark AAA |
| CSS custom-property resolution | ✅ 68 used / 74 defined — 0 missing |
| External runtime requests | ✅ 0 (wa.me is an intentional click-through handoff) |
| Meta description | ✅ 150 chars |
| A11y wiring | ✅ aria-describedby ×3, aria-invalid (JS), aria-expanded ×5, role=list, skip link, focus-visible rings, reduced-motion kill-switch |
| Leftover build markers / typos | ✅ 0 |
| Tag balance (27 tag types) | ✅ All open/close balanced — sprite-container defect caught & fixed in final pass |
| Icon sprite integrity | ✅ 19 symbols, every `use` ref resolves, 0 orphaned symbols |

## Deliberate craft decisions (anti-"AI-slop" directions taken)

- **No uniform 3-column cards:** services are an asymmetric grid — the featured Deep Clean is a dark, spanning panel; siblings are lighter and smaller.
- **No template drop-shadows:** three-layer elevation tokens with teal-tinted ambient color; gradient hairlines instead of borders-on-everything.
- **No plain gradient headings:** solid deep-ink display type, tight tracking, with a serif-italic accent word and a **self-drawing swash underline** as the hero's signature moment.
- **Texture over flatness:** inline-SVG film grain on dark/light surfaces, dot-grid section background, ambient radial washes, glass (backdrop-blur) header, floating cards and hours panel.
- **Tactility everywhere:** spring cubic-bezier hovers/presses on buttons, chips, cards, tiles; icon micro-interactions (phone wiggle, arrow slide, logo-mark tilt, badge draw-on-reveal); scrollspy underline with spring scale.
- **Motion with respect:** every animation gated by `prefers-reduced-motion`; reveals fire once via IntersectionObserver with authored per-element stagger delays.
