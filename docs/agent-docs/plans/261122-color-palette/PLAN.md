# Plan: Update the website color palette

Status: complete
Request: Update the color palette of the website.
Spec: none

## Approach

Colors are mostly centralized in `js/react/src/less/mixins.less` (10 color variables, see [[react-less]]), but 86 color literals are hardcoded across the other LESS files, many of them copies of existing variable values (`#5fb021` x8, `#dfefd3` x7, `#1e352f` x6, `#3c641e` x3). Changing only the variables would leave those copies on the old palette.

So the work runs in two phases:

1. **Consolidate (no visual change).** Replace every hardcoded color in the LESS files with a variable from `mixins.less`, adding new variables for repeated values that have no variable yet (e.g. `#1b1b1b`, `#cfcac2`, `#288000`). After this phase the compiled CSS should be visually identical.
2. **Swap.** Change the values in `mixins.less` to the new palette. One-off neutrals (`#fff`, `#000`, transparent) stay literal.

Doing consolidation first keeps the risky step (visual change) to a single file edit that is easy to review and revert.

Scope confirmed with user (2026-11-22): the React LESS files plus inline colors in 3 JSX files. Legacy Symbiota CSS (`css/main.css`, `css/base.css`, `css/speciesprofile.css`, loaded by older pages such as `ident/dichotomous/*` and `themes/*`) is out of scope and keeps the old palette. Variables are renamed to role-based names. The palette is neutral with one accent color: the existing dark green.

## Affected Code

- `js/react/src/less/mixins.less` (modified): add variables for unvariabled repeated colors (phase 1); new palette values (phase 2).
- `js/react/src/less/theme.less` (modified): 37 hardcoded colors to variables.
- `js/react/src/less/garden.less` (modified): 15.
- `js/react/src/less/inventory.less` (modified): 13.
- `js/react/src/less/taxa.less` (modified): 10.
- `js/react/src/less/header.less` (modified): 5.
- `js/react/src/less/search-results.less` (modified): 4. Not imported via `mixins.less` today (it is imported by `theme.less`), so confirm variables resolve or add the import.
- `js/react/src/less/rare.less` (modified): 2.
- `js/react/src/common/iconButton.jsx`, `js/react/src/common/filterSidebar.jsx`, `js/react/src/identify/sidebar.jsx` (modified): inline-style colors; move to classes in LESS or update values.
- `css/compiled/*.css` (rebuilt output).

## Data Model and Contracts

Approved palette (user, 2026-11-22): neutral (white, off-white, gray) with the existing dark green as the single accent for links and primary buttons. Variables get role-based names.

Final role variables in `mixins.less`. Contrast checked against WCAG AA (4.5:1 for normal text):

| New variable | Value | Role | Replaces | Contrast |
|---|---|---|---|---|
| `@bg-page` | `#fafaf7` off-white | page background | `@ivory` `#fbf7ee` | |
| `@bg-panel` | `#f0efeb` warm light gray | panels, cards, highlighted rows | `@light-green` `#dfefd3` | |
| `@bg-panel-alpha` | `#f0efebcc` | translucent overlays | `@light-green-alpha` | |
| `@accent` | `#3c641e` dark green | links, primary buttons | `@dark-green` (link uses), `@med-green` (primary button uses) | white text on it 6.92 |
| `@accent-hover` | darker green, value chosen in T6 | hover/active on accent | `@med-dark-green` (accent uses) | must be at least 6.92 |
| `@neutral-mid` | `#5c5c5c` mid gray | secondary buttons, borders, icons | `@med-green` / `@med-dark-green` (non-primary uses) | white text on it 6.69 |
| `@neutral-mid-hover` | `#454545` | hover on neutral | `@med-dark-green` (non-primary uses) | white text on it 9.59 |
| `@text-heading` | `#333333` charcoal | headings | `@dark-green` (heading uses) | on off-white 12.08 |
| `@text-strong` | `#222222` | dark text | `@dark-grn-type` `#1e352f` | on off-white 15.21 |
| `@text-primary` | `#000000c2` (unchanged) | body text | `@type-primary` | |
| `@text-dark` | `#000000e3` (unchanged) | | `@dark-gray` | |
| `@text-muted` | `#666666` | muted text | `@gray-type` `#999999` (2.85, fails) | on white 5.74 |

Notes:
- `@dark-green`, `@med-green` and `@med-dark-green` each split into two roles (accent vs. neutral/heading). In T6 each use site is classified as one or the other, so this is not a find-and-replace.
- `#707070` was considered for muted text but reaches only 4.30 on `#f0efeb`, so `#666666` is used.
- The old variable names are removed once all call sites use the new names. No aliases are kept.

## Libraries

None.

## Risks

- Phase 1 changes rendering by accident (a near-duplicate like `#3b631d` folded into `@dark-green` `#3c641e`): diff `css/compiled/*.css` before and after; only exact-value substitutions may produce zero diff, and any deliberate merge of near-duplicates is listed in the task.
- Stale CSS in browsers: compiled files are cache-busted by `filemtime()`, so all 7 entries must be rebuilt (see [[react-less]] gotcha).
- Contrast regressions: check new text/background pairs against WCAG AA (4.5:1 body text, 3:1 large text and UI).
- Legacy pages keep the old palette (accepted by user), so older Symbiota pages will look different from the React pages.
- Misclassifying a `@dark-green` / `@med-green` use (accent vs. neutral) gives gray links or green borders: review each use site in T6 and check hover/focus states in T7.
- Images and logos (`css/images/`, header logo) may carry the old green; review them in phase 2.

## Handoff

- Last session: 2026-11-22
- Branch: `feature/color-palette` (from `dev` at `a587c3192`)
- Current state: T1-T6 done. New palette is live in the local build; after-T6 screenshots saved to `~/symblca-baselines/261122-color-palette/screenshots-after-t6/` for side-by-side comparison with `screenshots/`. Previously: T1-T5 done (phase 1 complete). All palette colors in LESS and JSX go through `mixins.less` variables. Compiled CSS matches the baseline except for 3 expected rules in `theme.css` from T5; screenshots are pixel-identical.
- Baseline note: after T5, `theme.css` no longer matches the baseline checksum by design. Phase 2 changes colors anyway, so use screenshots from here on.
- Baseline: `~/symblca-baselines/261122-color-palette/` in WSL (outside the repo, because `css/compiled/` is gitignored). It holds `css/` (raw minified build output), `css-readable/` (one rule per line, for diffing), `SHA256SUMS`, `SOURCE_COMMIT` and `screenshots/` (9 PNGs: home, home-mobile, taxa, search, ident-key, projects, garden, rare, checklist). Built with `npm run build` from `js/react`, run inside WSL.
- Local site: native Apache + PHP 8.5 + MariaDB in WSL serves `http://localhost/symblca/` (`$CLIENT_ROOT = '/symblca'`). The DB is nearly empty (6 taxa, no checklists), so pages are mostly shells; the garden page's "An error occurred" box is caused by the empty DB, not by this work.
- Screenshots: take them from Windows with headless Edge launched through the PowerShell tool (`Start-Process msedge.exe -Wait` with `--headless=new --user-data-dir=<temp> --virtual-time-budget=8000 --window-size=W,H --screenshot=<png> <url>`). Launching Edge from Git Bash reported success but often didn't write the file.
- How to verify phase 1: rebuild, then `cd ~/symblca/css/compiled && sha256sum -c ~/symblca-baselines/261122-color-palette/SHA256SUMS`. Run it from `css/compiled`; running it from the baseline folder compares the baseline with itself. All OK means no visual change. On a mismatch, split the new file with `sed 's/}/}\n/g'` and diff it against `css-readable/`.
- Environment notes: run git and npm through WSL (`wsl.exe -d Ubuntu -- bash -s`). Windows Git refuses the `\\wsl.localhost` path (dubious ownership).
- Final state (2026-11-22): all tasks T1-T7 complete and signed off by the user; plan closed. Branch `feature/color-palette` is ready for review/merge into `dev` (not merged or pushed).
- Possible follow-ups (not planned): mobile home-page text overflows the right edge (pre-existing); hue-named mixins `.text-drk-grn()` / `.bg-light-grn()` and class `.text-light-green` could be renamed; green image assets (header fern photo, magnifier icons, garden view-toggle icons) don't follow the palette; unused `images/home.jpg` could be removed.
- Scope change (2026-11-22): `css/symbiota/customizations.css` is now partly in scope (its two color variables only); the rest of the legacy Symbiota CSS still keeps the old palette. Also for T7, note that the garden/rare grid and list view toggles use green PNG icons (`images/garden/gridViewIcon-{dark,light}.png`, `listViewIcon-*.png`), and the header/hero use fern photos; those images won't follow the palette. Replacement rules used in T3/T4, for reference:
  - exact copies: `#5fb021`/`#5FB021` → `@med-green`, `#dfefd3`/`#DFEFD3` → `@light-green`, `#1e352f` → `@dark-grn-type`, `#3c641e` → `@dark-green`, `#999999` → `@gray-type`
  - T2 variables: see the block below `@ivory` in `mixins.less`
  - `rgba(60, 100, 30, 0.5)` (theme.less, OpenLayers buttons) is `@dark-green` at 50%; try `fade(@dark-green, 50%)` and confirm the checksum still matches
  - leave literal: white/black/gray/transparent, `#fff`/`#000`, one-off neutral grays (`#888`, `#707070`, `#b0b0b0`, `#ccc`, `#ddd`, `#efefef`, `#dee2e6`), shadows (`rgba(0,0,0,x)`, `rgba(100,100,100,0.3)`), and alert/status colors (`#ffe300`, `#d50909`, `#ff5555`, `red`, `#945200`, `#844244`)
- Open questions: none. Decisions (2026-11-22): neutral palette approved; single accent = existing dark green `#3c641e`; legacy Symbiota CSS keeps the old palette; variables renamed to roles.
- Learned (promote to agent-docs if durable): 86 hardcoded color literals live outside `mixins.less`; `search-results.less` has no `mixins.less` import.
