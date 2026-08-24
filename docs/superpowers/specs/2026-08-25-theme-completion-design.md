# Natch Diamonds Shopify Theme — Completion Design

Status: approved, 2026-08-25
Scope: close out every unchecked item in the README's "Project status" checklist.

## Context

This theme is a Dawn fork for the Natch Diamonds brand. Hero section and basic
strip-down are done; everything else on the storefront is still stock Dawn.

A separate Next.js repo (`Natch Diamonds`, sibling directory) is a further-along
demo of the same brand — same visual language, and it already has real content
for several of the sections this theme still needs: the "Three Houses" story
copy, working Bespoke/Sourcing form field sets, and the brand's color/type
tokens. This spec ports that content into native Dawn patterns rather than
inventing new copy.

The client's actual live store (`natchdiamonds.myshopify.com`) was reviewed as
a quality baseline. It is stock Dawn with a hero photo swapped in: one hero,
one 3-product grid (all sold out), an untranslated English newsletter block on
an otherwise French site, and zero animation or hover states anywhere. This
spec needs to clear that bar convincingly, not just technically.

## Design tokens (foundation)

Source: `tailwind.config.ts` and `app/layout.tsx` in the Next.js repo.

- Colors: ivory `#FAF8F3` (background), ink `#1A1A1A` (text/primary button),
  charcoal `#141414` (dark sections), gold `#B08D57` (accent/hover).
- Fonts: serif display (Cormorant Garamond in the Next.js app; use the closest
  match in Shopify's font picker, e.g. "Cormorant") for headings, Inter for
  body — set via `config/settings_schema.json` typography settings, not
  hardcoded CSS.

Changes:
- `config/settings_data.json`: replace `scheme-1` through `scheme-5` under
  `presets.Dawn.color_schemes` with Maison-toned equivalents. Keep 5 schemes
  (Dawn sections reference them by number) but retone them: e.g. scheme-1
  ivory bg / ink text (primary), scheme-2 white bg / ink text (cards),
  scheme-3 charcoal bg / ivory text (dark sections), scheme-4 ink bg / ivory
  text (hero — already used by `hero.liquid`), scheme-5 gold-tinted accent
  scheme for CTAs/highlights.
- `config/settings_schema.json` typography block: set default heading font to
  the serif pick, body font to Inter equivalent.
- This must land first — every other section is authored against these
  scheme numbers and font settings.

## Content language: bilingual EN/FR

The live store is French (Swiss market: "Livraison en Suisse"). The ported
Next.js copy is English. Decision: ship both, using Dawn's native locale
system — `locales/en.default.json` gets the new keys, `locales/fr.json` is a
new file with the same keys translated to French. All customer-facing copy in
new sections uses `{{ 'namespace.key' | t }}`, never hardcoded strings.

Admin-facing schema `name`/`label` fields (visible only in the theme editor,
not the storefront) stay hardcoded English, matching the precedent already
set in `hero.liquid`.

Dawn's language selector is already enabled (`footer.liquid` block setting
`enable_language_selector: true` in `footer-group.json`). It will not
actually show a picker until the store has a second language published in
Shopify Admin → Settings → Languages — that is a store-config step outside
this theme's code, and must happen before/alongside deploying this theme to
the real store. Note it as a deploy prerequisite, not a build blocker.

## New sections

All new sections follow the existing convention: Liquid file in `sections/`,
paired stylesheet in `assets/section-<name>.css` loaded via
`{{ 'section-<name>.css' | asset_url | stylesheet_tag }}`, schema with
`blocks` for editable content (matching `hero.liquid`'s block pattern), scroll
reveal via existing `scroll-trigger`/`animate--fade-in` classes (respects
`prefers-reduced-motion` and the theme's Animations setting already).

### `sections/three-universes.liquid`
Homepage, directly after Featured Collection. Three-column block, content
ported from `lib/scroll-story.ts` (`STORY_PANELS`) via translation keys:
numbered 01/02/03, "The Collection" / "Bespoke" / "Sourcing" with their body
copy, each linking out (Collection → `/collections/all`, Bespoke →
`/pages/bespoke`, Sourcing → `/pages/diamond-sourcing`). Each column staggers
in on scroll (reuse the stagger pattern already used for hero content, offset
per column).

### Featured Collection — editorial redesign
Not a new section file — restyle the existing `featured-collection.liquid`
usage in `templates/index.json` via a new `assets/section-featured-collection-editorial.css`
plus new settings on that section instance: asymmetric large-image layout
(first product larger, others smaller) instead of Dawn's uniform grid, using
existing Dawn card markup/classes so it stays swappable back to the stock
grid layout by just removing the stylesheet link and settings.

### `sections/maison.liquid`
Homepage, brand story teaser — image + heading + body text block, no
dedicated page (matches the Next.js site's single-page structure). Content:
placeholder brand-story copy clearly marked `[CLIENT COPY NEEDED]` in both
locale files — there is no source-of-truth "About" copy anywhere in either
repo, and this spec will not fabricate brand history as if it were real.

### `sections/bespoke-journey.liquid`
Used on `pages/bespoke`. Five-step process block (numbered steps, each with
heading + short description). Content: placeholder step copy marked
`[CLIENT COPY NEEDED]` — the README specifies "5-step process" but no source
describes what the 5 steps are; inventing specific steps (consultation,
design, sourcing, crafting, delivery — the generic bespoke-jewelry sequence)
as a reasonable placeholder is fine, but it must be flagged for client review
rather than presented as final copy.

### `sections/sourcing-landing.liquid`
Used on `pages/diamond-sourcing`. Two-path block: "Private Client" /
"Trade & Professional", copy adapted from `SourcingForm.tsx`'s existing
buyer-type framing (private vs. trade, natural vs. lab-grown) — this one has
real source material, unlike Maison/Bespoke-journey.

## Intake forms

No backend exists in this theme, and wiring cross-origin calls to the
Next.js repo's Server Actions would require adding new API routes there —
out of scope for completing this theme. Use Shopify's native
`{% form 'contact' %}` (same pattern as the existing `contact-form.liquid`),
extended with extra named fields following the `contact[Field Label]`
convention Dawn already uses.

### `sections/bespoke-form.liquid` (on `pages/bespoke`, below bespoke-journey)
Fields: Name, Email (Dawn's built-in contact fields), plus
`contact[Piece Description]` (textarea). No photo upload — Shopify's native
contact form has no attachment support without an app; drop that field for
v1, documented as a known gap.

### `sections/sourcing-form.liquid` (on `pages/diamond-sourcing`, below sourcing-landing)
Fields: Name, Email, plus `contact[Buyer Type]` (private/trade — rendered as
two toggle buttons driving a hidden input, mirroring `SourcingForm.tsx`'s
UI), `contact[Company Name]` (shown conditionally when Trade is selected —
CSS-only show/hide, no JS required since Liquid can't conditionally render
post-submit state anyway), `contact[Diamond Interest]` (select: Natural /
Lab-grown / Both), `contact[Details]` (textarea).

Both forms submit to the store's Shopify contact notification email — zero
new backend.

## Pages

- `templates/page.bespoke.json`: sections in order `bespoke-journey`,
  `bespoke-form`.
- `templates/page.diamond-sourcing.json`: sections in order
  `sourcing-landing`, `sourcing-form`.
- Corresponding `pages/bespoke` and `pages/diamond-sourcing` page records are
  a store-content step (Shopify Admin → Online Store → Pages, assigning the
  `page.bespoke` / `page.diamond-sourcing` template) — theme code ships the
  templates; creating the actual page records happens at store setup, same
  as the existing `page.contact.json` pattern already in this theme.

## Footer

`sections/footer-group.json`'s `footer` section currently has empty
`blocks`/`block_order`. Add:
- A `link_list` block pointing at a new "Client Care" menu (Shipping &
  Returns, Privacy Policy, Terms of Service — Dawn auto-populates
  Privacy/Terms/Shipping from the store's Shopify Admin → Settings →
  Policies once those are filled in there; the theme just needs
  `show_policy: true`, already set).
- A `text` block for the brand line for use with the existing default preset.
- Language selector: already enabled via `enable_language_selector: true`;
  no theme change needed beyond the store-language-publish prerequisite
  noted above.

## Homepage assembly

`templates/index.json` order becomes: `hero` → `featured_collection`
(editorial) → `three_universes` → `maison`. (Bespoke journey and Sourcing
landing live on their own pages, not the homepage, per the README calling
Diamond Sourcing specifically a "landing" page.)

## Explicitly out of scope

- Dawn strip-down (B2B quick-order list, local pickup) — README already
  flags this as dormant/opt-in/low-risk either way; leaving it in place.
- Fixing the live store's sold-out inventory — a merchandising decision for
  the client, not a theme concern.
- Actually publishing French as a second store language — store admin
  config, not theme code (see Content language section above).

## Testing / verification

- `shopify theme check` after every section is added — catches broken
  Liquid/schema refs, missing translation keys, unused assets (per this
  repo's own README).
- Visual verification via `shopify theme dev` once connected to a dev store
  — Liquid needs Shopify's backend for product/cart data, so nothing here
  can be verified by static file inspection alone. Requires the interactive
  CLI login step called out when this was scoped.
- Confirm `prefers-reduced-motion` still suppresses all new scroll-reveal
  animation, matching the theme's existing 0-JS-library animation approach.

## Build order

1. Design tokens (color schemes, typography)
2. `locales/fr.json` scaffold (empty-but-structured, filled in as each
   section's copy is written)
3. Three Universes
4. Featured Collection editorial restyle
5. Maison
6. Bespoke journey + Bespoke form + `page.bespoke.json`
7. Sourcing landing + Sourcing form + `page.diamond-sourcing.json`
8. Footer blocks
9. `templates/index.json` reassembly
10. `shopify theme check` pass across everything
