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
`$(MARKETING_VERSION)`), so release metadata is written there, not in the plist.

## Distribute an update (so `app-update better-shot` picks it up)

The app is delivered through the HomeTool **app-update** server; users
install/update on any Mac with `app-update better-shot`.

1. **Publish** with the script in the *hometool* repo (not this one):
   ```
   ~/Pork/hometool/scripts/tool/publish-bettershot.sh                 # build + publish
   SKIP_BUILD=1 ~/Pork/hometool/scripts/tool/publish-bettershot.sh    # retry an already-reserved artifact
   DRY_RUN=1    ~/Pork/hometool/scripts/tool/publish-bettershot.sh    # reserve + build + zip, no upload
   ```
   It allocates app key `better-shot` through HomeTool, writes the returned
   number to `project.yml` and `version.json`, runs XcodeGen and `make release`,
   then stages `BetterShot.app` → **`bettershot.app`**
   (lowercase!), zips, and POSTs to `/int/app-update/better-shot/publish` with
   `build`, `build_host`, `version`, notes, and the zip. `SKIP_BUILD=1` is only
   for an artifact whose embedded build was already reserved. `DRY_RUN=1`
   consumes a reservation and leaves a gap even though it skips upload.
2. For a deliberately split build/publish flow, reserve directly with
   `BUILD="$(~/Pork/hometool/scripts/tool/hometool-build-number better-shot)"`,
   then write `BUILD` to both metadata files before XcodeGen and compilation.
3. Commit `project.yml`, `version.json`, and regenerated
   `BetterShot.xcodeproj/project.pbxproj` with the release.
4. **Verify**: `app-update better-shot` (downloads → sha256-verifies → installs →
   relaunches).

Never edit the build to `current + 1`, reuse an abandoned reservation, or
publish an unreserved build. Allocation failure stops the release.

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
