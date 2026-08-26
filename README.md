# Natch Diamonds — Shopify Theme

A custom Shopify theme for Natch Diamonds, forked from [Shopify Dawn](https://github.com/Shopify/dawn). Minimalist, editorial luxury Maison direction (think Messika, Cartier, Van Cleef & Arpels) — not a typical e-commerce catalogue look.

## Local development

1. Install the Shopify CLI (already done in this environment: `npm install -g @shopify/cli`).
2. Connect this folder to a Shopify store — a development store is fine for building against before the client's real store is ready:
   ```sh
   shopify theme dev --store=your-store.myshopify.com
   ```
   The first run opens a browser to authenticate with your Shopify Partner/store account. This step needs a human in the loop — the CLI login is interactive and can't be scripted.
3. `theme dev` serves a live preview URL and hot-reloads Liquid/CSS/JS changes as you edit.

## Linting

```sh
shopify theme check
```

Run this before committing. It catches broken Liquid/schema references, missing translation keys, and unused assets.

## Project status

- [x] Dawn cloned, detached from upstream history, fresh repo initialized
- [x] Shopify CLI installed
- [x] Blog/article templates removed (not needed for this brand)
- [x] Section 1 — Hero (`sections/hero.liquid`, `assets/section-hero.css`)
- [x] Three Universes intro (Collection / Bespoke / Sourcing)
- [x] Featured Collection showcase (editorial, not a grid)
- [x] Bespoke journey (5-step process)
- [x] Diamond Sourcing landing (Private / Professional paths)
- [x] Maison/About section
- [x] Footer (client care, shipping/returns, privacy, terms, language selector)
- [x] Bespoke + Sourcing intake form templates
- [ ] Further Dawn strip-down (B2B quick-order, local pickup — currently left in place, dormant/opt-in, low risk)
- [x] Design token pass (color schemes, type scale) tuned to the brand palette

Maison and Bespoke Journey sections ship with `[CLIENT COPY NEEDED]` placeholder text — no source material for brand history or the specific 5-step process exists yet. Replace before launch. French translations for the two intake forms are in `locales/fr.json`; publishing French as a second store language (required for the footer's language selector to appear) happens in Shopify Admin, not in theme code.

## Structure notes

- Custom sections live in `sections/` alongside Dawn's originals; each ships its own `assets/section-<name>.css` following Dawn's per-section stylesheet convention.
- Design tokens (colors, spacing, type scale) are driven by Dawn's existing CSS custom-property system, generated in `layout/theme.liquid` from `config/settings_schema.json` / `config/settings_data.json` — extend those rather than hardcoding new one-off values where possible.
- Reveal/fade animations reuse Dawn's built-in `scroll-trigger` / `animate--fade-in` system (`assets/animations.js`), which already respects `prefers-reduced-motion` and the theme's "Animations" setting — no extra JS libraries needed.
