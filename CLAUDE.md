# BetterShot — Claude Code guide

BetterShot is a **native Swift + xcodegen macOS app** (menu-bar screenshot +
screen-recording tool) — Mike's fork of `KartikLabhshetwar/better-shot`, working
branch `mike_tweak`. Product `BetterShot`, bundle id `com.bettershot.app`,
installed as `~/Applications/bettershot.app`.

> There is no web/React/Tailwind code here. `AGENTS.md`'s "UI Skills" (Tailwind /
> motion/react / Base UI) is inherited template boilerplate and does **not**
> apply to this codebase — ignore it for this repo.

## Build

- `make build` — Debug → `.build/Build/Products/Debug/BetterShot.app`
- `make release` — Release, codesigned with the stable identity
  `Apple Development: Yizhou He (U4HVU5232W)` (this keeps Screen-Recording /
  Accessibility TCC grants across rebuilds) → `.build/Build/Products/Release/BetterShot.app`
- `make run` — build + launch Debug
- `make ship` is **DEAD** (it calls `scripts/release.sh`, which doesn't exist) —
  use `make release`.

**Versioning lives in two files that must stay in sync:** `version.json`
(`version`, `build`) and `project.yml` (`MARKETING_VERSION`,
`CURRENT_PROJECT_VERSION`). After editing `project.yml`, run `xcodegen generate`
— the `.xcodeproj` is **tracked**, so commit the regenerated `project.pbxproj`.
`CFBundleVersion`/`CFBundleShortVersionString` in `Resources/Info.plist` come
from those two build settings (`$(CURRENT_PROJECT_VERSION)` /
`$(MARKETING_VERSION)`), so bump them there, not in the plist.

## Distribute an update (so `app-update better-shot` picks it up)

The app is delivered through the HomeTool **app-update** server; users
install/update on any Mac with `app-update better-shot`.

1. **Bump the build number** — `CURRENT_PROJECT_VERSION` in `project.yml` **and**
   `build` in `version.json`, then `xcodegen generate` and commit. This is
   mandatory: the server serves the **highest build number** as "latest"
   (`ORDER BY build DESC`), so a build that isn't higher than the current server
   max is never served — `app-update better-shot` would keep installing the old one.
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
B), but the app-update client greps the download zip for `bettershot.app`
(lowercase, **case-sensitive**). Never hand-zip `BetterShot.app` for publishing —
the install fails with "not found in zip". The publish script does the rename; use it.

The app-update client/registry is `~/.local/bin/app-update` (source of truth
`~/Pork/hometool/scripts/py/bootstrap_mac_update.py`; registry key `better-shot`,
bundle id `com.bettershot.app`, target `~/Applications/bettershot.app`).

## Local-only update (this Mac, no distribution)

```
make release
ditto .build/Build/Products/Release/BetterShot.app ~/Applications/bettershot.app
open ~/Applications/bettershot.app
```

Install to `~/Applications/` only — never `/Applications/`, and never both.
