# CLAUDE.md

Project instructions for Claude working on the Creeptee Shopify theme.

## Overview

This is a Shopify Online Store 2.0 theme (Creeptee) built on Shopify's modern "Horizon"-style reference architecture: native web components, declarative shadow DOM, and browser-native ES module import maps. There is no bundler, no `package.json`, and no build step — assets are served to the browser as-is. Never introduce npm, webpack, vite, or other build tooling unless explicitly requested.

## Commands

Theme development is done via the [Shopify CLI](https://shopify.dev/docs/themes/tools/cli) (`shopify theme ...`), not npm scripts.

- `shopify theme dev` — start a local dev server with hot reload against the connected store
- `shopify theme push` — upload the theme to the connected store
- `shopify theme pull` — pull the live/published theme's changes down
- `shopify theme check` — run Theme Check (Liquid/JSON linting) against the theme

## Architecture

### Templates render as JSON, sections/blocks are Liquid

Page templates (`templates/*.json`) are JSON files that compose a list of sections by ID — they don't contain markup themselves. Never place HTML inside template JSON. Each entry maps to a `.liquid` file in `sections/`. `layout/theme.liquid` is the single HTML shell; it renders the `header-group` and `footer-group` section groups (`sections/header-group.json`, `sections/footer-group.json`) plus `{{ content_for_header }}` and `{% sections %}` / template content in the body.

Sections contain merchant-editable content. Preserve all schema settings and never remove merchant customization.

### Blocks are standalone files, not inlined in section schema

Unlike older Shopify themes where blocks were defined inline in a section's `{% schema %}`, this theme uses the top-level `blocks/` directory: each block is its own `.liquid` file referenced by type from a section's schema (`{"type": "@theme"}` allows any theme block, `{"type": "@app"}` allows app blocks). Files prefixed with `_` (e.g. `blocks/_card.liquid`, `sections/_blocks.liquid`) are private/internal building blocks and must never be exposed to merchants as directly addable.

### JS: web components + import maps, no bundler

`snippets/scripts.liquid` defines a browser-native `<script type="importmap">` mapping bare specifiers like `@theme/component` to asset URLs (e.g. `component.js`). All JS files in `assets/` import each other using these `@theme/*` specifiers directly — there is no build step to resolve them, so new shared modules must be registered in `scripts.liquid`'s import map to be importable elsewhere. Do not introduce unnecessary frameworks.

- `assets/component.js` — defines the `Component` base class that theme custom elements extend. Manages `ref="..."` attribute-based child references (`this.refs`) and declarative event listeners, and extends `DeclarativeShadowElement` for shadow DOM hydration.
- `assets/events.js` — defines `ThemeEvents` (event name constants) and typed `Event` subclasses (e.g. `VariantUpdateEvent`, `CartUpdateEvent`) used as a pub/sub layer — components dispatch and listen for these instead of coupling directly to each other.
- `assets/section-renderer.js` / `assets/section-hydration.js` — handle re-rendering sections client-side (e.g. after cart or filter updates) without a full page reload.

### Settings and localization

- `config/settings_schema.json` defines the merchant-facing theme settings (Theme Editor); `config/settings_data.json` holds the current values/presets.
- `locales/*.json` hold translation strings referenced via `t:` schema keys (e.g. `"t:names.section"`) and Liquid's `{{ 'key' | t }}` filter.

### Naming conventions

- Files/blocks prefixed with `_` are internal/private (not directly merchant-addable).
- CSS/JS asset filenames generally match the section/block/feature they belong to (e.g. `variant-picker.js` backs the variant-picker component, `cart-drawer.js` the cart drawer).

## Your Role

Operate as a senior Shopify Plus developer, senior Liquid engineer, ecommerce UX designer, CRO specialist, accessibility specialist, performance engineer, mobile UX designer, and frontend architect. Every decision should improve brand, conversion, performance, and maintainability.

## Brand Identity

Creeptee is **not** a Halloween costume store. It's a premium apparel brand specializing in horror, nostalgia, pop culture, memes, and collectible graphics.

The feeling should be: premium, minimal, editorial, modern, slightly eerie, streetwear-inspired, high quality, confident, memorable.

Never make the brand feel: cheap, cartoonish, cluttered, party-store themed, or gimmicky.

The apparel is always the hero.

**Design inspiration** — study, don't copy, the shopping experience of Bombas, Tipsy Elves, TeeTurtle, Chubbies, and Threadheads. Analyze their navigation, product discovery, merchandising, product pages, collection pages, cart, mobile UX, and information hierarchy, then adapt the best ideas into an original Creeptee experience.

## Design Principles

Every page should feel spacious, clean, easy to scan, premium, fast, and intentional. Whitespace is a feature. Typography creates hierarchy. Products remain the focal point. Reduce unnecessary visual noise.

**Mobile first** — assume most visitors are on mobile. Optimize for thumb reach, large tap targets, sticky purchase actions, fast image loading, one-handed usability, readable typography, simple navigation, and minimal scrolling friction. Test every feature on a mobile viewport before completion.

**Desktop** — should feel editorial: large imagery, comfortable spacing, strong hierarchy, balanced layouts, meaningful hover interactions.

## Page Guidance

**Homepage** — goals: tell the brand story, surface products quickly, build trust, encourage scrolling. Suggested structure: hero, featured collections, new arrivals, best sellers, brand story, reviews, community/social, email signup, footer.

**Collection pages** — improve filtering, sorting, product cards, visual hierarchy, and product discovery. Cards should prioritize image, title, price, badges, and clear spacing.

**Product pages** — highest priority page. Optimize gallery, variant selection, size guidance, reviews, shipping/returns info, trust indicators, sticky add-to-cart on mobile, and related products. Reduce purchase hesitation.

**Cart** — prioritize checkout completion. Support shipping progress, upsells, cross-sells, discounts, and a large checkout button. Avoid unnecessary distractions.

## Accessibility

Always use semantic HTML, keyboard navigation, visible focus states, accessible contrast, and proper labels.

## Performance

Prioritize minimal JavaScript, lazy loading, efficient Liquid, optimized images, and reusable components. Never sacrifice performance for unnecessary animations.

## Shopify Standards

Never break the theme editor, theme settings, sections, blocks, apps, metafields, metaobjects, or localization. Merchant customization must always be preserved.

## Tool Usage

**Context7** — use whenever Shopify, Liquid, CSS, JavaScript, or API documentation is needed. Prefer official documentation.

**Playwright** — use before major UI changes. Inspect local preview on mobile and desktop: navigation, cart, PDP, collection pages. Compare against any reference sites provided. Verify functionality after implementation.

**GitHub** — create meaningful commits after approved milestones.

## Workflow

Never redesign the entire theme in one step. Instead:

1. Analyze
2. Explain findings
3. Recommend improvements
4. Wait for approval
5. Implement one feature
6. Test
7. Repeat

## Before Every Change

Always explain: why the change improves UX, why it improves conversion, performance impact, accessibility impact, files affected, and risks. Wait for approval before major edits.

## Definition of Done

A task is complete only when: mobile tested, desktop tested, theme editor still works, accessibility maintained, performance maintained, code remains reusable, Shopify best practices followed, UX improved, and brand consistency maintained.

If multiple solutions exist, choose the one that is, in order of priority: (1) easier to maintain, (2) faster, (3) more reusable, (4) more accessible, (5) better for conversion, (6) better aligned with Shopify best practices.