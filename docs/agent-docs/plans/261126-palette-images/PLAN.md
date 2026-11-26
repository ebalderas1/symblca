# Plan: Replace the green header photo

Status: complete
Request: Replace the green fern photo behind the site header with my own photo, with a solid gray accent, to match the neutral palette.
Spec: none

## Approach

The header band image is set as an inline style in `js/react/src/header/main.jsx:287` (`OF-Header_May8.png`). Inline styles override stylesheet rules, so move the background into `js/react/src/less/header.less` on `.header-wrapper`. There, layer a gray tint (`fade(@neutral-mid, 70%)`) over the new photo, so the white nav text stays readable and the gray comes from the palette variables.

Scope confirmed with user (2026-11-26): header photo only. Magnifier icons, grid/list toggle icons, the OregonFlora logo and page banner photos are out of scope.

## Affected Code

- `images/header/header-spores.jpg` (new): user's lichen spore micrograph (`Lichen spores_header.jpg`), rotated 90° to 911x232.
- `js/react/src/header/main.jsx` (modified): remove the inline `backgroundImage` style.
- `js/react/src/less/header.less` (modified): `.header-wrapper` background color, tint overlay, photo, `background-size: cover`.

## Risks

- White nav text unreadable over the photo: tint strength chosen by screenshot check; contrast of white on `@neutral-mid` is 6.69:1.
- Header changes height when scrolling: check both the tall and short header states.
- Photo cropped badly in a 100px-tall band: choose `background-position` after the first screenshot.

## Handoff

- Last session: 2026-11-26
- Branch: `feature/palette-images` (from `dev`, which includes the PR #1 palette)
- Current state: complete. T1-T3 done and signed off; header shows the gray-tinted lichen spore photo; old `OF-Header_May8*.png` images deleted.
- Next step: push `feature/palette-images` and open a PR into `dev`.
- Open questions: none.
- Learned:
  - Source photo is only 232x911; rotated 90° to fit the band. Use a higher-res original if available.
  - After editing LESS, confirm the change reached `css/compiled/*.css` (e.g. `grep header-spores css/compiled/header.css`); a stale build left the header with no background at all.
