# Tasks: Replace the green header photo

Status: in-progress
Plan: PLAN.md

## Wave 1: Baseline

- [x] T1: Baseline screenshots
  - Do: Build, then screenshot pages that show the header: home, home (mobile), identify key, taxon profile, garden, inventories, map search. Save to `~/symblca-baselines/261126-palette-images/screenshots/`.
  - Done when: 7 screenshots exist and show the current green header.

## Wave 2: Header photo

- [x] T2: Move the header background to LESS and swap in the new photo with a gray tint
  - Files: `images/header/header-spores.jpg`, `js/react/src/header/main.jsx`, `js/react/src/less/header.less`
  - Done when: build passes; header shows the new photo under a gray tint on all pages; nav text is readable.

## Wave 3: Review

- [ ] T3: Re-take the T1 screenshots, compare, and sign off
  - Done when: user signs off; the old `OF-Header_May8*.png` files are deleted if nothing references them.
