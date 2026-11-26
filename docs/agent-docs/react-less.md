---
name: react-less
description: js/react/src/less — LESS stylesheets; mixins.less is the central variable/mixin hub; compiled to css/compiled/ by a separate Webpack config
---

## Files (verified against js/react/src/less)
| File | Purpose | Imports |
|------|---------|---------|
| `config.less` | `@subdir` variable for relative paths | — |
| `config_template.less` | Example config for deployment | — |
| `mixins.less` | Hub — all variables + mixins | `config.less` |
| `theme.less` | Global base styles (~64K, largest file) | `mixins.less`, `search-results.less`, `node_modules/react-modal-video/css/modal-video.min.css` |
| `header.less` | Sticky nav header | `mixins.less` |
| `footer.less` | Footer navbar | `mixins.less` |
| `taxa.less` | Standard taxon profiles | `mixins.less`, `node_modules/react-tabs/style/react-tabs.less` |
| `garden.less` | Garden taxon profiles | `mixins.less` |
| `rare.less` | Rare plant profiles + policy page | `mixins.less` |
| `inventory.less` | Inventory browser, tables, vendor upload (~21K) | `mixins.less` |
| `search-results.less` | Search grid/list layouts | — (imported by `theme.less`) |

## Webpack LESS entries (verified against js/react/webpack.config.js `lessConfig`)
Only **7 of the 11 files above are Webpack entries**: `theme`, `header`, `footer`, `garden`, `rare`, `taxa`, `inventory`. `search-results.less`, `mixins.less`, `config.less`, `config_template.less` are not entries — they're imported by the 7 real entries and never compiled standalone.

## Variables (mixins.less)
```less
@light-green: #dfefd3;       @med-green: #5fb021;
@light-green-alpha: #dfefd3cc; @med-dark-green: #5a8c37;
@dark-green: #3c641e;         @dark-grn-type: #1e352f;
@dark-gray: #000000e3;        @gray-type: #999999;
@type-primary: #000000c2;     @ivory: #fbf7ee;
```
The block above is the pre-2026-11 green palette, kept for history. Since the `261122-color-palette` plan, the variables are role-based (neutral palette, single dark-green accent):
```less
@bg-page: #fafaf7;  @bg-panel: #f0efeb;  @bg-panel-alpha: fade(@bg-panel, 80%);
@accent: #3c641e;   @accent-hover: #2d4b16;   // links, primary buttons, active controls
@accent-divider: lighten(@accent, 20%);  @accent-tint: fade(@accent, 10%);
@neutral-mid: #5c5c5c;  @neutral-mid-hover: #454545;  // bars, secondary buttons, borders, scrollbars
@text-heading: #333333; @text-strong: #222222; @text-primary: #000000c2;
@text-dark: #000000e3;  @text-muted: #666666;
@near-black: #1b1b1b;   @warm-gray: #cfcac2;   @black-65: #000000a6;
@font: 'Source Sans Pro', sans-serif !important;

// Breakpoints (match Bootstrap)
@xlarge: 1200px; @large: 992px; @medium: 768px; @small: 576px;
```
Mixins: `.text-capitalize()`, `.text-italic()`, `.text-uppercase()`, `.text-bold()`, `.text-drk-grn()`, `.text-center()`, `.bg-light-grn()`, `.border-rounded-sm()`, `.border-rounded-med()`, `.dropdown-hover()`, `.dropdown-current()`, `.collapsed-header()`. Note: `.text-drk-grn()` and `.bg-light-grn()` kept their old names but now apply `@text-strong` and `@bg-panel` (gray, not green). Likewise the JSX class `.text-light-green` (used only on `h3` headings) now renders `@text-heading`.

Palette colors live only in `mixins.less`; the other LESS files and JSX have no palette hex literals (only neutrals like white/black/gray and alert colors). Images are not covered: the header fern photo, search magnifier icons and `images/garden/*ViewIcon-*.png` stay green. The home hero (`.wrapper` in `theme.less`) uses `images/home-lichens.jpg` under a 50% black overlay so the white hero text stays readable.

Symbiota-based pages (map search, collections, image library) are styled by `css/symbiota/customizations.css`, not by these LESS files. Its `--oregonflora-green` / `--oregonflora-light-green` variables are set to the same values as `@accent` / `@bg-panel` and must be updated by hand if the palette changes.

## PHP template -> compiled CSS mapping (verified by grepping `<link>` tags, not assumed from JS bundle names)
| Page template | CSS loaded |
|---------------|------------|
| every page | `theme.css` (global base) |
| `header.php` | + `header.css` |
| `footer.php` | + `footer.css` |
| `taxa/index.php`, `taxa/garden.php`, `taxa/rare.php`, `taxa/search.php` | + `taxa.css` only |
| `garden/index.php`, `rare/index.php`, `rare/policy.php` | + `garden.css` or `rare.css` (checklist-special pages) |
| `ident/key.php`, `projects/index.php`, `checklists/checklistvendor.php`, `checklists/checklist.php` | + `inventory.css` |

This corrects an earlier version of this memory, which claimed `taxa/garden.php` and `taxa/rare.php` also load `garden.css`/`rare.css`. Verified they load only `taxa.css` — the garden/rare LESS files are used exclusively by the [[react-checklist-special]] pages (`garden/index.php`, `rare/index.php`, `rare/policy.php`), which is a separate PHP page family from `taxa/garden.php` and `taxa/rare.php` (which render the [[react-taxa]] `taxa-garden`/`taxa-rare` bundles).

## Build Notes
- Webpack compiles LESS via a separate config object (`lessConfig`) from JS (`reactConfig`) — see [[react-frontend]]
- Output: `css/compiled/` (not `js/react/dist/`)
- Webpack's LESS pipeline emits spurious `.js` files alongside the CSS output; an `afterEmit` hook runs `rm -f css/compiled/*.js` to clean them up (verified in `js/react/webpack.config.js`)
- `config.less` vs `config_template.less`: deploy-specific `@subdir` path kept separate from the checked-in template

## Gotcha: CSS cache-busting via filemtime()

Verified across many PHP templates (`header.php`, `index.php`, `footer.php`, `ident/key.php`, `checklists/checklistvendor.php`, `taxa/rare.php`, `taxa/garden.php`, etc.): every compiled CSS `<link>` tag appends `?<?php echo filemtime($SERVER_ROOT . '/css/compiled/<file>.css'); ?>` as a cache-busting query string, e.g.:
```php
<link rel="stylesheet" href="<?php echo $CLIENT_ROOT?>/css/compiled/theme.css?<?php echo filemtime($SERVER_ROOT . '/css/compiled/theme.css'); ?>">
```
Because the version string is the file's own modification time, it only changes when Webpack actually rewrites that specific CSS file. If a LESS build is skipped or only touches a different entry, the unaffected files' `filemtime()` stays the same and browsers keep serving the previously cached CSS for those files.

## search-results.less
- Grid layout: 5 columns -> 4 -> 3 (responsive)
- Card styling, image thumbnails, hover effects

## Related
[[react-frontend]], [[react-header]], [[react-footer]], [[react-taxa]], [[react-checklist-special]], [[react-explore]], [[react-identify]], [[react-inventory]]
