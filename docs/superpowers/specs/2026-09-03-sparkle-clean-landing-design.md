# Sparkle Clean Landing Page — Design

Date: 2026-09-03
Status: Approved (Option A)

## Goal

A trustworthy, high-conversion single-page marketing site for "Sparkle Clean", a local house cleaning service in **Chandpur**. Static `index.html` with embedded CSS/JS. No build step, no dependencies.

## Design System

- Colors: teal primary `#0F9D9D`, dark teal accent `#0B7B7B`, white `#FFFFFF`, light grey section background `#F5F7F7`, text `#1F2937`
- Typography: system font stack (Poppins via Google Fonts optional; system stack used to avoid external dependency)
- Spacing: 8px baseline grid; sections max-width 1100px centered
- Border radius: 12px cards, 8px buttons; subtle shadows for depth

## Page Structure (top to bottom)

1. **Top bar** — thin teal strip. Left: tagline "Licensed & Insured Cleaning Professionals". Right: clickable `tel:` phone number (opens dialer).
2. **Header/nav** — logo "Sparkle Clean" (sparkle icon), links: Services, Why Us, Reviews, Contact; "Get a Free Quote" button (desktop), collapses gracefully on mobile.
3. **Hero** — two-column split. Left: eyebrow badge "Trusted by 500+ Chandpur families", H1 "Professional Cleaning in Chandpur", supporting sentence, primary CTA "Get a Free Quote" (scrolls to form), secondary CTA "Call Now" (tel: link). Trust chips row: "Bonded & Insured", "Eco-Friendly Products", "100% Satisfaction Guarantee". Right: stock photo of happy family in a clean living room with rounded corners and decorative teal blob accent.
4. **Services** — light grey background, section title + subtitle, three cards:
   - Standard Clean: weekly/biweekly maintenance — dusting, vacuuming, kitchen & baths, trash
   - Deep Clean: seasonal/top-to-bottom — inside appliances, baseboards, grout, windows
   - Move-out Clean: deposit-back guarantee focused — full appliance deep clean, walls, closets
   - Each card: icon, title, description, checkmark feature list (teal check icons), "starting at" price hint, CTA link to form with service preselected (via query/hash).
5. **Trust signals** — two-part section:
   - Insurance partner logos row (styled SVG/text placeholder logos labeled as partners)
   - Google review card: 5 star icons, "4.9 out of 5" average, review count, short testimonial quotes (2-3) with names.
6. **Contact form** — id="quote". Fields: Full name (required), Service address (required), Preferred date (date input), Preferred time (select: Morning/Afternoon/Evening), Phone (optional), service type select (preselectable). Submit: builds `mailto:` link with form contents AND shows inline success confirmation. Client-side validation with error messages.
7. **Footer** — logo, phone, service area "Serving Chandpur & surrounding areas", hours, copyright.

## Conversion Details

- Sticky mobile call button (floating phone icon on small screens).
- All CTAs scroll to #quote form.
- Service card CTA preselects that service in the form dropdown.
- tel: links use a placeholder number `+880 1712-345678` clearly marked in code for replacement.

## Technical Notes

- Single file: `index.html` (embedded `<style>` and `<script>`).
- Responsive: CSS grid/flex, breakpoints at 768px and 480px.
- Accessibility: semantic landmarks, labels on all inputs, focus styles, contrast-checked palette, `aria` where needed.
- No external JS dependencies. Hero image via stock-photo URL (Unsplash) with graceful background fallback.
- Form does not persist data (static site); mailto + confirmation only.

## Out of Scope

- Backend form handling, database, analytics, maps embed, i18n.
