# Sakura Nexus Shopify Theme

Sakura Nexus is a Shopify theme for an anime collectibles store. It started from Shopify's Skeleton Theme and now follows a merchant-editable architecture: JSON templates define page composition, sections provide configurable page modules, blocks provide nestable editor content, and snippets hold reusable rendering logic.

## Getting Started

### Prerequisites

- [Shopify CLI](https://shopify.dev/docs/api/shopify-cli)
- [Shopify Liquid VS Code Extension](https://shopify.dev/docs/storefronts/themes/tools/shopify-liquid-vscode), recommended for Liquid syntax, linting, and inline docs

### Preview

```bash
shopify theme dev
```

### Validate

```bash
shopify theme check
```

Use `jq empty` for JSON files that are plain JSON, such as locale and theme settings files:

```bash
jq empty locales/en.default.json
jq empty locales/en.default.schema.json
jq empty config/settings_schema.json
```

Shopify JSON templates can include Shopify's generated header comments, so validate them with Shopify tooling instead of raw `jq`.

## Theme Architecture

```bash
.
├── assets          # Global assets and page-critical files only
├── blocks          # Reusable, nestable theme editor components
├── config          # Global theme settings and saved setting data
├── layout          # Top-level wrappers with Shopify header/layout hooks
├── locales         # Storefront and schema translations
├── sections        # Merchant-editable page modules
├── snippets        # Reusable Liquid rendering logic
└── templates       # JSON page composition
```

### Templates

Templates should define composition, not rendering details. For example, `templates/index.json` composes the homepage from separate sections:

- `welcome-modal`
- `hero`
- `product-carousel-or-grid`
- `collection-links`
- `about-banner`
- `faq-cta`

This keeps the homepage editable in the Shopify theme editor and avoids one large page-specific Liquid file.

### Sections

Sections are full-width page modules with their own `{% schema %}` settings. Merchant-editable content belongs in section settings or section blocks, including headings, image choices, CTA URLs, selected collections, product limits, and background style.

Current homepage sections:

- `sections/welcome-modal.liquid`
- `sections/hero.liquid`
- `sections/product-carousel-or-grid.liquid`
- `sections/collection-links.liquid`
- `sections/about-banner.liquid`
- `sections/faq-cta.liquid`

Legacy `sections/sakura-nexus-home.liquid` is retained for compatibility, but new homepage composition should use the modular sections above.

### Blocks

Blocks are reusable editor components that can be nested inside compatible sections. Use blocks when merchants need to add, remove, reorder, or configure repeated content. Existing examples include `blocks/group.liquid` and `blocks/text.liquid`.

Blocks must include a `{% schema %}` tag. If a block is statically rendered with `{% content_for 'block' %}`, include a LiquidDoc header.

### Snippets

Snippets hold reusable Liquid markup and display logic that does not need direct theme editor controls. Snippets should include a LiquidDoc header documenting purpose, parameters, and examples.

Shared Sakura Nexus snippets:

- `snippets/sn-product-card.liquid` renders product cards used across homepage, collection, product, and cart contexts.
- `snippets/sn-condition-badge.liquid` resolves product condition from `product.metafields.custom.condition`, then falls back to condition tags.
- `snippets/sn-section-heading.liquid` renders shared section eyebrow, heading, and optional action link markup.
- `snippets/sn-faq-cta.liquid` renders the shared FAQ call-to-action.

## Component Rules

- Keep merchant-editable content in section or block schema settings.
- Keep reusable display logic in snippets.
- Add LiquidDoc to every snippet and to statically rendered blocks.
- Use `{% stylesheet %}` and `{% javascript %}` inside sections, blocks, and snippets for component-level CSS and JS.
- Keep `assets/critical.css` limited to CSS needed across every page.
- Use CSS variables for settings that map to one CSS property, such as spacing or color.
- Use CSS classes for settings that control multiple style rules, such as layout variants.
- Prefer `routes.*`, `url`, `page`, or `link_list` settings over hard-coded storefront URLs.
- Avoid unsupported Liquid patterns, including parentheses in conditions, ternaries, and treating `limit` as a filter.

## Product Card Architecture

Product card rendering is centralized in `snippets/sn-product-card.liquid`. Pass class names when a page needs context-specific styling:

```liquid
{% render 'sn-product-card',
  product: product,
  collection: collection,
  card_class: 'sn-listing-card',
  image_class: 'sn-listing-card__image',
  price_class: 'sn-listing-card__price',
  stock_class: 'sn-listing-card__stock',
  badge_class: 'sn-condition-badge',
  image_width: 640
%}
```

Condition badges should be rendered through `snippets/sn-condition-badge.liquid` so metafield and tag fallback behavior stays consistent:

```liquid
{% render 'sn-condition-badge', product: product, class: 'sn-product-badge' %}
```

## CSS and JavaScript

Use component-scoped `{% stylesheet %}` and `{% javascript %}` tags in Liquid components. This keeps CSS and JS close to the markup they support while letting Shopify load each block only once.

Use `assets/critical.css` only for global layout primitives, baseline resets, and styles required on every page. Component styles should live with their section, block, or snippet.

## Quality Gates

Before committing theme changes:

```bash
shopify theme check
jq empty locales/en.default.json
jq empty locales/en.default.schema.json
jq empty config/settings_schema.json
```

Manually review these scenarios in a Shopify preview:

- Homepage with modular sections in `templates/index.json`
- Collection page with filters, sorting, product cards, and empty state
- Product page with images, variants, sold-out state, related products, and FAQ CTA
- Cart page with empty and non-empty states
- Search page
- Mobile layouts for all major templates
- Products without images
- Products without `custom.condition` metafield

## License

This theme is based on Shopify Skeleton Theme and uses the [MIT License](./LICENSE.md).
