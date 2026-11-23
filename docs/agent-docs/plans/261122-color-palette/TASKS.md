# Tasks: Update the website color palette

Status: in-progress
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

- [x] T4: Replace hardcoded colors in taxa, garden, rare, inventory LESS
  - Files: `js/react/src/less/taxa.less`, `garden.less`, `rare.less`, `inventory.less`
  - Do: Same as T3.
  - Tests: build and diff against the baseline.
  - Done when: compiled CSS diff is empty.
  - Result (2026-11-22): 21 replacements (garden 10, taxa 6, inventory 3, rare 2). All 7 compiled files match the baseline checksums. No palette-colored literals remain in any LESS file outside `mixins.less`.

- [ ] T5: Move JSX inline colors to LESS
  - Files: `js/react/src/common/iconButton.jsx`, `js/react/src/common/filterSidebar.jsx`, `js/react/src/identify/sidebar.jsx`, plus the matching LESS file
  - Do: Move inline `color` / `backgroundColor` literals into class rules that use variables, unless the color is computed at runtime.
  - Tests: `npm run build` from `js/react`; visual check against T1 screenshots.
  - Done when: no hardcoded palette colors remain in JSX.

## Wave 3: Rename and swap palette

- [ ] T6: Introduce role variables and migrate call sites
  - Files: `js/react/src/less/mixins.less` and every LESS file that uses the old variables
  - Do: Add the role variables from the PLAN.md table with their new values. Replace each old variable use with its role variable. For `@dark-green`, `@med-green` and `@med-dark-green`, decide per use site whether it is an accent (links, primary buttons) or neutral (headings, borders, secondary buttons, icons), and list any unclear cases in the Handoff. Pick an `@accent-hover` value darker than `#3c641e`. Remove the old variables once nothing references them, and update the variable list in `docs/agent-docs/react-less.md`.
  - Tests: build (an undefined variable fails the LESS build); check text/background pairs against WCAG AA.
  - Done when: no old variable names remain (`grep -rE '@(light-green|med-green|med-dark-green|dark-green|dark-grn-type|dark-gray|gray-type|type-primary|ivory)\b' js/react/src/less` is empty), all 7 compiled CSS files are rebuilt, and contrast checks pass.

- [ ] T7: Visual review
  - Files: none, or touch-ups in LESS
  - Do: Re-take the T1 screenshots and compare. Check hover, focus and active states, the mobile menu, and images/logos that carry the old color.
  - Tests: manual review with the user.
  - Done when: the user signs off.

Out of scope (user decision, 2026-11-22): legacy Symbiota CSS (`css/main.css`, `css/base.css`, `css/speciesprofile.css`) keeps the old palette.
