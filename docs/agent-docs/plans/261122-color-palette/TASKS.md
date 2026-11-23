# Tasks: Update the website color palette

Status: complete
Plan: PLAN.md

Mark a task `[x]` only when its "Done when" is met, and append the commit hash: `(commit abc1234)`.
Tasks within a wave are independent; a wave starts only after the previous wave is complete.

## Wave 1: Baseline

- [x] T1: Snapshot current compiled CSS
  - Files: none committed; snapshot to a scratch directory
  - Do: From `js/react`, run the LESS build and copy `css/compiled/*.css` to a scratch folder as the baseline for later diffs. Take screenshots of the home page, a taxon profile, the identify key, an inventory page and a garden page for visual comparison.
  - Tests: build succeeds.
  - Done when: baseline CSS and screenshots exist and their location is noted in the Handoff.
  - Result (2026-11-22): CSS baseline and 9 screenshots saved; see the Handoff for location and how to re-take them.

## Wave 2: Consolidate (no visual change)

- [x] T2: Add missing variables to mixins.less (commit 7be2e8997)
  - Files: `js/react/src/less/mixins.less`
  - Do: Inventory every color literal in the LESS files (`grep -oiE '#[0-9a-f]{3,8}\b|rgba?\('`). For each value repeated 2+ times with no variable, add one (e.g. `#1b1b1b`, `#cfcac2`, `#288000`, `#000000a6`, `#9fd07a`, `#7abf46`, `#efffe3`). List near-duplicates (e.g. `#3b631d` vs `#3c641e`) in the Handoff for a user decision; do not merge them yet.
  - Tests: build succeeds.
  - Done when: every repeated color has a variable.
  - Result (2026-11-22): added 13 variables. Every green-family literal got one, even if used once, because all of them must change in T6; repeated neutrals (`#1b1b1b`, `#cfcac2`, `#000000a6`) did too. The near-duplicate `#3b631d` got its own variable (`@dark-green-alt`) instead of a merge, since both it and `@dark-green` become `@accent` in T6. Build checksums match the baseline.

- [x] T3: Replace hardcoded colors in theme.less, header.less, search-results.less (commit 3b7f5a443)
  - Files: `js/react/src/less/theme.less`, `header.less`, `search-results.less`
  - Do: Replace exact-match literals with the corresponding variable. Make sure `search-results.less` can resolve the variables (it is imported by `theme.less` after `mixins.less`; add an explicit import if the build fails).
  - Tests: build, then diff `css/compiled/theme.css` and `header.css` against the T1 baseline.
  - Done when: compiled CSS diff is empty or only reformatted equivalents.
  - Result (2026-11-22): 25 replacements (theme 21, search-results 3, header 1). `rgba(60, 100, 30, 0.5)` became `fade(@dark-green, 50%)`. `search-results.less` needed no extra import. All 7 compiled files match the baseline checksums. Remaining literals are the intended neutrals and alert colors.

- [x] T4: Replace hardcoded colors in taxa, garden, rare, inventory LESS (commit be47c0a4f)
  - Files: `js/react/src/less/taxa.less`, `garden.less`, `rare.less`, `inventory.less`
  - Do: Same as T3.
  - Tests: build and diff against the baseline.
  - Done when: compiled CSS diff is empty.
  - Result (2026-11-22): 21 replacements (garden 10, taxa 6, inventory 3, rare 2). All 7 compiled files match the baseline checksums. No palette-colored literals remain in any LESS file outside `mixins.less`.

- [x] T5: Move JSX inline colors to LESS (commit 9ddee3716)
  - Files: `js/react/src/common/iconButton.jsx`, `js/react/src/common/filterSidebar.jsx`, `js/react/src/identify/sidebar.jsx`, plus the matching LESS file
  - Do: Move inline `color` / `backgroundColor` literals into class rules that use variables, unless the color is computed at runtime.
  - Tests: `npm run build` from `js/react`; visual check against T1 screenshots.
  - Done when: no hardcoded palette colors remain in JSX.
  - Result (2026-11-22): the original file list was partly wrong. `filterSidebar.jsx` and `identify/sidebar.jsx` only use black/white (left as is). A full search of `js/react/src` found palette colors in:
    - `common/iconButton.jsx`: selected/unselected inline colors replaced by classes `icon-button-selected` / `icon-button-unselected` in `theme.less`, using `!important` to keep the precedence the inline styles had.
    - `style={{ background: '#DFEFD3' }}` on the sidebar in `explore.jsx`, `explore-vendor.jsx`, `identify.jsx` and `checklist-special/main.jsx`: removed; `background: @light-green` added to the existing `.explore, .identify, #react-garden, #react-rare { #sidebar }` rule, which covers all four pages. All four call sites passed this style, so no sidebar gains a background it didn't have.
  - Verification: `theme.css` diff vs. baseline is exactly the 3 expected rules; the other 6 compiled files match. Screenshots of ident-key, garden, rare, checklist, projects and home are pixel-identical to the baseline. The garden/rare sort and view buttons (`IconButton`) only render with search results, which the empty local DB doesn't produce, so that change is verified by CSS only; check it in T7 if data is available.

## Wave 3: Rename and swap palette

- [x] T6: Introduce role variables and migrate call sites
  - Files: `js/react/src/less/mixins.less` and every LESS file that uses the old variables
  - Do: Add the role variables from the PLAN.md table with their new values. Replace each old variable use with its role variable. For `@dark-green`, `@med-green` and `@med-dark-green`, decide per use site whether it is an accent (links, primary buttons) or neutral (headings, borders, secondary buttons, icons), and list any unclear cases in the Handoff. Pick an `@accent-hover` value darker than `#3c641e`. Remove the old variables once nothing references them, and update the variable list in `docs/agent-docs/react-less.md`.
  - Tests: build (an undefined variable fails the LESS build); check text/background pairs against WCAG AA.
  - Done when: no old variable names remain (`grep -rE '@(light-green|med-green|med-dark-green|dark-green|dark-grn-type|dark-gray|gray-type|type-primary|ivory)\b' js/react/src/less` is empty), all 7 compiled CSS files are rebuilt, and contrast checks pass.
  - Result (2026-11-22): 245 lines changed across 9 LESS files. One-to-one renames for single-role variables; 128 uses of `@med-green` / `@dark-green` / `@med-dark-green` classified one by one (accent: links, buttons, active states, sliders, clickable icons; text-heading: headings; neutral-mid: bars, dividers, scrollbars, input borders; text-strong / text-muted: typed text / placeholders). The T2 temporary variables were folded into roles. Added `@accent-hover` `#2d4b16`, `@accent-divider`, `@accent-tint`. No old names remain; build passes; 11 contrast pairs all pass AA (lowest: `@text-muted` on `@bg-panel`, 4.99).
  - Fix after first screenshots: `.text-light-green` (only used on `h3`) had been mapped to accent, which left taxa sidebar headings green; changed to `@text-heading`.
  - Judgement calls to review in T7:
    - `theme.less` `.slide-col-1 .row a` (home hero links over a dark photo) set to `@bg-panel` (light), not accent; dark green would be unreadable there.
    - Footer bar, scrolled-header dropdowns, dropdown menus and the rare-page filter modal backgrounds became `@neutral-mid` gray.
    - Map page (`body.spatial`) panels became `@neutral-mid-hover`; map buttons use `@accent`.
    - Filter chips (`#view-opts .cancel`) and the "reset" button became `@neutral-mid`.
    - Kept the hue-named mixins `.text-drk-grn()` / `.bg-light-grn()` (now gray) to avoid touching their call sites.

- [x] T7: Visual review (commit "Apply T7 review changes and lichen hero photo")
  - Files: none, or touch-ups in LESS
  - Do: Re-take the T1 screenshots and compare. Check hover, focus and active states, the mobile menu, and images/logos that carry the old color.
  - Tests: manual review with the user.
  - Done when: the user signs off.
  - Decisions (user, 2026-11-22): go with the recommendations. Items 1-5 kept as is; item 6 `#about-guide-flip` → `@neutral-mid`; item 7 extend scope to the two variables in `css/symbiota/customizations.css` (`--oregonflora-green` → `#3c641e`, `--oregonflora-light-green` → `#f0efeb`, with a comment to keep them in sync with `mixins.less`). Addition: replace the home hero photo with the user's `Lichens.jpg` (4032x2268), resized with PHP GD to 2000x1125 q82 as `images/home-lichens.jpg` (620 KB, EXIF stripped). The `.wrapper` background in `theme.less` now layers a `rgba(0,0,0,0.5)` overlay over it for text readability, with fallback color `@near-black` instead of `darkgreen`. The old `images/home.jpg` is left in place but no longer referenced.
  - Verification: build passes; re-shot home, home-mobile, rare, rare-mobile and map into `screenshots-after-t7/`; `compare.html` uses them. Mobile right-edge text cutoff is in the baseline too (pre-existing). User signed off 2026-11-22.
  - Progress (2026-11-22): T6 committed as 743bde707. Before/after comparison page at `~/symblca-baselines/261122-color-palette/compare.html` (11 page pairs, incl. new `map` and `rare-mobile`; their "before" shots were taken by temporarily copying the baseline CSS into `css/compiled/`, then rebuilding). Waiting on the user for the T6 judgement calls plus two new findings:
    - 6. Rare page "About This Guide" bar (`#about-guide-flip`, was `@med-dark-green`) is now a solid accent-green band; maybe `@neutral-mid` instead.
    - 7. The map search page (`collections/map/index.php`) is unchanged: it is styled by `css/symbiota/customizations.css` (loaded via `includes/head.php` on Symbiota-based pages), which defines `--oregonflora-green: #74B147` and `--oregonflora-light-green: #dfefd3` (7 uses). Out of scope per the legacy decision, but it's a main-nav feature, so ask whether to extend scope to just those two variables. The `body.spatial` rules changed in T6 did not visibly affect this page.

Out of scope (user decision, 2026-11-22): legacy Symbiota CSS (`css/main.css`, `css/base.css`, `css/speciesprofile.css`) keeps the old palette.
