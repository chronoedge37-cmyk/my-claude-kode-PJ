# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Static site for a LINE Official Account fortune-telling business (五行庵 宗玄 / "五行タロット鑑定" — Five Elements × Tarot readings). No build system, no package manager, no backend/server — three self-contained HTML files, each with inline `<style>` and `<script>`, deployed as-is to Netlify.

There are no build, lint, or test commands — there is no `package.json` and nothing to compile. Verify changes by opening the HTML file directly in a browser (or the deployed Netlify URL) and exercising the relevant flow manually.

## Pages and user flow

- `intro-landing.html` — static entry page. Links to `muryo-soudan.html`.
- `muryo-soudan.html` — free consultation / self-service diagnosis. Customer enters birth date (+ optional time) and gender; a client-side 四柱推命 (Four Pillars / bazi) engine computes and renders the chart entirely in the browser. Ends with a CTA linking out to a hardcoded Stripe Payment Link for the paid reading.
- `260805.html` — the paid-reading intake form **and** the practitioner's working page, in one file. Behavior is selected at runtime by a `?mode=` query param, dispatched at the bottom of the `<script>`:
  - `?mode=customer` (default) — customer fills in name/birth data/request/desired timeframe. On submit, the data is base64-encoded into a URL (`enc()`/`dec()` helpers) and handed to the practitioner via a LINE share link (`https://line.me/R/share?text=...`). There is no backend — the encoded URL *is* the data transport.
  - `?mode=reading&d=<encoded>` — practitioner-only view. Decodes the payload, re-runs the same bazi engine plus 大運 (dayun/decade luck cycles), and provides textareas for the practitioner's commentary. "報告書を生成する" assembles everything into a plain-text report block for copy/paste.
  - `?mode=setup` — stores EmailJS credentials (public key, service ID, template ID, notification email) in `localStorage` under `ej_cfg`. When configured, `submitCustomer()` also fires an EmailJS notification as a backup channel in case the customer forgets to tap the LINE share button — this is best-effort and never blocks the LINE flow.

## Architecture notes

- **The bazi/four-pillars calculation engine is duplicated verbatim in `muryo-soudan.html` and `260805.html`** (solar longitude `sl()`, jieqi solar-term root-finding `flj()`, ganzhi conversion, `bazi()`, `tally()`, five-element weighting, etc.). There is no shared JS module — because there's no build step to share code between static pages, any fix or change to this engine must be applied in both files or they will silently diverge. `260805.html` additionally has the `dayun()` (大運) function that `muryo-soudan.html` doesn't need.
- All customer PII (name, birth date/time, gender, request, timeframe) flows only through the browser: form → base64(JSON) in a URL query param → LINE share text / EmailJS template params. Nothing is persisted server-side; `localStorage` on the practitioner's own device is the only persistence (EmailJS config only).
- External dependencies are all CDN-loaded, no bundler: Google Fonts (Cinzel, Shippori Mincho B1, Shippori Antique B1) on all three pages, and `@emailjs/browser` (jsdelivr CDN) on `260805.html` only.
- Visual language is shared but not code-shared across the three files: dark ink background, gold/lavender palette, "flowing star" canvas-free CSS animation. Each page defines its own copy of these styles.
