---
name: ui-skills
description: Opinionated constraints for building better interfaces with agents.
---

# UI Skills

Opinionated constraints for building better interfaces with agents.

> **Note for this repo:** BetterShot is a **native Swift + xcodegen macOS app**,
> not a web project — the Tailwind / motion/react / Base UI rules below are
> inherited template boilerplate and do **not** apply here. For how to build,
> sign, publish, and update the app, see
> [Build, release & update](#build-release--update) at the bottom.

## Stack

- MUST use Tailwind CSS defaults (spacing, radius, shadows) before custom values
- MUST use `motion/react` (formerly `framer-motion`) when JavaScript animation is required
- SHOULD use `tw-animate-css` for entrance and micro-animations in Tailwind CSS
- MUST use `cn` utility (`clsx` + `tailwind-merge`) for class logic

## Components

- MUST use accessible component primitives for anything with keyboard or focus behavior (`Base UI`, `React Aria`, `Radix`)
- MUST use the project’s existing component primitives first
- NEVER mix primitive systems within the same interaction surface
- SHOULD prefer [`Base UI`](https://base-ui.com/react/components) for new primitives if compatible with the stack
- MUST add an `aria-label` to icon-only buttons
- NEVER rebuild keyboard or focus behavior by hand unless explicitly requested

## Interaction

- MUST use an `AlertDialog` for destructive or irreversible actions
- SHOULD use structural skeletons for loading states
- NEVER use `h-screen`, use `h-dvh`
- MUST respect `safe-area-inset` for fixed elements
- MUST show errors next to where the action happens
- NEVER block paste in `input` or `textarea` elements

## Animation

- NEVER add animation unless it is explicitly requested
- MUST animate only compositor props (`transform`, `opacity`)
- NEVER animate layout properties (`width`, `height`, `top`, `left`, `margin`, `padding`)
- SHOULD avoid animating paint properties (`background`, `color`) except for small, local UI (text, icons)
- SHOULD use `ease-out` on entrance
- NEVER exceed `200ms` for interaction feedback
- MUST pause looping animations when off-screen
- MUST respect `prefers-reduced-motion`
- NEVER introduce custom easing curves unless explicitly requested
- SHOULD avoid animating large images or full-screen surfaces

## Typography

- MUST use `text-balance` for headings and `text-pretty` for body/paragraphs
- MUST use `tabular-nums` for data
- SHOULD use `truncate` or `line-clamp` for dense UI
- NEVER modify `letter-spacing` (`tracking-`) unless explicitly requested

## Layout

- MUST use a fixed `z-index` scale (no arbitrary `z-x`)
- SHOULD use `size-x` for square elements instead of `w-x` + `h-x`

## Performance

- NEVER animate large `blur()` or `backdrop-filter` surfaces
- NEVER apply `will-change` outside an active animation
- NEVER use `useEffect` for anything that can be expressed as render logic

## Design

- NEVER use gradients unless explicitly requested
- NEVER use purple or multicolor gradients
- NEVER use glow effects as primary affordances
- SHOULD use Tailwind CSS default shadow scale unless explicitly requested
- MUST give empty states one clear next action
- SHOULD limit accent color usage to one per view
- SHOULD use existing theme or Tailwind CSS color tokens before introducing new ones

# Build, release & update

BetterShot is a **native Swift + xcodegen macOS app** (menu-bar screenshot +
screen-recording tool) — Mike's fork of `KartikLabhshetwar/better-shot`, working
branch `mike_tweak`. Product `BetterShot`, bundle id `com.bettershot.app`,
installed as `~/Applications/bettershot.app`.

## Build

- `make build` — Debug → `.build/Build/Products/Debug/BetterShot.app`
- `make release` — Release, codesigned with the stable identity
  `Apple Development: Yizhou He (U4HVU5232W)` (keeps Screen-Recording /
  Accessibility TCC grants across rebuilds) → `.build/Build/Products/Release/BetterShot.app`
- `make run` — build + launch Debug
- `make ship` is **DEAD** (calls a missing `scripts/release.sh`) — use `make release`.

Versioning lives in two files that must stay in sync: `version.json`
(`version`, `build`) and `project.yml` (`MARKETING_VERSION`,
`CURRENT_PROJECT_VERSION`). After editing `project.yml` run `xcodegen generate`
— the `.xcodeproj` is **tracked**, so commit the regenerated `project.pbxproj`.

## Distribute an update (so `app-update better-shot` picks it up)

The app is delivered through the HomeTool **app-update** server; users
install/update on any Mac with `app-update better-shot`.

1. **Bump the build number** — `CURRENT_PROJECT_VERSION` in `project.yml` **and**
   `build` in `version.json`, then `xcodegen generate` and commit. Mandatory:
   the server serves the **highest build number** as "latest"
   (`ORDER BY build DESC`), so a build that isn't higher than the current server
   max is never served — `app-update better-shot` keeps installing the old one.
2. **Publish** with the script in the *hometool* repo (not this one):
   ```
   ~/Pork/hometool/scripts/tool/publish-bettershot.sh                 # build + publish
   SKIP_BUILD=1 ~/Pork/hometool/scripts/tool/publish-bettershot.sh    # reuse .build/…
   DRY_RUN=1    ~/Pork/hometool/scripts/tool/publish-bettershot.sh    # build + zip, no upload
   ```
   It runs `make release`, stages `BetterShot.app` → **`bettershot.app`**
   (lowercase!), zips, and POSTs to `/int/app-update/better-shot/publish` with
   `build = CFBundleVersion`, `version = CFBundleShortVersionString`.
3. **Verify**: `app-update better-shot` (downloads → sha256-verifies → installs →
   relaunches).

**Gotcha — bundle name casing:** `make release` emits `BetterShot.app` (capital
B), but the app-update client greps the zip for `bettershot.app` (lowercase,
**case-sensitive**). Never hand-zip `BetterShot.app` for publishing — the install
fails with "not found in zip". The publish script does the rename; use it.

## Local-only update (this Mac, no distribution)

```
make release
ditto .build/Build/Products/Release/BetterShot.app ~/Applications/bettershot.app
open ~/Applications/bettershot.app
```

Install to `~/Applications/` only — never `/Applications/`, and never both.
