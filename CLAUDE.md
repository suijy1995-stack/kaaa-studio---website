# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The marketing/business site for Kaaa Studio, a fine art studio and residency program in Toride, Japan (near Tokyo), offering glass/ceramic/sculpture art residencies, art-school exam prep packages ("套餐"), kiln rentals, and short-term workshop experiences for Chinese-speaking artists and students. Content is primarily Simplified Chinese (`lang="zh-CN"`) with Japanese terms mixed in.

## Architecture

**The entire site is one file: `index.html`.** There is no build system, no package manager, no bundler, no framework — just a single static HTML document with:

- `<style>` block (lines ~7–2326): all CSS, using CSS custom properties defined in `:root` (`--c-*` colors, `--f-*` font stacks, `--page-*` layout tokens).
- HTML body: a single-page app where every top-level content block is a `<div class="section" id="...">` (e.g. `s01`–`s08`, `overview`, `about`). Only one section has class `visible` at a time; JS toggles visibility instead of routing between pages.
- `<script>` block (starts ~line 4025): all interactivity, in plain vanilla JS (no imports, no modules).

Because everything lives in one ~13MB HTML file, prefer targeted `Read` with `offset`/`limit` or `grep -n` over reading the whole file at once.

### Section-based navigation (no routing)

- `.nav-item[data-sec="..."]` elements in `.nav-wrap` drive navigation; clicking one calls `showSection(id)`.
- `showSection(id, addHistory)` hides all `.section` elements, shows the one matching `id`, activates the matching nav item, and pushes the previous section onto an in-memory `_sectionHistory` stack for the back button (`goBack()` / `#navBack`).
- Elements with `data-goto="<section-id>"` (e.g. `.ov-card`, `.entry-card`) also trigger `showSection` on click — used for "click a card to jump to a section" patterns.
- There is no URL/hash sync; refreshing the page always resets to the default visible section.

### Data-driven pricing UI

Pricing tables aren't static HTML — they're rendered from JS arrays and re-rendered on tab click:

- `packages[]` (~line 4082) → `renderPkg(idx)` → injects into `#pkgDisplay`, wired to `#pkgNav` tab buttons (`.pkg-btn[data-pkg]`).
- `expData[]` (~line 4159) → `renderExp(idx)` → injects into `#expDisplay`, wired to `#expNav` tab buttons (`.pkg-btn[data-exp]`).
- `schoolData` (near end of file) → `renderSchoolList(type)` → injects into `#schoolDisplay`, wired to `.school-tab[data-school]` buttons. Contains lists of Japanese art universities by discipline (glass/ceramic/sculpture) with professor names.
- The "Decision Guide" (`.dg-option[data-q]`) scores three answered questions and recommends one of the five `packages` via `dgGoToPkg(pkgIdx)`.

When editing prices, packages, or school lists, edit the JS data arrays/objects — do not hand-edit generated markup, it will be overwritten by the render functions.

### Contact configuration (single source of truth)

All CTAs and footer contact links derive from constants at the top of the `<script>` block (~line 4030):

```js
const CONTACT_LINE_URL, CONTACT_WECHAT_ID, CONTACT_EMAIL, CONTACT_INSTAGRAM_URL
```

Service-specific mailto links (`RESIDENCY_APPLICATION_URL`, `CONSULTATION_URL`, `KILN_BOOKING_URL`, etc.) are derived from `CONTACT_EMAIL` via the `_m()` helper, which pre-fills a subject line. `openContact(url, label)` is the single click-handler used across CTA buttons — it opens `mailto:` links in place and `http(s)` links in a new tab. `renderFooterContacts()` builds the footer links, showing a "待配置" (not configured) placeholder for any null contact method (currently LINE and Instagram are unset). WeChat has a dedicated modal (`showWechatModal`/`closeWechatModal`/`copyWechat`) instead of a direct link, since WeChat has no universal URL scheme.

**To change any contact channel or CTA destination, edit only the constants block — never hardcode a URL in a button's `onclick`.**

### Images

- Images actually used in the page are inlined as base64 `data:` URIs inside `index.html` — there are no `<img src="...">` references to files on disk.
- The dozens of loose `.jpg`/`.JPG`/`.png` files in the repo root (`kaaa-*.jpg`, `yuan-*.JPG`, `poster_*.jpg`, Chinese-named files, etc.) are **source assets**, not referenced anywhere by `index.html`. They exist for re-encoding into the page, not for direct serving. Don't assume adding a file here makes it appear on the site — it must be base64-encoded and inlined into the relevant section's `<style>` `background-image` or an inline `<img>`.

### robots.txt

Deliberately blocks AI training/retrieval crawlers (`Google-Extended`, `GPTBot`, `ClaudeBot`, `Applebot-Extended`) while allowing standard search engines (`Googlebot`, `Bingbot`) and other unlisted crawlers. Keep this distinction in mind if asked to modify crawler access — it's an intentional business decision, not an oversight.

## Design Direction & Brand Guidelines

Kaaa Studio is a fine art studio in Japan. **It is not an art school, and it is not a commercial design agency.** Every visual and content decision should read as a cultural institution or editorial publication — never as a startup landing page. Hold this in mind for any change to `<style>`, layout, imagery, or copy tone, not just when explicitly asked for a "redesign."

**Purpose:** This is not a sales site. Its job is to build trust, cultural value, and long-term relationships — not to maximize conversions. Don't optimize copy, layout, or CTA prominence for conversion the way a SaaS or e-commerce site would; that pressure is explicitly not the goal here.

**Narrative hierarchy:** Residency is the primary narrative — the studio's ongoing art practice comes first. Education (the exam-prep packages, school placement) is secondary and supporting. Any new content, section ordering, or emphasis should keep the site reading as an art practice first and a service provider second, even where the "套餐"/pricing content is necessarily commercial in substance.

**Photography:** Photography should always dominate the page — it carries the work. Avoid decorative graphics, icon sets, or illustration standing in for real photography of the studio, artists, and work.

**Writing style:** Objective, editorial, minimal. Avoid exaggerated marketing language and startup copywriting (no "unlock," "elevate," "transform your practice," etc.). When drafting or editing copy — including Chinese/Japanese copy — favor plain, descriptive statements over persuasive framing.

The overall test: the site should feel like walking into an exhibition, not landing on a product page.

**Primary references:** Nippon Design Center (NDC), Japanese editorial design, exhibition catalogues, contemporary art museums, independent publishing.

**Secondary references:** MUJI art direction, Kenya Hara, 21_21 DESIGN SIGHT.

**Target qualities:** quiet, refined, timeless, spacious, intellectual, calm, typography-first, editorial. Whitespace and type hierarchy carry the design — decoration should not.

**Avoid:**
- Startup UI / SaaS landing-page conventions (hero gradients, feature-grid-with-icons, big rounded CTA buttons)
- Glassmorphism, heavy gradients, neon colors
- Cartoon illustrations
- Over-animation (motion should be restrained — subtle fades/reveals only, never bouncy or attention-grabbing)
- Corporate marketing tone, in copy or in layout

When proposing new UI, default to what an exhibition catalogue or museum website would do, not what a typical product marketing site would do — restrained palette (see the existing `--c-*` tokens), generous margins, serif/sans pairing already established in `--f-serif`/`--f-sans`/`--f-display`, and understated interaction states.

## Working in this repo

- There is no dev server, test suite, linter, or build step — this is deployed as a static file (commits are literally titled "Add files via upload", indicating uploads via the GitHub web UI rather than a local dev workflow). To preview changes, open `index.html` directly in a browser.
- Because the whole page is one file, use `grep -n` to jump to the relevant `<style>` rule, `.section` block, or JS function before editing rather than scanning linearly.
- Keep new interactive features as vanilla JS appended to the existing `<script>` block, following the existing patterns (plain functions + `addEventListener`, template-literal HTML injection into a container by id).
- Prefer improving existing code over rewriting it. Do not rewrite large sections of `index.html` (a `<style>` block, a `.section`, the `<script>`) unless the user explicitly asks for that scope of change.
- Before implementing any major structural change (new section layout, new nav/routing pattern, restructuring the data-driven render functions, etc.), explain the change and get agreement first rather than just making it.
