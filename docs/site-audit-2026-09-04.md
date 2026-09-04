# Sparkle Clean — Comprehensive Site Audit

- **Date:** 2026-09-04
- **Scope:** Full codebase — `index.html` (1,026 lines), `assets/hero.webp`, `docs/`, `.gitignore`, git state
- **Auditor:** Cline

## Method (evidence-based, not eyeballed)

1. Full line-by-line read of `index.html`, the approved spec, and `.gitignore`.
2. Static scans: duplicate IDs, heading tree, external requests, asset inventory, inline styles, repeated SVGs.
3. WCAG contrast ratios computed with the official formula for **all 13 foreground/background pairs** in use.
4. Inline `<script>` extracted and syntax-checked with `node --check` (Node v24.13.1) → **OK**.
5. Implementation cross-checked against the approved spec (`docs/superpowers/specs/2026-09-03-sparkle-clean-landing-design.md`).

## Scorecard

| Category | Score | Summary |
|---|---|---|
| Accessibility | 6.5/10 | Solid landmarks/labels, but CTAs fail AA contrast; form errors not programmatic; no reduced-motion support |
| Performance | 5/10 | Hero image is 5000×3333px / 450KB displayed at ≤600×480 (~10× oversized, likely LCP); otherwise lean single file |
| SEO | 5/10 | Good title/H1/description basics; missing LocalBusiness schema, OG/Twitter, canonical, favicon |
| Conversion/UX | 6.5/10 | Strong persuasion structure, but mailto-only form can silently lose leads; poor mobile nav; bad select defaults |
| Code quality | 8/10 | Clean, consistent, syntax-valid; minor DRY issues (repeated SVGs), 5 inline style attrs |
| Spec compliance | 9/10 | Faithful to approved design; 3 minor deviations (documented below) |

## Verified facts (scans)

- IDs: 23 total, **0 duplicates**
- Headings: 1×h1 → 3×h2 → 8×h3 → 2×h4 — no skipped levels
- External HTTP references: **0** (fully self-contained, privacy-strong, CSP-ready)
- `tel:` links: 5 · email placeholders: 2 (footer text + JS mailto)
- Repeated inline SVGs: sparkle logo ×5, checkmark ×15
- Inline `style=""` attributes: 5
- `<img>` tags: 1 (hero) — `assets/hero.webp` is **5000×3333 px, 450 KB** (Windows Shell metadata)
- Meta description: **178 chars** (truncates in Google SERPs at ~155–160)
- Missing (0 matches each): favicon, `og:`, `twitter:`, canonical, JSON-LD, `prefers-reduced-motion`, `scroll-padding`, `aria-describedby`, `aria-invalid`, `srcset`, `fetchpriority`, `theme-color`, `<noscript>`
- Inline JS: `node --check` → **syntax OK**

## Contrast audit (computed, WCAG 2.x)

| Ratio | Pair | Verdict |
|---|---|---|
| **3.32** | white on `--teal #0F9D9D` (btn-primary L64, badge-popular L259, mobile-call L541) | ❌ **FAIL** — needs 4.5 for 16px text |
| 5.08 | white on `--teal-dark #0B7B7B` (button hover L65; btn-outline text) | ✅ Pass |
| 7.79 | white on `--teal-darker #085C5C` (topbar, contact section, h2) | ✅ Pass |
| 14.68 | `--text` on white | ✅ Pass |
| 5.58 | `--text-soft` on white | ✅ Pass |
| 5.19 | `--text-soft` on grey sections | ✅ Pass |
| 4.50 | `btn-outline` text on hover tint | ⚠️ Exactly at AA threshold |
| 5.82 | contact lead on dark teal | ✅ Pass |
| 6.63 | footer text on footer bg | ✅ Pass |
| 5.44 | form error text | ✅ Pass |
| 5.86 | form success text | ✅ Pass |
| 3.32 | teal focus ring on white (needs 3.0 UI) | ✅ Pass (thin margin) |
| 3.09 | teal focus ring on grey (needs 3.0 UI) | ⚠️ Barely passes |

Note the inversion: the **default** button state fails AA while the hover state passes.


---

## 🔴 CRITICAL — fix before launch

### C1. The form can silently lose leads (mailto-only + false success message)
**Where:** L854 (`novalidate` form), L999–1007 (JS), L908–910 (message)
The submit handler shows *"Your quote request has been received. We'll contact you within 24 hours"* — **but nothing has been received by anyone**. The only transmission is `window.location.href = 'mailto:...'` (L1002–1005), which:
- Does nothing detectable on machines with no configured desktop mail client (very common).
- On mobile often opens nothing, or the wrong app; webmail users (Gmail-in-browser) get no path at all.
- Requires the user to *manually press send* in their mail app for the lead to exist.

**Impact:** For a lead-gen page this is the single biggest business risk — visitors believe they requested a quote; the owner never hears anything. No error, no fallback, no logging.
**Fix (any one):** Netlify Forms / Formspree / Web3Forms endpoint (keeps static hosting), or WhatsApp deep-link handoff (`https://wa.me/8801712345678?text=...` — dominant channel in Bangladesh), or `action="https://formsubmit.co/..."`. At minimum, reword the confirmation to be truthful ("We've prepared your quote request — press send in your mail app to submit it") and keep focus on it.

### C2. All primary CTAs fail WCAG AA contrast (3.32:1)
**Where:** L64 (`.btn-primary` background `--teal`), L259 (`.badge-popular`), L541 (`.mobile-call`), L10 (`--teal` token)
White text on `#0F9D9D` = **3.32:1**; AA requires 4.5:1 for 16px/600 text. Every conversion element — "Get a Free Quote", "Get My Free Quote", the "Most Popular" badge, the floating call button — is non-compliant, including for low-vision users in bright daylight. Ironically the **hover** state (`--teal-dark`, 5.08:1) passes; the resting state doesn't.
**Fix:** Make solid buttons/badges use `--teal-dark #0B7B7B` (5.08:1 ✅) or darken the `--teal` token itself; keep `#0F9D9D` for decorative tints/icons only.

### C3. iOS Safari zooms the viewport on form focus
**Where:** L471 (`.field input, .field select { font-size: 0.95rem }` ≈ 15.2px)
Safari on iOS auto-zooms into any input with font-size < 16px, shifting the whole layout on every field of the money form. Classic, high-frequency mobile bug on the most important screen.
**Fix:** `font-size: 16px` (or `1rem`) on inputs/selects.

---

## 🟠 HIGH — strongly recommended

### H1. Hero image is ~10× oversized (LCP element)
**Where:** L643 (`assets/hero.webp`), asset is **5000×3333 px / 450 KB**, displayed at ≤600×480 (CSS 5/4 ratio).
~16.7 megapixels delivered for a 0.29-megapixel slot. This is almost certainly the Largest Contentful Paint element on every load, especially mobile.
**Fix:** Resize/export to 1200×960 (2× retina) WebP q75–80 → expect ~60–90 KB (−80%+). Add `fetchpriority="high" decoding="async"` to the `<img>`. Optionally an 800px `srcset` variant for small screens.

### H2. Zero structured data for a local business
**Where:** `<head>` L3–7
No JSON-LD `LocalBusiness`/`HomeAndConstructionBusiness` schema — no name, phone, hours, area served, rating in machine-readable form. For a local service page this is the highest-ROI SEO artifact (eligible for rich results, maps knowledge panel).
**Fix:** Add a small JSON-LD block (name, tel, address/area, openingHours, aggregateRating once reviews are real).

### H3. No social/share/canonical/favicon metadata
**Where:** `<head>` L3–7 — confirmed 0 occurrences of `og:`, `twitter:`, canonical, `rel="icon"`, `theme-color`.
Links shared on WhatsApp/Facebook (primary referral channels for a local BD business) render as a bare URL — no preview card, no brand favicon in tabs/bookmarks.
**Fix:** Add OG + Twitter tags, a favicon (the SVG sparkle logo makes an easy `.svg`/`.ico` icon), `theme-color: #085C5C`, canonical.

### H4. Form errors are visual-only — not programmatic (a11y)
**Where:** L860–874 (error `<p>`s), L976–985 (validation JS)
Confirmed **0** uses of `aria-invalid` / `aria-describedby`. Screen readers get no signal a field failed or which message belongs to it; sighted-keyboard users get no focus move to the success banner (L999–1000 scrolls to it but focus stays on the button). The `display:none → block` toggles for errors/success also announce unreliably in some screen-reader/browser combos.
**Fix:** On invalid: `input.setAttribute('aria-invalid','true')` + `aria-describedby="error-name"`; remove attrs on valid. Give `#form-success` `tabindex="-1"` and call `.focus()`.

### H5. No mobile navigation at all
**Where:** L559 — `nav.main-nav a:not(.btn) { display:none }` below 900px; no hamburger exists.
Spec says nav "collapses gracefully on mobile" — implementation just deletes it. Mobile users (the majority audience for a local cleaning service) cannot jump to Services/Why Us/Reviews; they only get the quote CTA.
**Fix:** Add a minimal disclosure hamburger (button + toggle), or at minimum keep a compact anchor row.

### H6. Past-date bookings accepted
**Where:** L879 (`<input type="date">` — no `min`), JS does no date check.
Users can request a cleaning for yesterday; leads need manual sanity-checking.
**Fix:** Set `min` to today via JS (`date.min = new Date().toISOString().split('T')[0]`) and validate on submit.

### H7. Smooth scrolling ignores motion preferences
**Where:** L28 `html { scroll-behavior: smooth; }` + hover `translateY` transitions; confirmed 0 `prefers-reduced-motion` matches.
Vestibular-disorder users get forced animation on every anchor jump (and this page is all anchor jumps).

---

## 🟡 MEDIUM

| # | Issue | Where | Detail / Fix |
|---|---|---|---|
| M1 | Placeholder contact details in **7 code locations** | L583, L622, L833, L939, L953 (tel ×5) · L940 + L1002 (email ×2) | Single grep misses = dead phone number in production. Keep a launch checklist or centralize via one `<script>` constant. |
| M2 | Footer email is plain text, not a link | L940 | Inconsistent with every other contact item; add `mailto:` (and expect scraping once real — consider obfuscation). |
| M3 | Select defaults skew lead data | L883–887 (time → "Morning"), L893–898 (service → "Standard Clean") | Users who don't care get silently recorded as Morning/Standard. Add "Any time" and make "Not sure yet" the default service. |
| M4 | No `<noscript>` fallback | L854 | With JS disabled the form silently reloads and loses everything. Add a noscript notice with tel/WhatsApp link. |
| M5 | Success banner behavior | L908–910, L999–1000 | `role="status"` with initial `display:none` announces inconsistently across SR/browser combos; banner persists on repeated submits; verify with VoiceOver/NVDA or switch to always-in-DOM + focus. |
| M6 | Featured card layout jump | L253 | `.service-card.featured` adds a 2px border siblings lack → 4px size/alignment difference. Use `box-shadow: inset 0 0 0 2px var(--teal)` or border on all + transparent color. |
| M7 | `aria-label` on role-less generic divs | L627 (`.trust-chips`), L646 (`.hero-rating`) | Ignored by most screen readers without a role. Add `role="list"`/`role="img"` or drop the labels. |
| M8 | `<cite>` misused for person names | L806, L811, L816 | `<cite>` is for titles of works; use `<footer>` or `<span class="reviewer">` inside the blockquote. |
| M9 | Hard-coded © 2026 | L947 | Will silently go stale in January; inject via `new Date().getFullYear()`. |
| M10 | Sticky header overlaps anchor targets | L28 area | No `scroll-padding-top`; 72px section padding mostly masks it, but add `html { scroll-padding-top: 80px }` for safety. |
| M11 | Floating call button only ≤480px | L533–534, L571 | Tablets/large phones (481–900px) — a huge touch segment — get no persistent tap-to-call. Show from ≤900px. |
| M12 | Meta description 178 chars | L7 | Truncates with "…" in SERPs; trim to ≤160 while keeping the CTA sentence. |

## 🟢 LOW / polish

| # | Issue | Where |
|---|---|---|
| L1 | SVG duplication: sparkle logo ×5, teal checkmark ×15 | throughout — define once in `<defs><symbol>` + `<use href>`; saves bytes and edit-risk |
| L2 | 5 inline `style=""` attributes | L791, L849, L850, L901 — move to classes |
| L3 | No explicit `::placeholder` styling | browser default only |
| L4 | `.form-note` at 12.8px is very small | L485 |
| L5 | No print stylesheet | — |
| L6 | Git repo not initialized (`.gitignore` present, no `.git`) | repo root — run `git init` + first commit |
| L7 | `.gitignore` is a 176-line multi-language template (Flutter/Java/Python) for a 4-file static site | harmless but noisy |
| L8 | No `404.html` / hosting config | deployment-time concern |

## 📐 Spec compliance (spec: 2026-09-03, "Approved Option A")

| Spec says | Implementation | Verdict |
|---|---|---|
| Breakpoints 768px & 480px | 900px & 480px (L553, L562) | ⚠️ Deviation (arguably better) — update spec or code |
| Service CTA preselect "via query/hash" | `data-service` attr + JS (L679/700/720 → L1010–1021) | ✅ Equivalent; values verified to match option values exactly |
| Hero via Unsplash stock URL + fallback | Local `assets/hero.webp` (L643) | ✅ Deviation, but strictly better (no third-party dependency) |
| Everything else (structure, colors, form fields, mailto+confirmation, floating call button, placeholders) | implemented as specified | ✅ |

## ✅ What's genuinely good (verified, not flattery)

- **Zero external requests** — no tracking, no fonts, no CDNs; fast, private, works offline, CSP-ready (no inline event handlers anywhere).
- **No CLS from the hero** — `width`/`height` attrs + `aspect-ratio` + `object-fit` (L205–214, L643–645).
- **Clean semantics** — 23/23 unique IDs, correct heading tree, `aria-labelledby` on every section, all inputs labeled with `autocomplete`, decorative SVGs `aria-hidden`.
- **Honest, scoped JS** — syntax-checked OK; no XSS vectors (user input never injected into DOM); graceful focus of first invalid field.
- **12 of 13 color pairs pass AA** — the palette was clearly contrast-minded; only the button fill misses.
- **Persuasion architecture matches spec** — trust chips, featured tier, Google review card, guarantee framing, sticky CTA, floating call button.

## 🎯 Prioritized fix plan (~1 focused day)

| Order | Fix | Effort | Addresses |
|---|---|---|---|
| 1 | Buttons/badge → `--teal-dark` background | 5 min | C2 |
| 2 | Inputs to `font-size:16px` | 2 min | C3 |
| 3 | Re-export hero.webp at 1200×960 q80 + `fetchpriority` | 15 min | H1 |
| 4 | Real form endpoint (Formspree/Web3Forms/Netlify) **or** WhatsApp handoff + truthful wording | 30–60 min | C1 |
| 5 | `aria-invalid`/`aria-describedby` + focus success banner | 20 min | H4, M5 |
| 6 | Head pack: OG, Twitter, canonical, favicon, theme-color, JSON-LD LocalBusiness | 45 min | H2, H3 |
| 7 | `prefers-reduced-motion` + `scroll-padding-top` | 5 min | H7, M10 |
| 8 | `min` date + select defaults + "Any time" | 10 min | H6, M3 |
| 9 | Mobile nav (hamburger or compact anchors) | 1–2 h | H5 |
| 10 | Footer email link, dynamic year, noscript note, tablet call button | 20 min | M1–M4, M11 |
| 11 | SVG `<symbol>` consolidation, inline styles → classes | 30 min | L1–L2 |

*Deferred by design (per spec "Out of Scope"): backend persistence, analytics, maps, i18n — revisit only if requirements change.*

**Fix:** `@media (prefers-reduced-motion: reduce) { html { scroll-behavior:auto } * { transition:none!important; animation:none!important } }`.