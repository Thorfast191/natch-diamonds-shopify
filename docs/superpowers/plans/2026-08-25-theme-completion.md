# Natch Diamonds Shopify Theme Completion — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Close out every unchecked item in this theme's README "Project status" checklist — design tokens, five new storefront sections, two intake-form pages, footer content, and homepage assembly — so the theme reads as a finished Maison storefront, not a Dawn default install.

**Architecture:** Pure Shopify Dawn theme conventions — Liquid sections with paired `assets/section-<name>.css` stylesheets, block-based schemas for merchant-editable content, Dawn's existing `scroll-trigger`/`animate--fade-in` reveal system (no new JS), native `{% form 'contact' %}` for the two intake forms (no backend), and `{{ 'key' | t }}` translation keys only for fixed UI chrome the theme hardcodes (not for merchant-editable section copy, which Shopify's Translate & Adapt app handles at the store level once French is published — see Global Constraints).

**Tech Stack:** Shopify Liquid, Shopify CLI (`theme check` for verification), vanilla CSS. No new dependencies.

**Spec:** `docs/superpowers/specs/2026-08-25-theme-completion-design.md`

## Global Constraints

- Every new/changed `.liquid` and `.json` file must pass `shopify theme check` with zero new errors before that task is committed.
- No JavaScript libraries or frameworks — reuse Dawn's existing `scroll-trigger`/`animate--fade-in` system (`assets/animations.js`) for reveal animation; it already respects `prefers-reduced-motion` and the theme's Animations setting.
- Color values: ivory `#FAF8F3`, ink `#1A1A1A`, charcoal `#141414`, gold `#B08D57` (from `tailwind.config.ts` in the sibling Next.js repo). Never hardcode these in a new CSS file — reference the color-scheme system (`color-scheme-N` wrapper classes) established in Task 1, exactly as `hero.liquid` and Dawn's stock sections already do.
- **Two different i18n mechanisms — do not conflate them:**
  1. Merchant-editable section copy (headings/body text entered via the theme editor, e.g. Three Universes panel text, Maison story) uses schema settings of type `text`/`richtext`/`inline_richtext`. This content is translated by Shopify's free "Translate & Adapt" app once French is published as a second store language — **no `locales/*.json` entries needed for this**, and no theme code handles it.
  2. Fixed UI chrome the theme hardcodes and merchants can't edit (form field labels, button text, success/error messages — mirroring how `contact-form.liquid` already uses `'templates.contact.form.name' | t`) uses `{{ 'namespace.key' | t }}` and **does** need matching entries added to both `locales/en.default.json` and `locales/fr.json`. This only applies to the two intake forms (Tasks 5 and 6).
- Dawn's footer language selector is already enabled (`enable_language_selector: true` in `footer-group.json`). It stays invisible until French is published as a second language in Shopify Admin → Settings → Languages — a store-config step, not something any task here can complete, since this theme isn't connected to a live store. Do not treat this as a blocker for any task.
- Admin-only schema `"name"`/`"label"` fields stay hardcoded English (matches the precedent already set in `sections/hero.liquid`) — only storefront-facing strings go through translation keys.

## Setup (one-time, before Task 1)

Install the Shopify CLI so `shopify theme check` is available. This is a pure static analyzer — it does **not** require a connected store, so every task below can be verified in this environment without the interactive `shopify theme dev` login.

```bash
npm install -g @shopify/cli
shopify version
```

Expected: prints a version number (e.g. `3.x.x`).

---

### Task 1: Design tokens — color schemes and typography

**Files:**
- Modify: `config/settings_data.json` (`presets.Dawn.color_schemes`, `presets.Dawn.type_header_font`, `presets.Dawn.type_body_font`)
- Modify: `config/settings_schema.json` (the two `font_picker` settings' `"default"` values, so a fresh install of this theme also starts on-brand)

**Interfaces:**
- Produces: `scheme-1` (ivory/ink, primary), `scheme-2` (white/ink, cards), `scheme-3` (charcoal/ivory, dark sections), `scheme-4` (ink/ivory, already used by the hero), `scheme-5` (gold/ink, accent) — every later task's `color_scheme` setting picks one of these five by name.

- [ ] **Step 1: Replace the five color schemes**

In `config/settings_data.json`, replace the `"color_schemes"` object (currently `scheme-1` through `scheme-5` with Dawn's stock grayscale/blue values) with:

```json
"color_schemes": {
  "scheme-1": {
    "settings": {
      "background": "#FAF8F3",
      "background_gradient": "",
      "text": "#1A1A1A",
      "button": "#1A1A1A",
      "button_label": "#FAF8F3",
      "secondary_button_label": "#1A1A1A",
      "shadow": "#1A1A1A"
    }
  },
  "scheme-2": {
    "settings": {
      "background": "#FFFFFF",
      "background_gradient": "",
      "text": "#1A1A1A",
      "button": "#1A1A1A",
      "button_label": "#FFFFFF",
      "secondary_button_label": "#1A1A1A",
      "shadow": "#1A1A1A"
    }
  },
  "scheme-3": {
    "settings": {
      "background": "#141414",
      "background_gradient": "",
      "text": "#FAF8F3",
      "button": "#FAF8F3",
      "button_label": "#141414",
      "secondary_button_label": "#FAF8F3",
      "shadow": "#141414"
    }
  },
  "scheme-4": {
    "settings": {
      "background": "#1A1A1A",
      "background_gradient": "",
      "text": "#FAF8F3",
      "button": "#FAF8F3",
      "button_label": "#1A1A1A",
      "secondary_button_label": "#FAF8F3",
      "shadow": "#1A1A1A"
    }
  },
  "scheme-5": {
    "settings": {
      "background": "#B08D57",
      "background_gradient": "",
      "text": "#1A1A1A",
      "button": "#1A1A1A",
      "button_label": "#B08D57",
      "secondary_button_label": "#1A1A1A",
      "shadow": "#1A1A1A"
    }
  }
}
```

- [ ] **Step 2: Switch the typography preset values**

In the same file, under `presets.Dawn`, change:
```json
"type_header_font": "cormorant_n4",
"type_body_font": "inter_n4",
```
(both currently `"assistant_n4"`).

- [ ] **Step 3: Update the schema fallback defaults**

In `config/settings_schema.json`, find the two `font_picker` settings (`type_header_font`, `type_body_font`, both currently `"default": "assistant_n4"`) and change their `"default"` to `"cormorant_n4"` and `"inter_n4"` respectively, matching Step 2.

- [ ] **Step 4: Verify**

```bash
shopify theme check
```
Expected: no new errors (pre-existing warnings, if any, are unaffected — only confirm nothing new appears referencing `config/settings_data.json` or `config/settings_schema.json`).

- [ ] **Step 5: Commit**

```bash
git add config/settings_data.json config/settings_schema.json
git commit -m "Retone color schemes and typography to the Maison brand palette"
```

---

### Task 2: Three Universes section

**Files:**
- Create: `sections/three-universes.liquid`
- Create: `assets/section-three-universes.css`

**Interfaces:**
- Consumes: `scheme-2` from Task 1.
- Produces: section type `three-universes`, block type `universe` (settings: `heading`, `body`, `link`, `link_label`) — Task 8 places this section on the homepage.

- [ ] **Step 1: Create the section file**

`sections/three-universes.liquid`:
```liquid
{{ 'section-three-universes.css' | asset_url | stylesheet_tag }}

{%- style -%}
  .section-{{ section.id }}-padding {
    padding-top: {{ section.settings.padding_top | times: 0.75 | round: 0 }}px;
    padding-bottom: {{ section.settings.padding_bottom | times: 0.75 | round: 0 }}px;
  }

  @media screen and (min-width: 750px) {
    .section-{{ section.id }}-padding {
      padding-top: {{ section.settings.padding_top }}px;
      padding-bottom: {{ section.settings.padding_bottom }}px;
    }
  }
{%- endstyle -%}

<div class="color-{{ section.settings.color_scheme }} gradient">
  <div class="three-universes page-width section-{{ section.id }}-padding">
    {%- if section.settings.eyebrow != blank -%}
      <p class="three-universes__eyebrow">{{ section.settings.eyebrow }}</p>
    {%- endif -%}
    <div class="three-universes__grid">
      {%- for block in section.blocks -%}
        {%- assign padded_index = forloop.index | prepend: '0' -%}
        <a
          href="{{ block.settings.link | default: '#' }}"
          class="three-universes__panel{% if settings.animations_reveal_on_scroll %} scroll-trigger animate--fade-in{% endif %}"
          style="--animation-order: {{ forloop.index0 }};"
          {{ block.shopify_attributes }}
        >
          <span class="three-universes__number" aria-hidden="true">{{ padded_index | slice: -2, 2 }}</span>
          <h3 class="three-universes__title">{{ block.settings.heading }}</h3>
          <div class="three-universes__body">{{ block.settings.body }}</div>
          {%- if block.settings.link_label != blank -%}
            <span class="three-universes__link-label">{{ block.settings.link_label }}</span>
          {%- endif -%}
        </a>
      {%- endfor -%}
    </div>
  </div>
</div>

{% schema %}
{
  "name": "Three Universes",
  "tag": "section",
  "class": "section",
  "settings": [
    {
      "type": "text",
      "id": "eyebrow",
      "label": "Eyebrow text",
      "default": "Three Houses, One Vision"
    },
    {
      "type": "color_scheme",
      "id": "color_scheme",
      "label": "Color scheme",
      "default": "scheme-2"
    },
    {
      "type": "header",
      "content": "Section padding"
    },
    {
      "type": "range",
      "id": "padding_top",
      "min": 0,
      "max": 100,
      "step": 4,
      "unit": "px",
      "label": "Padding top",
      "default": 72
    },
    {
      "type": "range",
      "id": "padding_bottom",
      "min": 0,
      "max": 100,
      "step": 4,
      "unit": "px",
      "label": "Padding bottom",
      "default": 72
    }
  ],
  "blocks": [
    {
      "type": "universe",
      "name": "Universe",
      "settings": [
        {
          "type": "text",
          "id": "heading",
          "label": "Heading",
          "default": "The Collection"
        },
        {
          "type": "richtext",
          "id": "body",
          "label": "Body text",
          "default": "<p>Ready-to-order pieces, designed once, cut precisely, available now.</p>"
        },
        {
          "type": "url",
          "id": "link",
          "label": "Link"
        },
        {
          "type": "text",
          "id": "link_label",
          "label": "Link label",
          "default": "Explore"
        }
      ]
    }
  ],
  "max_blocks": 3,
  "presets": [
    {
      "name": "Three Universes",
      "blocks": [
        {
          "type": "universe",
          "settings": {
            "heading": "The Collection",
            "body": "<p>Ready-to-order pieces from The Studs, The Hoops, and The Tennis — designed once, cut precisely, available now.</p>",
            "link": "shopify://collections/all",
            "link_label": "Explore the Collection"
          }
        },
        {
          "type": "universe",
          "settings": {
            "heading": "Bespoke",
            "body": "<p>A piece built around one idea: yours. Guided from the first sketch to the final polish.</p>",
            "link": "shopify://pages/bespoke",
            "link_label": "Begin a Bespoke Piece"
          }
        },
        {
          "type": "universe",
          "settings": {
            "heading": "Sourcing",
            "body": "<p>Natural or lab-grown, private client or trade buyer — stones sourced and verified to the specification you set.</p>",
            "link": "shopify://pages/diamond-sourcing",
            "link_label": "Start Sourcing"
          }
        }
      ]
    }
  ]
}
{% endschema %}
```

- [ ] **Step 2: Create the stylesheet**

`assets/section-three-universes.css`:
```css
.three-universes__eyebrow {
  font-size: 0.75rem;
  letter-spacing: 0.3em;
  text-transform: uppercase;
  text-align: center;
  margin: 0 0 3rem;
}

.three-universes__grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 3rem;
}

@media screen and (min-width: 750px) {
  .three-universes__grid {
    grid-template-columns: repeat(3, 1fr);
    gap: 2rem;
  }
}

.three-universes__panel {
  display: block;
  text-decoration: none;
  color: currentColor;
}

.three-universes__number {
  display: block;
  font-family: var(--font-heading-family);
  font-size: 4rem;
  line-height: 1;
  opacity: 0.12;
}

.three-universes__title {
  margin: 1rem 0 0.75rem;
  font-family: var(--font-heading-family);
}

.three-universes__body {
  opacity: 0.75;
}

.three-universes__body p {
  margin: 0;
}

.three-universes__link-label {
  display: inline-block;
  margin-top: 1rem;
  font-size: 0.75rem;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  border-bottom: 1px solid currentColor;
  padding-bottom: 2px;
  transition: opacity 0.2s ease;
}

.three-universes__panel:hover .three-universes__link-label {
  opacity: 0.6;
}
```

- [ ] **Step 3: Verify**

```bash
shopify theme check
```
Expected: no new errors.

- [ ] **Step 4: Commit**

```bash
git add sections/three-universes.liquid assets/section-three-universes.css
git commit -m "Add Three Universes section (Collection / Bespoke / Sourcing)"
```

---

### Task 3: Featured Collection editorial restyle

**Files:**
- Modify: `sections/featured-collection.liquid:1` (stylesheet includes) and `:99` (product grid `<ul>` class attribute)
- Create: `assets/section-featured-collection-editorial.css`

**Interfaces:**
- Consumes: `.grid.product-grid` / `.grid__item` markup already produced by `featured-collection.liquid` — this task only adds a modifier class and a schema toggle, it does not change the Liquid loop structure.
- Produces: `section.settings.editorial_layout` (checkbox) — Task 8 leaves this at its default (`true`) when placing the section on the homepage.

- [ ] **Step 1: Add the stylesheet include**

At the top of `sections/featured-collection.liquid` (currently starts with `{{ 'component-card.css' | asset_url | stylesheet_tag }}` on line 1), add immediately after the existing stylesheet includes (after the `template-collection.css` line, before the `image_shape == 'blob'` block):

```liquid
{%- if section.settings.editorial_layout -%}
  {{ 'section-featured-collection-editorial.css' | asset_url | stylesheet_tag }}
{%- endif -%}
```

- [ ] **Step 2: Add the modifier class to the grid**

Find the `<ul>` element (currently around line 96-99):
```liquid
class="grid product-grid contains-card contains-card--product{% if settings.card_style == 'standard' %} contains-card--standard{% endif %} grid--{{ section.settings.columns_desktop }}-col-desktop{% if section.settings.collection == blank %} grid--2-col-tablet-down{% else %} grid--{{ section.settings.columns_mobile }}-col-tablet-down{% endif %}{% if show_mobile_slider or show_desktop_slider %} slider{% if show_desktop_slider %} slider--desktop{% endif %}{% if show_mobile_slider %} slider--tablet grid--peek{% endif %}{% endif %}"
```

Change it to prepend the editorial modifier:
```liquid
class="grid product-grid{% if section.settings.editorial_layout %} product-grid--editorial{% endif %} contains-card contains-card--product{% if settings.card_style == 'standard' %} contains-card--standard{% endif %} grid--{{ section.settings.columns_desktop }}-col-desktop{% if section.settings.collection == blank %} grid--2-col-tablet-down{% else %} grid--{{ section.settings.columns_mobile }}-col-tablet-down{% endif %}{% if show_mobile_slider or show_desktop_slider %} slider{% if show_desktop_slider %} slider--desktop{% endif %}{% if show_mobile_slider %} slider--tablet grid--peek{% endif %}{% endif %}"
```

- [ ] **Step 3: Add the schema setting**

In the `{% schema %}` block's `"settings"` array, add (near the top, after the `"title"` setting):
```json
{
  "type": "checkbox",
  "id": "editorial_layout",
  "label": "Use editorial layout",
  "info": "Large first product, smaller others, instead of a uniform grid.",
  "default": true
}
```

- [ ] **Step 4: Create the editorial stylesheet**

`assets/section-featured-collection-editorial.css`:
```css
@media screen and (min-width: 750px) {
  .product-grid--editorial {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    grid-auto-rows: auto;
    gap: 2rem;
  }

  .product-grid--editorial .grid__item:first-child {
    grid-column: span 2;
    grid-row: span 2;
  }

  .product-grid--editorial .grid__item:first-child .card__media {
    aspect-ratio: 1 / 1.15;
  }

  .product-grid--editorial .grid__item:not(:first-child) .card__media {
    aspect-ratio: 3 / 4;
  }
}
```

- [ ] **Step 5: Verify**

```bash
shopify theme check
```
Expected: no new errors.

- [ ] **Step 6: Commit**

```bash
git add sections/featured-collection.liquid assets/section-featured-collection-editorial.css
git commit -m "Add editorial layout option to Featured Collection"
```

---

### Task 4: Maison section

**Files:**
- Create: `sections/maison.liquid`
- Create: `assets/section-maison.css`

**Interfaces:**
- Consumes: `scheme-1` from Task 1.
- Produces: section type `maison` — Task 8 places this on the homepage.

- [ ] **Step 1: Create the section file**

`sections/maison.liquid`:
```liquid
{{ 'section-maison.css' | asset_url | stylesheet_tag }}

{%- style -%}
  .section-{{ section.id }}-padding {
    padding-top: {{ section.settings.padding_top | times: 0.75 | round: 0 }}px;
    padding-bottom: {{ section.settings.padding_bottom | times: 0.75 | round: 0 }}px;
  }

  @media screen and (min-width: 750px) {
    .section-{{ section.id }}-padding {
      padding-top: {{ section.settings.padding_top }}px;
      padding-bottom: {{ section.settings.padding_bottom }}px;
    }
  }
{%- endstyle -%}

<div class="color-{{ section.settings.color_scheme }} gradient">
  <div class="maison page-width section-{{ section.id }}-padding">
    <div class="maison__grid">
      <div class="maison__media{% if settings.animations_reveal_on_scroll %} scroll-trigger animate--fade-in{% endif %}">
        {%- if section.settings.image != blank -%}
          {{
            section.settings.image
            | image_url: width: 1200
            | image_tag: loading: 'lazy', sizes: '(min-width: 750px) 50vw, 100vw', widths: '400,600,900,1200'
          }}
        {%- else -%}
          {{ 'lifestyle-1' | placeholder_svg_tag: 'maison__media-placeholder placeholder' }}
        {%- endif -%}
      </div>
      <div class="maison__content{% if settings.animations_reveal_on_scroll %} scroll-trigger animate--fade-in{% endif %}">
        {%- if section.settings.eyebrow != blank -%}
          <p class="maison__eyebrow">{{ section.settings.eyebrow }}</p>
        {%- endif -%}
        {%- if section.settings.heading != blank -%}
          <h2 class="maison__heading">{{ section.settings.heading }}</h2>
        {%- endif -%}
        <div class="maison__body">{{ section.settings.body }}</div>
        {%- if section.settings.link != blank and section.settings.link_label != blank -%}
          <a href="{{ section.settings.link }}" class="maison__link-label">{{ section.settings.link_label }}</a>
        {%- endif -%}
      </div>
    </div>
  </div>
</div>

{% schema %}
{
  "name": "Maison",
  "tag": "section",
  "class": "section",
  "settings": [
    {
      "type": "image_picker",
      "id": "image",
      "label": "Image"
    },
    {
      "type": "text",
      "id": "eyebrow",
      "label": "Eyebrow text",
      "default": "The Maison"
    },
    {
      "type": "text",
      "id": "heading",
      "label": "Heading",
      "default": "[CLIENT COPY NEEDED] A brand story heading"
    },
    {
      "type": "richtext",
      "id": "body",
      "label": "Body text",
      "default": "<p>[CLIENT COPY NEEDED] No brand-history source material exists yet in either the Shopify theme or the Next.js demo — replace this placeholder with the client's actual story before launch.</p>"
    },
    {
      "type": "url",
      "id": "link",
      "label": "Link"
    },
    {
      "type": "text",
      "id": "link_label",
      "label": "Link label",
      "default": "Our Story"
    },
    {
      "type": "color_scheme",
      "id": "color_scheme",
      "label": "Color scheme",
      "default": "scheme-1"
    },
    {
      "type": "header",
      "content": "Section padding"
    },
    {
      "type": "range",
      "id": "padding_top",
      "min": 0,
      "max": 100,
      "step": 4,
      "unit": "px",
      "label": "Padding top",
      "default": 72
    },
    {
      "type": "range",
      "id": "padding_bottom",
      "min": 0,
      "max": 100,
      "step": 4,
      "unit": "px",
      "label": "Padding bottom",
      "default": 72
    }
  ],
  "presets": [
    {
      "name": "Maison"
    }
  ]
}
{% endschema %}
```

- [ ] **Step 2: Create the stylesheet**

`assets/section-maison.css`:
```css
.maison__grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 2rem;
  align-items: center;
}

@media screen and (min-width: 750px) {
  .maison__grid {
    grid-template-columns: 1fr 1fr;
    gap: 4rem;
  }
}

.maison__media img,
.maison__media svg {
  width: 100%;
  height: auto;
  aspect-ratio: 4 / 5;
  object-fit: cover;
}

.maison__eyebrow {
  font-size: 0.75rem;
  letter-spacing: 0.3em;
  text-transform: uppercase;
  margin: 0 0 1rem;
}

.maison__heading {
  font-family: var(--font-heading-family);
  margin: 0 0 1.5rem;
}

.maison__body {
  opacity: 0.75;
}

.maison__link-label {
  display: inline-block;
  margin-top: 1.5rem;
  font-size: 0.75rem;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  border-bottom: 1px solid currentColor;
  padding-bottom: 2px;
  text-decoration: none;
  color: currentColor;
}
```

- [ ] **Step 3: Verify**

```bash
shopify theme check
```
Expected: no new errors.

- [ ] **Step 4: Commit**

```bash
git add sections/maison.liquid assets/section-maison.css
git commit -m "Add Maison brand-story section (placeholder copy pending client input)"
```

---

### Task 5: Bespoke page (journey + intake form)

**Files:**
- Create: `sections/bespoke-journey.liquid`
- Create: `assets/section-bespoke-journey.css`
- Create: `sections/bespoke-form.liquid`
- Create: `assets/section-bespoke-form.css`
- Modify: `locales/en.default.json` (add `sections.bespoke_form.*` keys)
- Create: `locales/fr.json`
- Create: `templates/page.bespoke.json`

**Interfaces:**
- Consumes: `scheme-1`, `scheme-2` from Task 1; native `{% form 'contact' %}` pattern already established in `sections/contact-form.liquid`.
- Produces: section types `bespoke-journey`, `bespoke-form`; template `page.bespoke`; locale namespace `sections.bespoke_form`.

- [ ] **Step 1: Create the journey section**

`sections/bespoke-journey.liquid`:
```liquid
{{ 'section-bespoke-journey.css' | asset_url | stylesheet_tag }}

{%- style -%}
  .section-{{ section.id }}-padding {
    padding-top: {{ section.settings.padding_top | times: 0.75 | round: 0 }}px;
    padding-bottom: {{ section.settings.padding_bottom | times: 0.75 | round: 0 }}px;
  }

  @media screen and (min-width: 750px) {
    .section-{{ section.id }}-padding {
      padding-top: {{ section.settings.padding_top }}px;
      padding-bottom: {{ section.settings.padding_bottom }}px;
    }
  }
{%- endstyle -%}

<div class="color-{{ section.settings.color_scheme }} gradient">
  <div class="bespoke-journey page-width page-width--narrow section-{{ section.id }}-padding">
    {%- if section.settings.heading != blank -%}
      <h1 class="bespoke-journey__heading">{{ section.settings.heading }}</h1>
    {%- endif -%}
    {%- if section.settings.intro != blank -%}
      <div class="bespoke-journey__intro">{{ section.settings.intro }}</div>
    {%- endif -%}
    <ol class="bespoke-journey__steps">
      {%- for block in section.blocks -%}
        <li
          class="bespoke-journey__step{% if settings.animations_reveal_on_scroll %} scroll-trigger animate--fade-in{% endif %}"
          style="--animation-order: {{ forloop.index0 }};"
          {{ block.shopify_attributes }}
        >
          <span class="bespoke-journey__step-number" aria-hidden="true">{{ forloop.index }}</span>
          <h3 class="bespoke-journey__step-heading">{{ block.settings.heading }}</h3>
          <p class="bespoke-journey__step-body">{{ block.settings.body }}</p>
        </li>
      {%- endfor -%}
    </ol>
  </div>
</div>

{% schema %}
{
  "name": "Bespoke Journey",
  "tag": "section",
  "class": "section",
  "settings": [
    {
      "type": "text",
      "id": "heading",
      "label": "Heading",
      "default": "The Bespoke Journey"
    },
    {
      "type": "richtext",
      "id": "intro",
      "label": "Intro text",
      "default": "<p>Five steps from first idea to a piece that's entirely yours.</p>"
    },
    {
      "type": "color_scheme",
      "id": "color_scheme",
      "label": "Color scheme",
      "default": "scheme-1"
    },
    {
      "type": "header",
      "content": "Section padding"
    },
    {
      "type": "range",
      "id": "padding_top",
      "min": 0,
      "max": 100,
      "step": 4,
      "unit": "px",
      "label": "Padding top",
      "default": 72
    },
    {
      "type": "range",
      "id": "padding_bottom",
      "min": 0,
      "max": 100,
      "step": 4,
      "unit": "px",
      "label": "Padding bottom",
      "default": 36
    }
  ],
  "blocks": [
    {
      "type": "step",
      "name": "Step",
      "settings": [
        {
          "type": "text",
          "id": "heading",
          "label": "Heading",
          "default": "Consultation"
        },
        {
          "type": "textarea",
          "id": "body",
          "label": "Body text",
          "default": "[CLIENT COPY NEEDED] Describe this step of the bespoke process."
        }
      ]
    }
  ],
  "max_blocks": 5,
  "presets": [
    {
      "name": "Bespoke Journey",
      "blocks": [
        { "type": "step", "settings": { "heading": "Consultation", "body": "[CLIENT COPY NEEDED] We start with a conversation — your inspiration, budget, and timeline." } },
        { "type": "step", "settings": { "heading": "Design", "body": "[CLIENT COPY NEEDED] A sketch and a stone selection, refined until it's right." } },
        { "type": "step", "settings": { "heading": "Sourcing", "body": "[CLIENT COPY NEEDED] Your diamond is sourced and certified to specification." } },
        { "type": "step", "settings": { "heading": "Crafting", "body": "[CLIENT COPY NEEDED] The piece is hand-finished in the workshop." } },
        { "type": "step", "settings": { "heading": "Delivery", "body": "[CLIENT COPY NEEDED] Your piece arrives, ready to wear or gift." } }
      ]
    }
  ]
}
{% endschema %}
```

- [ ] **Step 2: Create the journey stylesheet**

`assets/section-bespoke-journey.css`:
```css
.bespoke-journey__heading {
  font-family: var(--font-heading-family);
  text-align: center;
}

.bespoke-journey__intro {
  text-align: center;
  opacity: 0.75;
  max-width: 40em;
  margin: 1rem auto 0;
}

.bespoke-journey__steps {
  list-style: none;
  margin: 3rem 0 0;
  padding: 0;
  display: grid;
  gap: 2.5rem;
}

.bespoke-journey__step-number {
  display: block;
  font-family: var(--font-heading-family);
  font-size: 2.5rem;
  line-height: 1;
  opacity: 0.3;
}

.bespoke-journey__step-heading {
  margin: 0.5rem 0 0.5rem;
}

.bespoke-journey__step-body {
  opacity: 0.75;
  margin: 0;
}
```

- [ ] **Step 3: Add the bespoke-form locale keys**

`locales/en.default.json` already has a top-level `"sections"` object (used by Dawn's stock sections, e.g. `"sections": { "contact-form": { ... }, ... }`). Add a new `"bespoke_form"` key as a sibling *inside* that existing `"sections"` object — do not create a second top-level `"sections"` key:
```json
"sections": {
  "bespoke_form": {
    "name_label": "Name",
    "email_label": "Email",
    "description_label": "Describe the piece you have in mind",
    "submit_label": "Submit Inquiry",
    "success_heading": "Thank you.",
    "success_body": "We've received your bespoke inquiry and will be in touch shortly."
  }
}
```
(This makes the full path `sections.bespoke_form.name_label`, matching what `bespoke-form.liquid` requests via `{{ 'sections.bespoke_form.name_label' | t }}`.)

- [ ] **Step 4: Create `locales/fr.json`**

```json
{
  "sections": {
    "bespoke_form": {
      "name_label": "Nom",
      "email_label": "E-mail",
      "description_label": "Décrivez la pièce que vous avez en tête",
      "submit_label": "Envoyer la demande",
      "success_heading": "Merci.",
      "success_body": "Nous avons bien reçu votre demande sur-mesure et vous répondrons sous peu."
    }
  }
}
```

- [ ] **Step 5: Create the bespoke form section**

`sections/bespoke-form.liquid`:
```liquid
{{ 'section-bespoke-form.css' | asset_url | stylesheet_tag }}

{%- style -%}
  .section-{{ section.id }}-padding {
    padding-top: {{ section.settings.padding_top | times: 0.75 | round: 0 }}px;
    padding-bottom: {{ section.settings.padding_bottom | times: 0.75 | round: 0 }}px;
  }

  @media screen and (min-width: 750px) {
    .section-{{ section.id }}-padding {
      padding-top: {{ section.settings.padding_top }}px;
      padding-bottom: {{ section.settings.padding_bottom }}px;
    }
  }
{%- endstyle -%}

<div class="color-{{ section.settings.color_scheme }} gradient">
  <div class="bespoke-form-section page-width page-width--narrow section-{{ section.id }}-padding">
    {%- form 'contact', id: 'BespokeForm', class: 'bespoke-form' -%}
      {%- if form.posted_successfully? -%}
        <h2 class="form-status bespoke-form__success" tabindex="-1" autofocus>
          {{ 'sections.bespoke_form.success_heading' | t }}
          <span class="bespoke-form__success-body">{{ 'sections.bespoke_form.success_body' | t }}</span>
        </h2>
      {%- else -%}
        <div class="field">
          <input
            type="text"
            id="BespokeForm-name"
            class="field__input"
            autocomplete="name"
            name="contact[{{ 'sections.bespoke_form.name_label' | t }}]"
            placeholder="{{ 'sections.bespoke_form.name_label' | t }}"
            required
          >
          <label class="field__label" for="BespokeForm-name">{{ 'sections.bespoke_form.name_label' | t }}</label>
        </div>
        <div class="field">
          <input
            type="email"
            id="BespokeForm-email"
            class="field__input"
            autocomplete="email"
            name="contact[email]"
            placeholder="{{ 'sections.bespoke_form.email_label' | t }}"
            required
          >
          <label class="field__label" for="BespokeForm-email">{{ 'sections.bespoke_form.email_label' | t }}</label>
        </div>
        <div class="field">
          <textarea
            id="BespokeForm-description"
            class="text-area field__input"
            rows="6"
            name="contact[{{ 'sections.bespoke_form.description_label' | t }}]"
            placeholder="{{ 'sections.bespoke_form.description_label' | t }}"
            required
          ></textarea>
          <label class="field__label" for="BespokeForm-description">{{ 'sections.bespoke_form.description_label' | t }}</label>
        </div>
        <div class="bespoke-form__submit">
          <button type="submit" class="button">{{ 'sections.bespoke_form.submit_label' | t }}</button>
        </div>
      {%- endif -%}
    {%- endform -%}
  </div>
</div>

{% schema %}
{
  "name": "Bespoke Form",
  "tag": "section",
  "class": "section",
  "settings": [
    {
      "type": "color_scheme",
      "id": "color_scheme",
      "label": "Color scheme",
      "default": "scheme-2"
    },
    {
      "type": "header",
      "content": "Section padding"
    },
    {
      "type": "range",
      "id": "padding_top",
      "min": 0,
      "max": 100,
      "step": 4,
      "unit": "px",
      "label": "Padding top",
      "default": 36
    },
    {
      "type": "range",
      "id": "padding_bottom",
      "min": 0,
      "max": 100,
      "step": 4,
      "unit": "px",
      "label": "Padding bottom",
      "default": 72
    }
  ],
  "presets": [
    {
      "name": "Bespoke Form"
    }
  ]
}
{% endschema %}
```

- [ ] **Step 6: Create the bespoke form stylesheet**

`assets/section-bespoke-form.css`:
```css
.bespoke-form .field {
  margin-bottom: 1.5rem;
}

.bespoke-form .field__input {
  width: 100%;
  border: none;
  border-bottom: 1px solid rgb(var(--color-foreground) / 0.2);
  background: transparent;
  padding: 0.75rem 0;
  font: inherit;
  color: currentColor;
}

.bespoke-form .field__input:focus {
  outline: none;
  border-bottom-color: currentColor;
}

.bespoke-form .field__label {
  display: block;
  font-size: 0.75rem;
  opacity: 0.6;
  margin-bottom: 0.25rem;
}

.bespoke-form__submit {
  margin-top: 1rem;
}

.bespoke-form__success {
  text-align: center;
  font-family: var(--font-heading-family);
}

.bespoke-form__success-body {
  display: block;
  margin-top: 0.75rem;
  font-family: var(--font-body-family);
  font-size: 1rem;
  opacity: 0.75;
}
```

- [ ] **Step 7: Create the page template**

`templates/page.bespoke.json`:
```json
{
  "sections": {
    "bespoke-journey": {
      "type": "bespoke-journey",
      "settings": {}
    },
    "bespoke-form": {
      "type": "bespoke-form",
      "settings": {}
    }
  },
  "order": [
    "bespoke-journey",
    "bespoke-form"
  ]
}
```

- [ ] **Step 8: Verify**

```bash
shopify theme check
```
Expected: no new errors. If `theme check` flags `MissingTranslation` for the new `sections.bespoke_form.*` keys used in `fr.json`, confirm both `locales/en.default.json` and `locales/fr.json` use the exact same key path — the two files must match.

- [ ] **Step 9: Commit**

```bash
git add sections/bespoke-journey.liquid assets/section-bespoke-journey.css sections/bespoke-form.liquid assets/section-bespoke-form.css locales/en.default.json locales/fr.json templates/page.bespoke.json
git commit -m "Add Bespoke journey section, intake form, and page template"
```

---

### Task 6: Diamond Sourcing page (landing + intake form)

**Files:**
- Create: `sections/sourcing-landing.liquid`
- Create: `assets/section-sourcing-landing.css`
- Create: `sections/sourcing-form.liquid`
- Create: `assets/section-sourcing-form.css`
- Modify: `locales/en.default.json` (add `sections.sourcing_form.*` keys)
- Modify: `locales/fr.json` (add matching `sections.sourcing_form.*` keys)
- Create: `templates/page.diamond-sourcing.json`

**Interfaces:**
- Consumes: `scheme-1`, `scheme-2` from Task 1; locale file structure from Task 5.
- Produces: section types `sourcing-landing`, `sourcing-form`; template `page.diamond-sourcing`; locale namespace `sections.sourcing_form`.

- [ ] **Step 1: Create the landing section**

`sections/sourcing-landing.liquid`:
```liquid
{{ 'section-sourcing-landing.css' | asset_url | stylesheet_tag }}

{%- style -%}
  .section-{{ section.id }}-padding {
    padding-top: {{ section.settings.padding_top | times: 0.75 | round: 0 }}px;
    padding-bottom: {{ section.settings.padding_bottom | times: 0.75 | round: 0 }}px;
  }

  @media screen and (min-width: 750px) {
    .section-{{ section.id }}-padding {
      padding-top: {{ section.settings.padding_top }}px;
      padding-bottom: {{ section.settings.padding_bottom }}px;
    }
  }
{%- endstyle -%}

<div class="color-{{ section.settings.color_scheme }} gradient">
  <div class="sourcing-landing page-width section-{{ section.id }}-padding">
    {%- if section.settings.heading != blank -%}
      <h1 class="sourcing-landing__heading">{{ section.settings.heading }}</h1>
    {%- endif -%}
    <div class="sourcing-landing__paths">
      {%- for block in section.blocks -%}
        <div
          class="sourcing-landing__path{% if settings.animations_reveal_on_scroll %} scroll-trigger animate--fade-in{% endif %}"
          style="--animation-order: {{ forloop.index0 }};"
          {{ block.shopify_attributes }}
        >
          <h3 class="sourcing-landing__path-heading">{{ block.settings.heading }}</h3>
          <p class="sourcing-landing__path-body">{{ block.settings.body }}</p>
        </div>
      {%- endfor -%}
    </div>
  </div>
</div>

{% schema %}
{
  "name": "Sourcing Landing",
  "tag": "section",
  "class": "section",
  "settings": [
    {
      "type": "text",
      "id": "heading",
      "label": "Heading",
      "default": "Diamond Sourcing"
    },
    {
      "type": "color_scheme",
      "id": "color_scheme",
      "label": "Color scheme",
      "default": "scheme-1"
    },
    {
      "type": "header",
      "content": "Section padding"
    },
    {
      "type": "range",
      "id": "padding_top",
      "min": 0,
      "max": 100,
      "step": 4,
      "unit": "px",
      "label": "Padding top",
      "default": 72
    },
    {
      "type": "range",
      "id": "padding_bottom",
      "min": 0,
      "max": 100,
      "step": 4,
      "unit": "px",
      "label": "Padding bottom",
      "default": 36
    }
  ],
  "blocks": [
    {
      "type": "path",
      "name": "Path",
      "settings": [
        {
          "type": "text",
          "id": "heading",
          "label": "Heading",
          "default": "Private Client"
        },
        {
          "type": "textarea",
          "id": "body",
          "label": "Body text",
          "default": "Natural or lab-grown stones, sourced and verified to your specification."
        }
      ]
    }
  ],
  "max_blocks": 2,
  "presets": [
    {
      "name": "Sourcing Landing",
      "blocks": [
        { "type": "path", "settings": { "heading": "Private Client", "body": "Natural or lab-grown, sourced and certified to the specification you set — for a single piece or a collection." } },
        { "type": "path", "settings": { "heading": "Trade & Professional", "body": "Wholesale sourcing for jewellers and retailers, natural and lab-grown, with certification on every stone." } }
      ]
    }
  ]
}
{% endschema %}
```

- [ ] **Step 2: Create the landing stylesheet**

`assets/section-sourcing-landing.css`:
```css
.sourcing-landing__heading {
  font-family: var(--font-heading-family);
  text-align: center;
}

.sourcing-landing__paths {
  display: grid;
  grid-template-columns: 1fr;
  gap: 2rem;
  margin-top: 3rem;
}

@media screen and (min-width: 750px) {
  .sourcing-landing__paths {
    grid-template-columns: 1fr 1fr;
    gap: 3rem;
  }
}

.sourcing-landing__path {
  border: 1px solid rgb(var(--color-foreground) / 0.15);
  padding: 2rem;
}

.sourcing-landing__path-heading {
  margin: 0 0 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  font-size: 1rem;
}

.sourcing-landing__path-body {
  opacity: 0.75;
  margin: 0;
}
```

- [ ] **Step 3: Add the sourcing-form locale keys**

In `locales/en.default.json`, add `"sourcing_form"` as a sibling of the `"bespoke_form"` key Task 5 added — both live inside the same top-level `"sections"` object:
```json
"sourcing_form": {
  "name_label": "Name",
  "email_label": "Email",
  "buyer_type_label": "Buyer type",
  "buyer_type_private": "Private Client",
  "buyer_type_trade": "Trade / Professional",
  "company_label": "Company name",
  "interest_label": "Diamond interest",
  "interest_natural": "Natural",
  "interest_lab_grown": "Lab-grown",
  "interest_both": "Both",
  "details_label": "Details",
  "submit_label": "Submit Inquiry",
  "success_heading": "Thank you.",
  "success_body": "Our sourcing team will follow up shortly."
}
```

In `locales/fr.json`, add the matching sibling key inside `sections`:
```json
"sourcing_form": {
  "name_label": "Nom",
  "email_label": "E-mail",
  "buyer_type_label": "Type de client",
  "buyer_type_private": "Client privé",
  "buyer_type_trade": "Professionnel / Négoce",
  "company_label": "Nom de l'entreprise",
  "interest_label": "Type de diamant recherché",
  "interest_natural": "Naturel",
  "interest_lab_grown": "De laboratoire",
  "interest_both": "Les deux",
  "details_label": "Détails",
  "submit_label": "Envoyer la demande",
  "success_heading": "Merci.",
  "success_body": "Notre équipe sourcing vous répondra sous peu."
}
```

- [ ] **Step 4: Create the sourcing form section**

`sections/sourcing-form.liquid`:
```liquid
{{ 'section-sourcing-form.css' | asset_url | stylesheet_tag }}

{%- style -%}
  .section-{{ section.id }}-padding {
    padding-top: {{ section.settings.padding_top | times: 0.75 | round: 0 }}px;
    padding-bottom: {{ section.settings.padding_bottom | times: 0.75 | round: 0 }}px;
  }

  @media screen and (min-width: 750px) {
    .section-{{ section.id }}-padding {
      padding-top: {{ section.settings.padding_top }}px;
      padding-bottom: {{ section.settings.padding_bottom }}px;
    }
  }
{%- endstyle -%}

<div class="color-{{ section.settings.color_scheme }} gradient">
  <div class="sourcing-form-section page-width page-width--narrow section-{{ section.id }}-padding">
    {%- form 'contact', id: 'SourcingForm', class: 'sourcing-form' -%}
      {%- if form.posted_successfully? -%}
        <h2 class="form-status sourcing-form__success" tabindex="-1" autofocus>
          {{ 'sections.sourcing_form.success_heading' | t }}
          <span class="sourcing-form__success-body">{{ 'sections.sourcing_form.success_body' | t }}</span>
        </h2>
      {%- else -%}
        <fieldset class="sourcing-form__buyer-type">
          <legend class="field__label">{{ 'sections.sourcing_form.buyer_type_label' | t }}</legend>
          <input type="radio" id="SourcingForm-private" name="contact[{{ 'sections.sourcing_form.buyer_type_label' | t }}]" value="{{ 'sections.sourcing_form.buyer_type_private' | t }}" checked>
          <label for="SourcingForm-private">{{ 'sections.sourcing_form.buyer_type_private' | t }}</label>
          <input type="radio" id="SourcingForm-trade" name="contact[{{ 'sections.sourcing_form.buyer_type_label' | t }}]" value="{{ 'sections.sourcing_form.buyer_type_trade' | t }}">
          <label for="SourcingForm-trade">{{ 'sections.sourcing_form.buyer_type_trade' | t }}</label>
        </fieldset>

        <div class="field">
          <input type="text" id="SourcingForm-name" class="field__input" autocomplete="name" name="contact[{{ 'sections.sourcing_form.name_label' | t }}]" placeholder="{{ 'sections.sourcing_form.name_label' | t }}" required>
          <label class="field__label" for="SourcingForm-name">{{ 'sections.sourcing_form.name_label' | t }}</label>
        </div>

        <div class="field">
          <input type="email" id="SourcingForm-email" class="field__input" autocomplete="email" name="contact[email]" placeholder="{{ 'sections.sourcing_form.email_label' | t }}" required>
          <label class="field__label" for="SourcingForm-email">{{ 'sections.sourcing_form.email_label' | t }}</label>
        </div>

        <div class="field sourcing-form__company">
          <input type="text" id="SourcingForm-company" class="field__input" name="contact[{{ 'sections.sourcing_form.company_label' | t }}]" placeholder="{{ 'sections.sourcing_form.company_label' | t }}">
          <label class="field__label" for="SourcingForm-company">{{ 'sections.sourcing_form.company_label' | t }}</label>
        </div>

        <div class="field">
          <label class="field__label" for="SourcingForm-interest">{{ 'sections.sourcing_form.interest_label' | t }}</label>
          <select id="SourcingForm-interest" class="field__input" name="contact[{{ 'sections.sourcing_form.interest_label' | t }}]">
            <option value="{{ 'sections.sourcing_form.interest_natural' | t }}">{{ 'sections.sourcing_form.interest_natural' | t }}</option>
            <option value="{{ 'sections.sourcing_form.interest_lab_grown' | t }}">{{ 'sections.sourcing_form.interest_lab_grown' | t }}</option>
            <option value="{{ 'sections.sourcing_form.interest_both' | t }}">{{ 'sections.sourcing_form.interest_both' | t }}</option>
          </select>
        </div>

        <div class="field">
          <textarea id="SourcingForm-details" class="text-area field__input" rows="6" name="contact[{{ 'sections.sourcing_form.details_label' | t }}]" placeholder="{{ 'sections.sourcing_form.details_label' | t }}" required></textarea>
          <label class="field__label" for="SourcingForm-details">{{ 'sections.sourcing_form.details_label' | t }}</label>
        </div>

        <div class="sourcing-form__submit">
          <button type="submit" class="button">{{ 'sections.sourcing_form.submit_label' | t }}</button>
        </div>
      {%- endif -%}
    {%- endform -%}
  </div>
</div>

{% schema %}
{
  "name": "Sourcing Form",
  "tag": "section",
  "class": "section",
  "settings": [
    {
      "type": "color_scheme",
      "id": "color_scheme",
      "label": "Color scheme",
      "default": "scheme-2"
    },
    {
      "type": "header",
      "content": "Section padding"
    },
    {
      "type": "range",
      "id": "padding_top",
      "min": 0,
      "max": 100,
      "step": 4,
      "unit": "px",
      "label": "Padding top",
      "default": 36
    },
    {
      "type": "range",
      "id": "padding_bottom",
      "min": 0,
      "max": 100,
      "step": 4,
      "unit": "px",
      "label": "Padding bottom",
      "default": 72
    }
  ],
  "presets": [
    {
      "name": "Sourcing Form"
    }
  ]
}
{% endschema %}
```

- [ ] **Step 5: Create the sourcing form stylesheet**

`assets/section-sourcing-form.css`:
```css
.sourcing-form .field {
  margin-bottom: 1.5rem;
}

.sourcing-form .field__input {
  width: 100%;
  border: none;
  border-bottom: 1px solid rgb(var(--color-foreground) / 0.2);
  background: transparent;
  padding: 0.75rem 0;
  font: inherit;
  color: currentColor;
}

.sourcing-form .field__input:focus {
  outline: none;
  border-bottom-color: currentColor;
}

.sourcing-form .field__label {
  display: block;
  font-size: 0.75rem;
  opacity: 0.6;
  margin-bottom: 0.25rem;
}

.sourcing-form__buyer-type {
  border: none;
  padding: 0;
  margin: 0 0 1.5rem;
  display: flex;
  gap: 1.5rem;
}

.sourcing-form__buyer-type legend {
  width: 100%;
  margin-bottom: 0.5rem;
}

/* Company name field only makes sense for trade buyers — pure CSS
   sibling-selector toggle, no JS needed since the radios are native
   form controls and this fieldset always precedes the company field
   in DOM order. */
.sourcing-form__company {
  display: none;
}

.sourcing-form__buyer-type:has(#SourcingForm-trade:checked) ~ .sourcing-form__company {
  display: block;
}

.sourcing-form__submit {
  margin-top: 1rem;
}

.sourcing-form__success {
  text-align: center;
  font-family: var(--font-heading-family);
}

.sourcing-form__success-body {
  display: block;
  margin-top: 0.75rem;
  font-family: var(--font-body-family);
  font-size: 1rem;
  opacity: 0.75;
}
```

- [ ] **Step 6: Create the page template**

`templates/page.diamond-sourcing.json`:
```json
{
  "sections": {
    "sourcing-landing": {
      "type": "sourcing-landing",
      "settings": {}
    },
    "sourcing-form": {
      "type": "sourcing-form",
      "settings": {}
    }
  },
  "order": [
    "sourcing-landing",
    "sourcing-form"
  ]
}
```

- [ ] **Step 7: Verify**

```bash
shopify theme check
```
Expected: no new errors.

- [ ] **Step 8: Commit**

```bash
git add sections/sourcing-landing.liquid assets/section-sourcing-landing.css sections/sourcing-form.liquid assets/section-sourcing-form.css locales/en.default.json locales/fr.json templates/page.diamond-sourcing.json
git commit -m "Add Diamond Sourcing landing section, intake form, and page template"
```

---

### Task 7: Footer content blocks

**Files:**
- Modify: `sections/footer-group.json`

**Interfaces:**
- Consumes: `link_list` and `brand_information` block types already defined in `sections/footer.liquid`'s schema (no Liquid/schema changes needed — only the stored block instances change).
- Produces: populated footer `blocks`/`block_order` — nothing later depends on this.

- [ ] **Step 1: Add footer blocks**

In `sections/footer-group.json`, the `sections.footer` object currently has `"blocks": {}` and `"block_order": []`. Replace with:

```json
"blocks": {
  "client_care": {
    "type": "link_list",
    "settings": {
      "heading": "Client Care",
      "menu": "footer"
    }
  },
  "brand": {
    "type": "brand_information",
    "settings": {
      "show_social": true
    }
  }
},
"block_order": [
  "client_care",
  "brand"
]
```

Keep every other existing key in `sections.footer` (`type`, `settings.color_scheme`, `settings.newsletter_enable`, etc.) unchanged — only `blocks` and `block_order` change.

- [ ] **Step 2: Verify**

```bash
shopify theme check
```
Expected: no new errors.

- [ ] **Step 3: Commit**

```bash
git add sections/footer-group.json
git commit -m "Populate footer with Client Care menu and brand info blocks"
```

**Manual follow-up (store content, not theme code — note for whoever deploys this to the live store):** the `menu: "footer"` block renders whatever links exist in Shopify Admin → Online Store → Navigation → "Footer" menu. Add Shipping & Returns, Privacy Policy, and Terms of Service entries there (Privacy/Terms/Refund policies also need their text filled in under Settings → Policies for the separate automatic `show_policy` row to appear).

---

### Task 8: Homepage assembly

**Files:**
- Modify: `templates/index.json`

**Interfaces:**
- Consumes: section types `three-universes` (Task 2), `featured_collection` editorial mode (Task 3), `maison` (Task 4).

- [ ] **Step 1: Add the new sections and reorder**

In `templates/index.json`, add two new entries to `"sections"` (alongside the existing `hero` and `featured_collection`):

```json
"three_universes": {
  "type": "three-universes",
  "settings": {}
},
"maison": {
  "type": "maison",
  "settings": {}
}
```

Change `"order"` from `["hero", "featured_collection"]` to:
```json
"order": [
  "hero",
  "featured_collection",
  "three_universes",
  "maison"
]
```

Leave the existing `hero` and `featured_collection` section entries' settings exactly as they are (the `featured_collection` entry doesn't need an explicit `"editorial_layout": true` — that's the schema default from Task 3).

- [ ] **Step 2: Verify**

```bash
shopify theme check
```
Expected: no new errors.

- [ ] **Step 3: Commit**

```bash
git add templates/index.json
git commit -m "Assemble homepage: hero, featured collection, three universes, maison"
```

---

### Task 9: Final verification and checklist update

**Files:**
- Modify: `README.md` (check off completed items)

**Interfaces:**
- Consumes: everything from Tasks 1-8.

- [ ] **Step 1: Full repo lint**

```bash
shopify theme check
```
Expected: no errors anywhere in the theme. If pre-existing warnings unrelated to this work are present, leave them — this task only guarantees no *new* errors from Tasks 1-8.

- [ ] **Step 2: Update the README checklist**

In `README.md`'s "Project status" section, change every item this plan covers from `- [ ]` to `- [x]`:
```markdown
- [x] Three Universes intro (Collection / Bespoke / Sourcing)
- [x] Featured Collection showcase (editorial, not a grid)
- [x] Bespoke journey (5-step process)
- [x] Diamond Sourcing landing (Private / Professional paths)
- [x] Maison/About section
- [x] Footer (client care, shipping/returns, privacy, terms, language selector)
- [x] Bespoke + Sourcing intake form templates
- [x] Design token pass (color schemes, type scale) tuned to the brand palette
```

Leave `- [ ] Further Dawn strip-down (B2B quick-order, local pickup — currently left in place, dormant/opt-in, low risk)` unchecked — explicitly out of scope per the design spec.

Add a short note directly below the checklist:
```markdown
Maison and Bespoke Journey sections ship with `[CLIENT COPY NEEDED]` placeholder
text — no source material for brand history or the specific 5-step process exists
yet. Replace before launch. French translations for the two intake forms are in
`locales/fr.json`; publishing French as a second store language (required for the
footer's language selector to appear) happens in Shopify Admin, not in theme code.
```

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "Update project status checklist — theme completion pass done"
```
