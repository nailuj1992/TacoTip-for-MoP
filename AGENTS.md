# AGENTS.md — TacoTip (MoP Classic)

Guidance for AI coding agents working in this repository.

## Project

TacoTip enriches World of Warcraft unit and item tooltips with GearScore, item level, talents/spec, guild, target, Pawn score and more. It also adds GearScore/iLvl indicators to the character frame and quality/iLvl/durability overlays on equipped item slots. It is a port of kebabstorm's (anzz1) original Classic addon.

- **Original author:** kebabstorm (anzz1). **MoP port maintainer:** Nailuj1992
- **CurseForge:** https://www.curseforge.com/wow/addons/tacotip-for-mop (project ID `1243220`)
- **Language:** Lua 5.1 (WoW UI environment), plus TOC and XML

### Supported clients and branches

| Branch | Target | Interface |
|---|---|---|
| `master` | MoP Classic Phase 5 (active development) | `50504` |
| `mop-stable-version-phase-4` | MoP Classic up to Phase 4 (stable) | `50500` |
| `cata-stable-version` | Cataclysm Classic (stable) | `40402` |

`main.lua` exits early on any client whose major version is outside 1–5. TacoTip does **not** run on Retail or WoW Forever.

## Layout

| File | Purpose |
|---|---|
| `TacoTip.toc` | Metadata, interface, load order |
| `embeds.xml` | Loads libraries, then `Locale/*.lua` (`enUS.lua` last, as the fallback) |
| `gearscore.lua` | GearScore calculation (`TT_GS`) |
| `pawn.lua` | Optional Pawn integration |
| `options.lua` | SavedVariables (`TacoTipConfig`), settings panel, `/tacotip` |
| `main.lua` | Tooltip hooks, character frame, item slot overlays |
| `Locale/<locale>.lua` | Translations stored in the global `TACOTIP_LOCALE` |
| `Libs/LibClassicInspector/`, `Libs/LibDetours-1.0/` | **Tracked** libraries maintained with this addon |
| `Libs/*` (everything else) | Third-party libraries. Not tracked in git; fetched by the packager |
| `.pkgmeta`, `.github/workflows/release.yml` | Packaging; tag push → CurseForge release |

## Guidelines

- **Client checks:**
  - Use `LibClassicInspector`'s helpers (`CI:IsMop()`, `CI:IsCata()`, …) instead of parsing build numbers yourself.
  - MoP has no Ranged slot, which shifts the slot counts.
- **Localisation:**
  - Every user-facing string goes through `L["..."]`.
  - When you add a key, add it to `Locale/enUS.lua` and to every other locale file.
  - Use the English text as a placeholder rather than leaving a key missing.
  - Keep each file's `Translators:` header accurate, and credit new translators in the README table.
- **Performance is a feature** ("no FPS drops"). Avoid per-frame allocations in tooltip and `OnUpdate` paths, cache lookups, and reuse tables.
- **Lua errors are bugs.** Nil-check unit tokens, GUIDs and item info: other addons (e.g. Plater) can hand over frames whose fields differ from Blizzard's.
- **Hooks:** use `LibDetours-1.0` / `hooksecurefunc`. Never replace Blizzard secure functions.
- **Saved variables:** add new options with defaults. Never rename or remove existing keys without migrating them.
- **Library changes:**
  - Changes to `LibClassicInspector` or `LibDetours-1.0` must stay backwards-compatible.
  - Bump the library's LibStub minor version, and update `API.txt` when the API changes.
- **Line endings:** `.lua` files use CRLF. Keep them that way.
- **Style:** match the surrounding code (tabs in `main.lua`, `local` everything).

### Releases

- Bump `## Version` in the TOC, add a `CHANGELOG.md` entry, and update "Latest" in the README's Changelog section, all together.
- Commit messages follow Conventional Commits with an optional scope (`fix(inspector): …`, `docs(changelog): …`).

## Restrictions

- **Never push git tags.** Any tag push triggers `release.yml` and publishes to CurseForge. Releasing is the maintainer's decision.
- **Never push to `master` or to the `*-stable-version*` branches directly.** Work on a branch and open a pull request.
- **Don't port changes between branches** (master ↔ phase-4 ↔ cata) unless asked.
- **Never commit untracked `Libs/` folders.** Only `LibClassicInspector` and `LibDetours-1.0` belong in git.
- **Don't change `.github/workflows/` or `.pkgmeta`** unless asked. Never print, log or commit secrets (`CURSE_API_TOKEN`).
- **Credits:** keep the original author credit (`## Author: kebabstorm`, README Credits) intact.
- **Don't touch the user's WoW installation** (`World of Warcraft\_classic_\Interface\AddOns`, `WTF`/SavedVariables) unless explicitly asked. Back files up before overwriting them.
- **Changes can't be run here.** No WoW client or Lua runtime is available. Say clearly that in-game testing is still needed; don't claim a change "works".
