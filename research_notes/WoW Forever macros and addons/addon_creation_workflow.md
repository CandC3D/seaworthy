# Add-on creation workflow for World of Warcraft: Forever (empty folder to published add-on)

Research date: 2026-11-12. Evidence labels used throughout:
- **[Forever-verified]** = read directly from a Forever-specific primary artefact (the `forever` branch of the exported Blizzard UI source, a live service API, or an add-on repository whose TOC declares `16001`).
- **[Wiki-Forever]** = stated for Forever on Warcraft Wiki (community-maintained, but the de facto reference).
- **[General]** = documented for WoW add-ons in general (retail/Midnight pages); applies to Forever by inference because Forever is a Mainline-family client, not separately verified in a Forever client.
- **[Community]** = one add-on author's or one guide site's statement; treat as a lead, not as fact.

Important premise correction: the brief describes Forever as "in beta as of November 2026". Every dated source found says the beta opened 2026-09-17 and the game launched on 2026-11-04; the exported UI source `forever` branch reads `1.60.1.70888` on 2026-11-10. Beta-era facts (folder `_classic_beta_`, early `WOW_PROJECT_ID = 1`) are flagged as such below. I did not find a Blizzard-hosted page confirming the launch date; it rests on press coverage, CurseForge's blog and repository evidence.

No add-on code could be run in a Forever client during this research. Everything is read from documentation and source, not executed.

---

## 1. Anatomy of an add-on (folder, TOC manifest, Lua/XML files, load order)

### Takeaway
Forever add-ons live in `World of Warcraft\_forever_\Interface\AddOns\<AddonName>\` (the beta used `_classic_beta_`), and the manifest must declare `## Interface: 16001`. Forever is a "Mainline-family" game type (`forever`, alias `camelot`), so the full modern TOC feature set applies (comma-separated interface lists, `_Forever.toc`/`_Camelot.toc`/`_Mainline.toc` suffix files, `AddonCompartmentFunc`, `IconTexture`, `Category`, `LoadSavedVariablesFirst`, inline `[AllowLoadGameType ...]` conditions).

### Cited Findings

**Install location**
- Live client folder is `World of Warcraft\_forever_`; beta folder was `World of Warcraft\_classic_beta_`. CurseForge's migration guide (published 2026-10-25) tells users to copy `Interface` and `WTF` from the beta folder into the live folder. [Community/official-platform] — [CurseForge blog](https://blog.curseforge.com/wow-forever-migrate-addons-beta-to-live/)
- BetterBags' maintainer notes record: "launched 2026-11-04 as build `1.60.1.70790`, product `wowf`, installed in `_forever_`; the beta was `wow_classic_beta`". [Community] — [BetterBags camelot-forever.md](https://github.com/Cidan/BetterBags/blob/HEAD/.claude/rules/camelot-forever.md)
- A Forever-only add-on workspace gives the Windows path `C:\Program Files (x86)\World of Warcraft\_forever_\Interface\AddOns\<Name>` and says the folder name comes from the registry `GamePath` of an installed client; to find it, use Battle.net launcher → WoW Forever → "Show in Explorer". [Community] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)
- Add-ons placed in the Retail or Classic folders do not appear in Forever; it must be the Forever install's `Interface\AddOns`. [Community] — [Kami Labs list, data checked 2026-10-20](https://kami-labs.fr/en/addons-wow-forever/)
- Saved settings live in the sibling `WTF` folder (`WTF\Account\<account>\SavedVariables\` for account-wide data, per-character folders for per-character data). [General/Wiki] — [TOC format](https://warcraft.wiki.gg/wiki/TOC_format); [CurseForge blog](https://blog.curseforge.com/wow-forever-migrate-addons-beta-to-live/)

**Folder and file naming**
- The TOC filename must match the folder name or the game will not detect the add-on (e.g. `HelloWorld\HelloWorld.toc`). [General] — [Create a WoW AddOn in 15 Minutes](https://warcraft.wiki.gg/wiki/Create_a_WoW_AddOn_in_15_Minutes)
- The same rule is followed in Forever-only repositories ("one folder per addon, named exactly like its `.toc` file"). [Forever-verified by example] — [TautelliniAddons README](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/README.md)
- A double-nested add-on folder (zip extracted one level too deep) prevents loading — a common install failure. [Community, from search summary of install guides] — [allthings.how guide](https://allthings.how/world-of-warcraft-forever-how-to-install-addons-without-the-curseforge-app/)

**Interface number and game type**
- Forever's interface number is `16001` (patch 1.60.1). Other current numbers listed on the same page (2026-11): Midnight Standard `120105` (test `120107`), Mists `50504`, Cata `40402`, Wrath `30405`, Titan Reforged `38002`, TBC `20506`, Vanilla `11509`. [Wiki-Forever] — [TOC format](https://warcraft.wiki.gg/wiki/TOC_format)
- `/dump select(4, GetBuildInfo())` prints the running client's interface number. [General] — [TOC format](https://warcraft.wiki.gg/wiki/TOC_format)
- If none of the listed interface values match the client, the add-on is classed as out of date. [General] — [TOC format](https://warcraft.wiki.gg/wiki/TOC_format); [BigWigs packager README](https://github.com/BigWigsMods/packager)
- Forever's TOC game type is `forever`, with `camelot` (internal codename) as an equivalent; the `forever` name was added in build 70291 (beta 4, 2026-10-07). Both game types belong to the `mainline` family alongside `standard` (Midnight), `plunderstorm` and `wowhack`. [Wiki-Forever] — [TOC format](https://warcraft.wiki.gg/wiki/TOC_format); [Patch 1.60.1/API changes](https://warcraft.wiki.gg/wiki/Patch_1.60.1/API_changes)

**Multi-version support**
- One TOC can list several interface values, comma-separated, e.g. `## Interface: 11509, 20506, 30405, 38002, 40402, 50504, 120100, 16001` — the packager README describes this example as marking an add-on compatible with "the latest version of all client flavors and Forever Beta". [General + Forever] — [BigWigs packager README](https://github.com/BigWigsMods/packager)
- Per-flavour TOC suffixes: `_Standard.toc` (Midnight), `_Forever.toc` / `_Camelot.toc` (Forever), `_Mists.toc`, `_Cata.toc`, `_Wrath.toc`, `_TBC.toc`, `_Vanilla.toc`; family suffixes `_Mainline.toc` (standard + forever/camelot + plunderstorm + wowhack) and `_Classic.toc` (vanilla…mists). The client picks the game-specific suffix first, then the family suffix, then the unsuffixed `AddOnName.toc`. [Wiki-Forever] — [TOC format](https://warcraft.wiki.gg/wiki/TOC_format)
- Consequence stated by a Forever author: an add-on that ships only `_Mainline.toc` also loads on Forever, but is flagged out of date unless that file lists `16001`. [Community] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)
- Inline per-line conditions: `ForeverOnly.lua [AllowLoadGameType camelot][ExcludeLoadGameType standard, classic]`. The wiki's 1.60.1 page recommends pairing `AllowLoadGameType: forever` with `ExcludeLoadGameType: standard, classic` for clients that do not yet recognise `forever`; an unknown game type in a condition is treated as satisfied. [Wiki-Forever] — [Patch 1.60.1/API changes](https://warcraft.wiki.gg/wiki/Patch_1.60.1/API_changes); [BigWigs packager README](https://github.com/BigWigsMods/packager); [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)
- TOC variable substitutions usable in file paths: `[Family]` → `Mainline`/`Classic`; `[Game]` → `Standard`, `Camelot`, `Vanilla`, …; `[TextLocale]` → `enUS`, `frFR`, …; `[RegionalTextLocale]` supported "from Midnight 12.1.7 and Forever". [Wiki-Forever] — [TOC format](https://warcraft.wiki.gg/wiki/TOC_format)
- Forever-specific metadata lines in a shared TOC must come first, because later lines overwrite earlier ones on other clients. [Wiki-Forever] — [Patch 1.60.1/API changes](https://warcraft.wiki.gg/wiki/Patch_1.60.1/API_changes)

**TOC directives (all [General], listed on the wiki page that also covers Forever)** — [TOC format](https://warcraft.wiki.gg/wiki/TOC_format)
- `Title`, `Notes` (both localisable, e.g. `Title-frFR`), `Author`, `Version` (readable with `C_AddOns.GetAddOnMetadata`).
- `SavedVariables` (account-wide globals) and `SavedVariablesPerCharacter` (per-character globals).
- `Dependencies` (aliases `RequiredDeps`, any `Dep*`): must load first. `OptionalDeps`: load first if present.
- `LoadOnDemand: 1` (loads only via `C_AddOns.LoadAddOn`), `LoadWith`, `LoadManagers`, `DefaultState: disabled`.
- `AddonCompartmentFunc`, `AddonCompartmentFuncOnEnter`, `AddonCompartmentFuncOnLeave`: names of global functions for the minimap add-on compartment; the first is required for automatic registration.
- `IconTexture` / `IconAtlas` (add-on list icon), `Category` and `Group` (add-on list grouping).
- `AllowLoadGameType`, `ExcludeLoadGameType`, `LoadSavedVariablesFirst: 1`, `OnlyBetaAndPTR: 1`.
- Restricted (Blizzard-only) directives exist and are not usable by third parties: `AllowLoad`, `LoadFirst`, `SavedVariablesMachine`, `UseSecureEnvironment`.

**Directives seen in real Forever TOCs [Forever-verified by example]**
- `AddonCompartmentFunc: LegacyForever_OnAddonCompartmentClick`, `LoadSavedVariablesFirst: 1`, `Dependencies`, `OptionalDeps`, `Category: Map`, `IconTexture` in a Forever-only TOC. — [legacy-forever LegacyForever.toc](https://github.com/cjber/legacy-forever/blob/HEAD/LegacyForever.toc)
- `LoadSavedVariablesFirst: 1`, `SavedVariables` + `SavedVariablesPerCharacter`, localised `Notes-xxXX`/`Category-xxXX` in Questie's Forever TOC. — [Questie_Forever.toc](https://github.com/Questie/Questie/blob/HEAD/Questie_Forever.toc)
- `IconTexture` shows at 20×20 in the add-on list; without it the list shows a question mark; a 64×64 RGBA `.tga` is recommended. [Community] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)
- The Blizzard add-on compartment source file exists on the Forever branch (`Blizzard_Minimap/Mainline/AddonCompartment.lua` returned HTTP 200 on 2026-11-12). [Forever-verified] — [wow-ui-source forever branch](https://github.com/Gethe/wow-ui-source/tree/forever)

**Minimal Forever manifest (from a Forever-only template repository)** — [ForeverAddonTemplate.toc](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/templates/ForeverAddonTemplate/ForeverAddonTemplate.toc)
```
## Interface: 16001
## Title: Forever Addon Template
## Notes: Starting point for a WoW Forever addon
## Author: Tautellini
## Version: @project-version@
## SavedVariables: ForeverAddonTemplateDB

ForeverAddonTemplate.lua
```
Plain English: line 1 says "built for client 16001"; the next lines are the name, tooltip, author and version shown in the in-game add-on list (`@project-version@` is a placeholder the release packager replaces with the git tag); `SavedVariables` names one global table the client will save to disk; the last line is the list of code files to run, in order.

**Files and load order [General]** — [AddOn loading process](https://warcraft.wiki.gg/wiki/AddOn_loading_process)
- Blizzard's own add-ons load first, then third-party add-ons alphabetically, modified by TOC directives (dependencies, load-on-demand).
- Within an add-on, Lua and XML files run in the order listed in the TOC; XML files can pull in more files via `Include`/`Script` tags, executed when reached.
- Saved variables are loaded after the last listed file by default; `LoadSavedVariablesFirst` (added in patch 11.1.5, 2025-04-22) loads them before any file.
- `#` lines in the file list are comments; real Forever TOCs use `# section` comments and comment out file lines. [Forever-verified by example] — [ArcHUD3_Camelot.toc](https://github.com/nyyr/ArcHUD3/blob/HEAD/ArcHUD3_Camelot.toc)

**When a restart is needed** — sources conflict
- Wiki tutorial: `/reload` tests code changes and "can replace a client restart when updating the TOC file". [General] — [Create a WoW AddOn in 15 Minutes](https://warcraft.wiki.gg/wiki/Create_a_WoW_AddOn_in_15_Minutes)
- Forever author: "Edits to `.lua` files need a `/reload`. New files and TOC changes need a full client restart." [Community, Forever] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)

### Inferences
- Safest layout for a Forever-only add-on is one unsuffixed `MyAddon.toc` with `## Interface: 16001`; for Forever + Midnight with identical files use `## Interface: 120105, 16001`; only split into suffix TOCs when the file lists differ. (Follows from the suffix lookup order and matches the layout table in the Tautellini guide.)
- The interface number follows the usual `major*10000 + minor*100 + patch` pattern (1.60.1 → 16001), so a future 1.61.0 would be `16100` and every Forever add-on will be flagged out of date until its TOC is bumped. One repository's tooling comment already anticipates "1.61.0 is 16100" ([j1d2/wow-addons](https://github.com/j1d2/wow-addons/blob/HEAD/scripts/client_watch.sh)); this is a projection, not an announced patch.
- On the restart conflict: the conservative rule is to restart the client when adding a brand-new add-on folder or new files, and `/reload` for edits to existing files.

### Gaps
- No Blizzard-authored documentation page for the Forever folder name was found; `_forever_` rests on CurseForge's blog, press guides and add-on author notes (all consistent).
- macOS path for the Forever client not found (expected `/Applications/World of Warcraft/_forever_/Interface/AddOns`, unverified).
- Whether the "Load out of date AddOns" checkbox still exists and behaves identically on Forever was not verified in-client.

---

## 2. Language and runtime (Lua, sandbox, namespace, frames, events, slash commands, saved variables, timers, localisation, settings, hooks, mixins)

### Takeaway
The runtime is Blizzard's modified Lua 5.1 with no OS or file I/O; each file receives `(addonName, namespaceTable)` through `...`; everything is event-driven via frames. All of the general patterns apply on Forever, and real Forever add-ons use exactly the standard ones (one event frame, `ADDON_LOADED` for saved variables, `SLASH_X1` globals, the modern `Settings` API). The Forever-specific twist is that Midnight's "secret values" are active, so code must tolerate API returns it cannot do arithmetic on.

### Cited Findings

**Lua version and sandbox**
- "World of Warcraft uses a modified version of Lua 5.1 that supports taint"; "operating system and file I/O libraries are not present". Available: base functions incl. `loadstring`, `pcall`, `xpcall`, `setfenv`, `getfenv`, `select`; libraries `math`, `string`, `table`, `coroutine`, `bit`. WoW additions: `strsplit`, `strjoin`, `strtrim`, `tinsert`, `wipe`, `tContains`, `table.freeze`, `Clamp`, `Lerp`, `Round`, etc. [General] — [Lua functions](https://warcraft.wiki.gg/wiki/Lua_functions)
- Forever repositories lint with `std = "lua51"` and set LuaLS `"runtime.version": "Lua 5.1"`. [Forever-verified by example] — [ForeverLootSparkles .luacheckrc](https://github.com/svengabr/ForeverLootSparkles/blob/HEAD/.luacheckrc); [legacy-forever .luarc.json](https://github.com/cjber/legacy-forever/blob/HEAD/.luarc.json)
- New in the Forever diff: a `require` function appears in the added-API list (1.60.1 vs 12.1.5), and the Beta 4 notes say "The require functionality recently added to Lua has been renamed to import." [Wiki-Forever] — [Patch 1.60.1/API changes](https://warcraft.wiki.gg/wiki/Patch_1.60.1/API_changes)
- Secret-value helper functions on Forever: `issecretvalue(v)`, `hasanysecretvalues(...)`, `issecrettable(t)`, `canaccessvalue(v)`, `canaccesstable(t)`, `canaccesssecrets()`, `scrubsecretvalues(...)` (listed from `FrameScriptDocumentation.lua`). [Community, citing Forever source] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)

**Add-on namespace and globals hygiene**
- Every file gets two arguments via `...`: `local addonName, ns = ...`; `ns` is a private table shared by all files of the same add-on, used instead of globals. [General] — [Create a WoW AddOn in 15 Minutes](https://warcraft.wiki.gg/wiki/Create_a_WoW_AddOn_in_15_Minutes)
- Forever practice: "Only SavedVariables, slash command entries and deliberate public APIs belong in globals", and each is whitelisted in `.luacheckrc` so an accidental global fails lint. [Forever-verified by example] — [TautelliniAddons CLAUDE.md](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/CLAUDE.md); [ForeverLootSparkles .luacheckrc](https://github.com/svengabr/ForeverLootSparkles/blob/HEAD/.luacheckrc)

**Frames, events and the standard skeleton**
- Events are received by registering them on a frame and setting an `OnEvent` script: `local f = CreateFrame("Frame"); f:RegisterEvent("PLAYER_LEVEL_UP"); f:SetScript("OnEvent", function() ... end)`; for many events dispatch by name (`self[event](self, event, ...)`). [General] — [Create a WoW AddOn in 15 Minutes](https://warcraft.wiki.gg/wiki/Create_a_WoW_AddOn_in_15_Minutes)
- Working Forever skeleton (template repository, Interface 16001) — [ForeverAddonTemplate.lua](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/templates/ForeverAddonTemplate/ForeverAddonTemplate.lua):
```lua
local addonName, ns = ...
ns.isForever = WOW_PROJECT_ID == (WOW_PROJECT_CAMELOT or 18)
local defaults = { enabled = true }

local frame = CreateFrame("Frame")
frame:SetScript("OnEvent", function(self, event, ...) self[event](self, ...) end)
frame:RegisterEvent("ADDON_LOADED")
frame:RegisterEvent("PLAYER_LOGIN")

function frame:ADDON_LOADED(name)
    if name ~= addonName then return end
    ForeverAddonTemplateDB = ForeverAddonTemplateDB or {}
    for key, value in pairs(defaults) do
        if ForeverAddonTemplateDB[key] == nil then ForeverAddonTemplateDB[key] = value end
    end
    ns.db = ForeverAddonTemplateDB
    self:UnregisterEvent("ADDON_LOADED")
end

SLASH_FOREVERADDONTEMPLATE1 = "/fat"
SlashCmdList.FOREVERADDONTEMPLATE = function() print(addonName .. ": hello") end
```
Plain English: grab the add-on's name and private table; note whether this is the Forever client; create an invisible frame whose only job is to receive events and route each to a method of the same name; when *this* add-on finishes loading, create the saved-settings table if it does not exist, fill in any missing defaults, and stop listening; finally register a `/fat` chat command. (Last two lines lightly abridged from the template.)
- Frame names and children on Forever differ from Midnight in places (player, target, pet, target-of-target, totem, focus frames and the minimap header have Forever skins; some child frames moved): "Check that a frame exists before you anchor to it or skin it. Use `/fstack` to find current names." [Community, Forever] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)
- A frame parented to `WorldFrame` with strata `BACKGROUND` and frame level 0 draws over the 3D world and under the UI (tested in-game on Forever by that author). Texture blend modes are `DISABLE`, `BLEND`, `ALPHAKEY`, `ADD`, `MOD`. [Community, Forever] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)
- Forever 1.60.1 added a `FrameStrata` secret aspect: `GetFrameStrata` returns a secret value for frames that carry it. [Wiki-Forever] — [Patch 1.60.1/API changes](https://warcraft.wiki.gg/wiki/Patch_1.60.1/API_changes)
- XML is still used on Forever for templates and library embedding (e.g. `templates\container.xml`, `Frames.xml`, `embeds.xml` in Forever TOCs); XML template `OnLoad` order is inherited template first, then children, then the parent. [Forever-verified by example + General] — [BetterBags_Camelot.toc](https://github.com/Cidan/BetterBags/blob/HEAD/BetterBags_Camelot.toc); [AddOn loading process](https://warcraft.wiki.gg/wiki/AddOn_loading_process)

**Event flow at login/reload [General]** — [AddOn loading process](https://warcraft.wiki.gg/wiki/AddOn_loading_process)
1. `ADDON_LOADED(addOnName)` — fires per add-on after all its files and saved variables are loaded; "the earliest time for an AddOn to read its saved variables". Check the name argument, then unregister.
2. `PLAYER_LOGIN` — once, during the loading screen, before the first `PLAYER_ENTERING_WORLD`; most world info available; do one-time setup.
3. `PLAYER_ENTERING_WORLD(isInitialLogin, isReloadingUi)` — every loading screen (login, reload, instance change).
4. `PLAYER_REGEN_DISABLED` — entering combat lockdown (secure frames become unalterable).
5. `SPELLS_CHANGED` — spellbook populated.
- `VARIABLES_LOADED` "is no longer a reliable part of the addon loading process" (since 3.0.2).
- Logout: `PLAYER_LEAVING_WORLD` → `PLAYER_LOGOUT` ("last chance to modify SavedVariables", wiki marks citation-needed) → `ADDONS_UNLOADING`; saved variables are not committed on a crash or Alt-F4.
- A Forever-only add-on registers exactly `ADDON_LOADED`, `PLAYER_LOGIN`, `PLAYER_REGEN_ENABLED` (and `CVAR_UPDATE` via `pcall`). [Forever-verified by example] — [ForeverLootSparkles.lua](https://github.com/svengabr/ForeverLootSparkles/blob/HEAD/ForeverLootSparkles.lua)
- Forever-only event/API: `DIEL_CYCLE_CHANGED` (payload `isDayTime`) and `C_DateAndTime.IsDayTime()`, used by Blizzard's `Blizzard_Minimap/Camelot/Diel.lua`. [Community, citing Forever source] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)

**Slash commands [General]**
- Define globals `SLASH_<NAME>1 = "/cmd"`, `SLASH_<NAME>2 = "/alias"` and a handler `SlashCmdList.<NAME> = function(msg, editBox) ... end`; any `SLASH_*` global is registered automatically. — [Create a WoW AddOn in 15 Minutes](https://warcraft.wiki.gg/wiki/Create_a_WoW_AddOn_in_15_Minutes)

**Saved variables lifecycle [General]**
- Declare the global name in the TOC; read only after `ADDON_LOADED`; initialise with `DB = DB or {}`; merge a defaults table so new keys appear for existing users; inspect with `/dump DB` or `/tinspect DB`; reset with `DB = nil; ReloadUI()`. — [Create a WoW AddOn in 15 Minutes](https://warcraft.wiki.gg/wiki/Create_a_WoW_AddOn_in_15_Minutes)
- `SAVED_VARIABLES_TOO_LARGE` fires if the saved data exhausts memory. — [AddOn loading process](https://warcraft.wiki.gg/wiki/AddOn_loading_process)
- Forever-specific advice: cache runtime-discovered item/quest data in SavedVariables rather than shipping static Classic tables, because Forever hides item stats until first drop and adds new content; store a schema version once migrations are needed. [Community, Forever] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)

**Timers and performance**
- "Prefer events and `C_Timer` over `OnUpdate`. Throttle `OnUpdate` when you cannot avoid it"; reuse frames with `CreateFramePool`. `C_Timer` is in the Forever template's allowed-globals list. [Community, Forever] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md); [TautelliniAddons .luacheckrc](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/.luacheckrc)

**Localisation**
- TOC-level: `Title-xxXX`, `Notes-xxXX`, `Category-xxXX` lines; file-level: `[AllowLoadTextLocale enUS, frFR]` on a file line or `Locales\[TextLocale].lua` path substitution. [General/Wiki-Forever] — [TOC format](https://warcraft.wiki.gg/wiki/TOC_format)
- Pattern in a Forever-only add-on: `Locales\enUS.lua` first as fallback, then one file per locale (deDE, esES, esMX, frFR, koKR, ptBR, ruRU, zhCN, zhTW). [Forever-verified by example] — [legacy-forever LegacyForever.toc](https://github.com/cjber/legacy-forever/blob/HEAD/LegacyForever.toc)
- Framework route: `AceLocale-3.0` string tables per language (used by ArcHUD3, which ships a Forever TOC). — [ArcHUD3 AGENTS.md](https://github.com/nyyr/ArcHUD3/blob/HEAD/AGENTS.md)
- Caution from one Forever author: "on WoW: Forever GetLocale() answers enUS while the spell text is German", and Forever's own non-English strings are "unverified" against Classic Era text. [Community, single source — verify] — [DoesItDie Locale.lua](https://github.com/swirllyman/DoesItDie/blob/HEAD/DoesItDie/Locale.lua)

**Settings panel API**
- Modern API (replaced `InterfaceOptionsFrame_OpenToCategory` in 10.0.0): for a custom frame, `local cat = Settings.RegisterCanvasLayoutCategory(panel, name, name); Settings.RegisterAddOnCategory(cat)`; open with `Settings.OpenToCategory(...)`. [General] — [Create a WoW AddOn in 15 Minutes](https://warcraft.wiki.gg/wiki/Create_a_WoW_AddOn_in_15_Minutes)
- Used on Forever: a Forever-only add-on builds its options with `Settings.RegisterVerticalLayoutCategory("Forever Loot Sparkles")`, `Settings.RegisterAddOnSetting(category, "FOREVER_LOOT_SPARKLES_ENABLED", "enabled", db, Settings.VarType.Boolean, "Show quest item sparkles", Settings.Default.True)`, `Settings.CreateCheckbox(category, setting, ...)`, `Settings.RegisterAddOnCategory(category)`; the panel appears under Esc > Options > AddOns. Plain English: create an auto-laid-out options page, bind one checkbox to the `enabled` key of the saved table, and list the page under AddOns. [Forever-verified by example] — [ForeverLootSparkles.lua](https://github.com/svengabr/ForeverLootSparkles/blob/HEAD/ForeverLootSparkles.lua)
- `Blizzard_Settings_Shared/Blizzard_Settings.lua` exists on the Forever branch (HTTP 200, 2026-11-12). [Forever-verified] — [wow-ui-source forever branch](https://github.com/Gethe/wow-ui-source/tree/forever)
- AceConfigDialog note (applies from 12.0): `AddToBlizOptions` returns a category ID as second value that must be stored and passed to `Settings.OpenToCategory`. [General] — [Ace3 changelog r1390](https://github.com/WoWUIDev/Ace3/blob/master/changelog.txt)
- Taint trap seen on Forever: a settings header with a "shown" predicate got Blizzard's options search tainted (author's CurseForge issue #4), so the author avoided it. [Community, Forever] — [ForeverLootSparkles AGENTS.md](https://github.com/svengabr/ForeverLootSparkles/blob/HEAD/AGENTS.md)

**Hooking and mixins**
- "Never overwrite Blizzard functions or frame methods. Use `hooksecurefunc` and `frame:HookScript`"; defer protected changes while `InCombatLockdown()` is true until `PLAYER_REGEN_ENABLED`; use `SecureActionButtonTemplate` for anything that casts/targets/uses items. [Community, Forever] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)
- Mixins: Blizzard's API documentation add-on on the Forever branch is itself built from mixin files (`BaseAPIMixin.lua`, `FunctionsAPIMixin.lua`, …), and Forever add-ons reference mixins such as `MinimalSliderWithSteppersMixin`. [Forever-verified] — [Blizzard_APIDocumentation.toc on forever branch](https://github.com/Gethe/wow-ui-source/blob/forever/Interface/AddOns/Blizzard_APIDocumentation/Blizzard_APIDocumentation.toc); [TautelliniAddons .luacheckrc](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/.luacheckrc)

**Detecting the Forever client at runtime**
- Blizzard's Forever-only constants file sets `WOW_PROJECT_CAMELOT = 18; WOW_PROJECT_ID = WOW_PROJECT_CAMELOT;`, loaded via `Camelot\ProjectConstants.lua [AllowLoadGameType camelot]`. [Forever-verified] — [ProjectConstants.lua on forever branch](https://github.com/Gethe/wow-ui-source/blob/forever/Interface/AddOns/Blizzard_ProjectConstants/Camelot/ProjectConstants.lua)
- This was added in build 70170 (2026-10-01); earlier beta builds reported the retail project ID, which broke add-ons that tested `WOW_PROJECT_ID == WOW_PROJECT_MAINLINE` to mean "modern client". [Wiki-Forever + Community] — [Patch 1.60.1/API changes](https://warcraft.wiki.gg/wiki/Patch_1.60.1/API_changes); [BetterBags camelot-forever.md](https://github.com/Cidan/BetterBags/blob/HEAD/.claude/rules/camelot-forever.md)
- Idioms in major add-ons: DBM `private.isForever = WOW_PROJECT_ID == (WOW_PROJECT_CAMELOT or 18)`; ElvUI `E.Forever = WOW_PROJECT_ID == WOW_PROJECT_CAMELOT`; BetterBags guards with `WOW_PROJECT_CAMELOT ~= nil and ...` because `nil == nil` is true when neither global exists. [Forever-verified by example] — [DBM GameVersion.lua](https://github.com/DeadlyBossMods/DeadlyBossMods/blob/HEAD/DBM-Core/modules/objects/GameVersion.lua); [ElvUI Initialize.lua](https://github.com/tukui-org/ElvUI/blob/HEAD/ElvUI/Game/Shared/General/Initialize.lua); [BetterBags camelot-forever.md](https://github.com/Cidan/BetterBags/blob/HEAD/.claude/rules/camelot-forever.md)
- "Never detect Camelot with `GetBuildInfo`/TOC version checks. The 16001 interface number is not a reliable discriminator"; prefer feature detection (`if C_DamageMeter then`, probing `Enum.BagIndex`). [Community] — [BetterBags camelot-forever.md](https://github.com/Cidan/BetterBags/blob/HEAD/.claude/rules/camelot-forever.md); [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)

### Inferences
- A Forever beginner can follow the retail-era wiki tutorial ("Create a WoW AddOn in 15 Minutes") almost verbatim, changing only the folder (`_forever_`) and interface number (`16001`); every construct it teaches appears in Forever-only repositories.
- Because secrets are active, the "hello world" level is unaffected, but anything reading unit health, auras, threat or combat data needs an `issecretvalue` guard (detailed coverage belongs to the secure-execution researcher).

### Gaps
- The full list of frame types (Frame, Button, StatusBar, EditBox, ScrollFrame, GameTooltip, …), anchor points (`SetPoint`) and the strata ladder (BACKGROUND → TOOLTIP) was not fetched from a source in this session. These are documented at [Widget API](https://warcraft.wiki.gg/wiki/Widget_API) (page not read here); treat any such list as general retail knowledge, unverified for Forever.
- `Mixin()` / `CreateFromMixins()` and `hooksecurefunc` reference pages were not fetched; only their use in Forever code and guidance was observed.
- Whether `import` (ex-`require`) is usable by third-party add-ons, and what it does, was not established — only that it exists in the API diff.
- The DoesItDie claim about `GetLocale()` returning enUS on non-English Forever clients is a single unconfirmed observation.

---

## 3. Which API surface the Forever client exposes, and where the authoritative reference is

### Takeaway
Forever exposes the modern Mainline (Midnight) API — the `C_` namespace world — not the Classic Era API: Blizzard's statement (2026-09-15, as quoted on Warcraft Wiki) is that Forever "shares Mainline WoW's UI architecture, including the vast majority of APIs available in 12.1.5", with Midnight's add-on restrictions and secret values active. It is best described as "retail API, vanilla-based content, plus a small Forever-only delta". The authoritative references are the `forever` branch of Gethe/wow-ui-source (including `Blizzard_APIDocumentationGenerated`), the in-game `/api` command, and Warcraft Wiki's `Patch 1.60.1/API changes` diff.

### Cited Findings
- Blizzard (2026-09-15): "WoW Forever shares Mainline WoW's UI architecture, including the vast majority of APIs available in 12.1.5." and "The addon disarmament changes introduced in Midnight, including secrets, will also be active in Forever." Camelot and Standard are game types inside the Mainline code family. [Wiki-Forever, quoting Blizzard posts linked from Discord] — [Patch 1.60.1/API changes](https://warcraft.wiki.gg/wiki/Patch_1.60.1/API_changes)
- Diff of 12.1.5 (69952) → 1.60.1 (70205), dated 2026-10-02: Global API 176 added, 10 removed; FrameXML 471 added, 1041 removed. Removed include `AbandonSkill`, `GetPetFoodTypes`, some `C_PvP` training-ground and `C_TransmogOutfitInfo` functions; added include `C_Flyout.*`, `C_SkillInfo.*`, `C_Traits.*`, `C_VideoOptions.*`, `C_GameRules.IsHardcoreActive`, `C_Spell.GetItemCooldown`, `UnitUsesAmmo`, `UnitIsInInteractRange`. [Wiki-Forever; grouping of removed/added was reconstructed from a flattened list] — [Patch 1.60.1/API changes](https://warcraft.wiki.gg/wiki/Patch_1.60.1/API_changes)
- Behaviour changes on Forever: `C_PvP.IsWarModeFeatureEnabled` returns false; `GetInspectHonorData` returns two extra values; `SendChatMessage` with `"EMOTE"` needs a hardware event outside instances; trial accounts cannot send add-on messages (2026-10-08); creature/vehicle/pet/game-object GUID server and spawn IDs are obfuscated (Beta 5, 2026-10-16) to prevent layer detection. [Wiki-Forever] — [Patch 1.60.1/API changes](https://warcraft.wiki.gg/wiki/Patch_1.60.1/API_changes)
- Add-on authors' characterisation: "a mainline retail fork: the engine and UI are retail"; "retail API with vanilla content"; "Addons written for Classic Era do not load, and Classic API habits do not carry over. Addons written for Midnight usually load and are the right starting point." [Community] — [BetterBags camelot-forever.md](https://github.com/Cidan/BetterBags/blob/HEAD/.claude/rules/camelot-forever.md); [ArcHUD3 AGENTS.md](https://github.com/nyyr/ArcHUD3/blob/HEAD/AGENTS.md); [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)
- Modern namespaces in use by Forever-only add-ons: `C_Container`, `C_Item`, `C_Spell`, `C_TooltipInfo`, `C_Map`, `C_CVar`, `C_Timer`, `C_ChatInfo`, `C_DamageMeter`, `C_CurveUtil`, `C_DurationUtil`, `C_AddOns`, `Settings`, `MenuUtil`, `ScrollUtil`/`CreateDataProvider`. [Forever-verified by example] — [TautelliniAddons .luacheckrc](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/.luacheckrc)
- Content-shaped differences despite the retail API: 9 character bank tabs and no Warband bank although `Enum.BagIndex` still declares account-tab entries; Forever has a native damage meter, Cooldown Manager and click casting; Edit Mode layouts saved separately for keyboard/mouse and gamepad. [Community] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md); [BetterBags camelot-forever.md](https://github.com/Cidan/BetterBags/blob/HEAD/.claude/rules/camelot-forever.md)
- Restrictions: `COMBAT_LOG_EVENT_UNFILTERED` is marked `HasRestrictions` in the API docs; "The combat log is closed to addons". [Community] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md); [Kami Labs](https://kami-labs.fr/en/addons-wow-forever/)

**Reference sources**
- Gethe/wow-ui-source has a `forever` branch (last updated 2026-11-10; `version.txt` = `1.60.1.70888`), alongside `live`, `ptr`, `classic`, `classic_era`, `classic_anniversary`, `classic_titan`. There is no `camelot` or `1.60` branch. [Forever-verified] — [wow-ui-source branches](https://github.com/Gethe/wow-ui-source/branches/all); [version.txt](https://github.com/Gethe/wow-ui-source/blob/forever/version.txt)
- Machine-readable API docs with secret annotations (`SecretReturns`, `SecretArguments = "AllowedWhenTainted"`, `HasRestrictions`) are in `Interface/AddOns/Blizzard_APIDocumentationGenerated` on that branch; authors are advised to confirm every API there rather than trusting Midnight-only docs. [Community, pointing at Forever source] — [TautelliniAddons CLAUDE.md](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/CLAUDE.md)
- In-game: the API "is officially documented in Blizzard_APIDocumentation which is accessible via the /api command". [General] — [World of Warcraft API](https://warcraft.wiki.gg/wiki/World_of_Warcraft_API). `Blizzard_APIDocumentation` exists as a load-on-demand add-on on the Forever branch. [Forever-verified] — [Blizzard_APIDocumentation.toc](https://github.com/Gethe/wow-ui-source/blob/forever/Interface/AddOns/Blizzard_APIDocumentation/Blizzard_APIDocumentation.toc)
- Warcraft Wiki: main API page was "up to date as of PTR Patch 12.1.5 (69594) Aug 28 2026" and lists Forever in its navigation; the Forever delta is on [Patch 1.60.1/API changes](https://warcraft.wiki.gg/wiki/Patch_1.60.1/API_changes), with Blizzard's notes at [Patch 1.60.1/API changes/notes](https://warcraft.wiki.gg/wiki/Patch_1.60.1/API_changes/notes) and source diffs at `github.com/Gethe/wow-ui-source/compare/12.1.5..forever`. — [World of Warcraft API](https://warcraft.wiki.gg/wiki/World_of_Warcraft_API)
- Ketho/BlizzardInterfaceResources has a `forever` branch and a `1.60.1` tag (enums, global strings, API dumps). [Forever-verified via `git ls-remote`, 2026-11-12] — [BlizzardInterfaceResources](https://github.com/Ketho/BlizzardInterfaceResources)
- Blizzard's public beta development notes thread (Kaivax; 2026-09-24, 10-01, 10-08, 10-16) contains gameplay/UI notes but, in the portion read, no add-on API statements; the add-on-author notes were posted on Discord and mirrored on the wiki. — [Blizzard forums](https://us.forums.blizzard.com/en/wow/t/wow-forever-beta-development-notes-%E2%80%93-updated-september-24/2360696)

### Inferences
- "Hybrid" is the wrong mental model for the *API*; it is the Mainline API with a small Forever delta. "Hybrid" is right for *content and data*: Classic-shaped world, items and class kits behind retail-shaped functions, so retail add-ons can crash on missing content assumptions (e.g. bank layout) and Classic data tables are incomplete.
- When Midnight-era wiki pages and the Forever branch disagree, the Forever branch wins; wiki pages are written against retail first.

### Gaps
- Townlong Yak: no evidence found either way on whether it hosts Forever FrameXML/API builds; not checked directly.
- The original Blizzard posts (2026-09-15 etc.) were only seen as quoted on Warcraft Wiki; the primary Discord/forum posts were not retrieved, and a web search for the exact quote found no Blizzard-hosted copy.
- Whether the wiki's per-function pages carry a Forever availability flag was not established.

---

## 4. Libraries and frameworks (Ace3, LibStub, CallbackHandler, LibDataBroker, LibDBIcon, LibSharedMedia)

### Takeaway
Ace3 officially declares Forever support (its TOC lists `16001`), and large add-ons ship Ace3, LibStub, CallbackHandler, LibDataBroker, LibDBIcon and LibSharedMedia in their Forever builds. Libraries are embedded by listing them first in the TOC (or an `embeds.xml`) and fetched at release time via `.pkgmeta` externals. Small Forever-only add-ons generally use no framework at all.

### Cited Findings
- Ace3's TOC: `## Interface: 11508, 11509, 16001, 20505, 20506, 30405, 38002, 40402, 50503, 50504, 120100, 120105, 120107`, loading `LibStub\LibStub.lua`, `CallbackHandler-1.0`, `AceAddon-3.0`, `AceEvent-3.0`, `AceTimer-3.0`, `AceBucket-3.0`, `AceHook-3.0`, `AceDB-3.0`, `AceDBOptions-3.0`, `AceLocale-3.0`, … [Forever-verified: declares 16001] — [Ace3.toc](https://github.com/WoWUIDev/Ace3/blob/master/Ace3.toc)
- Ace3's `.luacheckrc` whitelists `WOW_PROJECT_CAMELOT`, i.e. the library code is Forever-aware. [Forever-verified] — [Ace3 .luacheckrc](https://github.com/WoWUIDev/Ace3/blob/master/.luacheckrc)
- Latest Ace3 release in the changelog is r1403 (2026-08-12: AceGUI fixes for 12.1, AceComm/ChatThrottleLib "Fixed errors from secrets during chat lockdown"); the changelog has no entry that names Forever. — [Ace3 changelog](https://github.com/WoWUIDev/Ace3/blob/master/changelog.txt)
- BetterBags' Forever TOC embeds, in order: `libs\LibStub\LibStub.lua`, `CallbackHandler-1.0.xml`, `AceAddon-3.0.xml`, `AceDB-3.0.xml`, `AceHook-3.0.xml`, `AceConsole-3.0.xml`, `AceEvent-3.0.xml`, `AceGUI-3.0.xml`, `AceConfig-3.0.xml`, `LibSharedMedia-3.0\lib.xml`, `LibWindow-1.1`, `LibDataBroker-1.1.lua`, `LibUIDropDownMenu.xml`. [Forever-verified by example] — [BetterBags_Camelot.toc](https://github.com/Cidan/BetterBags/blob/HEAD/BetterBags_Camelot.toc)
- Questie's Forever TOC lists `OptionalDeps: ... Ace3, CallbackHandler-1.0, HereBeDragons, LibDataBroker-1.1, LibDBIcon-1.0, LibSharedMedia-3.0, LibStub, LibUIDropDownMenu` and loads libraries via `embeds.xml`, after a compatibility file that "includes Forever's library bridges, so it must precede embedded libraries". [Forever-verified by example] — [Questie_Forever.toc](https://github.com/Questie/Questie/blob/HEAD/Questie_Forever.toc)
- Embedding mechanics: libraries are not committed; `.pkgmeta` `externals:` maps a target folder to a repository URL (e.g. `Libs/AceAddon-3.0: https://repos.wowace.com/wow/ace3/trunk/AceAddon-3.0`, `libs/LibDataBroker-1.1: https://github.com/tekkub/libdatabroker-1-1`, `libs/LibSharedMedia-3.0: https://repos.wowace.com/wow/libsharedmedia-3-0/trunk/LibSharedMedia-3.0`), and the packager pulls them into the zip. Listing the libraries in `OptionalDeps` lets a standalone copy load first if the user has one. [Forever-verified by example] — [BetterBags .pkgmeta](https://github.com/Cidan/BetterBags/blob/HEAD/.pkgmeta); [ArcHUD3 .pkgmeta](https://github.com/nyyr/ArcHUD3/blob/HEAD/.pkgmeta)
- Ace3 object creation as used by an add-on with a Forever TOC: `ArcHUD = LibStub("AceAddon-3.0"):NewAddon("ArcHUD", "AceConsole-3.0", "AceEvent-3.0", "AceHook-3.0", "AceTimer-3.0")` with AceDB for profiles and AceConfig/AceGUI for options. — [ArcHUD3 AGENTS.md](https://github.com/nyyr/ArcHUD3/blob/HEAD/AGENTS.md)
- Not every library is Forever-clean: vendored `LibUIDropDownMenu` "detects Forever as `projectID == WOW_PROJECT_MAINLINE` with a 1.x interface, so on 70170+ it falls into none of its flavor modes. That needs an upstream fix". [Community] — [BetterBags camelot-forever.md](https://github.com/Cidan/BetterBags/blob/HEAD/.claude/rules/camelot-forever.md)
- Rule of thumb from a Forever author: "Before depending on a library, check that it has a release with `16001` in its TOC." [Community] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)
- No-framework examples: DoesItDie (16 Lua files), ForeverLootSparkles (1 Lua file), WorldLight (3 Lua files) and the Tautellini template use only the namespace table, one event frame and the `Settings` API. [Forever-verified by example] — [DoesItDie.toc](https://github.com/swirllyman/DoesItDie/blob/HEAD/DoesItDie/DoesItDie.toc); [ForeverLootSparkles.toc](https://github.com/svengabr/ForeverLootSparkles/blob/HEAD/ForeverLootSparkles.toc); [WorldLight.toc](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/addons/WorldLight/WorldLight.toc)
- Alternative pattern: a shared "core" add-on as a hard dependency (`## Dependencies: ForeverCore`), bundled into the release zip as a second folder via a `.pkgmeta` external + `move-folders`, because "Wago's uploads carry no dependencies at all". [Forever-verified by example] — [legacy-forever .pkgmeta](https://github.com/cjber/legacy-forever/blob/HEAD/.pkgmeta)

### Inferences
- A framework is unnecessary when the add-on has one settings table, a handful of events and a simple options page; the built-in `Settings` API, `AddonCompartmentFunc` TOC field and `C_Timer` cover what AceConfig, LibDBIcon and AceTimer were historically used for. Ace3 earns its place for profile management (AceDB), large option trees (AceConfig), inter-player comms (AceComm) and add-ons that span many client flavours.
- LibDBIcon/LibDataBroker working on Forever is inferred from their presence in Questie's and BetterBags' Forever builds; their own TOCs were not retrieved (two guessed repository paths returned 404).

### Gaps
- No explicit "Forever supported" release note was found for LibStub, CallbackHandler, LibDataBroker, LibDBIcon or LibSharedMedia individually.
- Ace3 documentation on wowace.com was not consulted in this session.
- LibUIDropDownMenu upstream fix status unknown.

---

## 5. Development tooling

### Takeaway
The in-game toolkit is the standard Mainline one (`/reload`, `/dump`, `/etrace`, `/fstack`, `/tinspect`, `/api`, `scriptErrors`, BugSack + BugGrabber), and the Blizzard debug add-ons exist on the Forever branch. Outside the game, Forever authors use VS Code + LuaLS with Ketho's WoW API annotations, `luacheck` with `std = "lua51"`, git, and a directory junction/symlink from the repository into `_forever_\Interface\AddOns`.

### Cited Findings
- `/reload` to test changes (disable unneeded add-ons to speed it up); `/console scriptErrors 1` or an error add-on like BugSack to see Lua errors; `/dump`, `/etrace` (event trace), `/fstack` (frame stack); `/tinspect` for tables; `/run` or `/script` for one-liners. [General] — [Create a WoW AddOn in 15 Minutes](https://warcraft.wiki.gg/wiki/Create_a_WoW_AddOn_in_15_Minutes)
- Same list recommended specifically for Forever: `/console scriptErrors 1`, BugGrabber and BugSack, `/fstack`, `/etrace`, `/dump <expr>`, `/tinspect <expr>`; `/console taintLog 1` writes to `Logs\taint.log` for taint hunting. [Community, Forever] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)
- `Blizzard_EventTrace` and `Blizzard_DebugTools` source files exist on the Forever branch (HTTP 200 for `Blizzard_EventTrace/Blizzard_EventTrace.lua` and `Blizzard_DebugTools/Blizzard_DebugTools.lua`, 2026-11-12). [Forever-verified] — [wow-ui-source forever branch](https://github.com/Gethe/wow-ui-source/tree/forever)
- BugSack is listed as working on Forever in a community compatibility list (data checked 2026-10-20). [Community] — [Kami Labs](https://kami-labs.fr/en/addons-wow-forever/)
- Quick client check: `/run print(GetBuildInfo()) print("project", WOW_PROJECT_ID)` (expect interface 16001 and project 18). [Community, Forever] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)
- Editor: Ketho's "WoW API" VS Code extension is a third-party add-on for LuaLS (sumneko Lua); it auto-activates when a folder contains a `.toc`, "Includes the WoW Lua 5.1 environment", loads Blizzard's API documentation and parses Warcraft Wiki docs, with completion for API, widgets, events, CVars, enums and global strings. Extension version 0.22.3 at time of research. [General] — [Ketho/vscode-wow-api](https://github.com/Ketho/vscode-wow-api)
- The extension's README badges name mainline 12.0.1, mists 5.5.3, vanilla 1.15.8 and its README/package.json do not mention Forever. — [Ketho/vscode-wow-api](https://github.com/Ketho/vscode-wow-api)
- CLI/CI equivalent: a Forever-only repository vendors the annotations and points LuaLS at them in `.luarc.json` (`"runtime.version": "Lua 5.1"`, `"workspace.library": [".types/vscode-wow-api/Annotations/Core", ".types/vscode-wow-api/Annotations/FrameXML", ...]`) plus hand-written `types/forever.lua`. [Forever-verified by example] — [legacy-forever .luarc.json](https://github.com/cjber/legacy-forever/blob/HEAD/.luarc.json)
- Linting: `luacheck` with `std = "lua51"`, `globals = { <SavedVariables>, <SLASH_ names> }` and `read_globals = { "CreateFrame", "C_Timer", "Settings", ... }` so "a typo or an accidental global fails CI instead of erroring in the client"; run in GitHub Actions with `lunarmodules/luacheck@v1`. [Forever-verified by example] — [ForeverLootSparkles .luacheckrc](https://github.com/svengabr/ForeverLootSparkles/blob/HEAD/.luacheckrc); [ForeverLootSparkles test.yml](https://github.com/svengabr/ForeverLootSparkles/blob/HEAD/.github/workflows/test.yml)
- Hot-reload practice: keep the source in a git working copy and link it into the game folder with a directory junction — `New-Item -ItemType Junction -Path "C:\Program Files (x86)\World of Warcraft\_forever_\Interface\AddOns\MyAddon" -Target "C:\dev\...\addons\MyAddon"` — then restart once so the client sees the new TOC and use `/reload` thereafter. [Community, Forever] — [TautelliniAddons README](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/README.md)
- Testing guidance specific to secrets: "A test run alone in a city proves nothing. Before a release, test in the open world, in a dungeon and during a boss encounter"; also test with only your add-on enabled, then alongside popular add-ons. [Community, Forever] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)
- Offline testing exists but is DIY: repositories carry stubbed-WoW harnesses (`tools/wowmock.lua`, `tools/forever_sim/`, busted specs) because "Nothing here can be tested outside the game client". [Forever-verified by example] — [TautelliniAddons CLAUDE.md](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/CLAUDE.md); [legacy-forever repository](https://github.com/cjber/legacy-forever)

### Inferences
- Since the Ketho extension does not advertise a Forever annotation set, its retail (mainline) annotations are the closest fit; completions may include retail-only APIs absent on Forever and miss the Forever delta. Authors compensate by checking the `forever` branch docs and writing small local type stubs.
- Slash-command tools are assumed to behave as on retail because their Blizzard add-ons ship on the Forever branch; none were run in a Forever client here.

### Gaps
- No source confirmed in-client behaviour of `/etrace`, `/fstack`, `/tinspect`, `/api` on Forever (only that the code ships).
- Whether the Ketho extension offers a selectable Forever/Camelot flavour in settings was not determined.
- No 2026 WoWUIDev Discord summary could be retrieved (Discord is not indexed).

---

## 6. Packaging and distribution

### Takeaway
The standard pipeline works for Forever: tag a release in git, let the `BigWigsMods/packager@v2` GitHub Action build the zip and upload it to CurseForge, Wago Addons and GitHub Releases, with the game version derived from the TOC's `16001`. CurseForge and Wago both have a distinct Forever flavour (`1.60.1`); WoWInterface has none, and the packager skips it for Forever. One trap: the packager recognises `_Camelot.toc` but not `_Forever.toc`.

### Cited Findings

**BigWigs packager**
- The script's flavour map includes `["forever"]="forever" ["camelot"]="forever"`; any `16???` interface maps to game type `forever`; `-g forever` (and `-g camelot`) are accepted. [Forever-verified in `release.sh`, master, read 2026-11-12] — [packager release.sh](https://github.com/BigWigsMods/packager/blob/master/release.sh)
- Multi-TOC suffix detection list is `Mainline|Classic|Vanilla|BCC|TBC|Wrath|WOTLKC|Cata|Mists|Camelot` (with `-` or `_`): `_Camelot.toc` is scanned, `_Forever.toc` is not; when the packager generates split TOCs it writes `_Camelot.toc`. [Forever-verified in `release.sh`] — [packager release.sh](https://github.com/BigWigsMods/packager/blob/master/release.sh)
- An add-on author hit this: their Forever TOC is named `_Camelot` "because BigWigs packager does not scan `_Forever.toc`". [Community, corroborating] — [ArcHUD3 AGENTS.md](https://github.com/nyyr/ArcHUD3/blob/HEAD/AGENTS.md)
- Contradiction: another Forever author recommends `_Forever.toc` for new add-ons because Blizzard said the Camelot name may change. — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md); contradicted for packaging purposes by [packager release.sh](https://github.com/BigWigsMods/packager/blob/master/release.sh)
- Build-type keywords include `version-forever` (e.g. `--@version-forever@` … blocks to keep code only in Forever builds) alongside `version-retail`, `version-classic`, etc.; the README's keyword list does not yet mention it. [Forever-verified in `release.sh`] — [packager release.sh](https://github.com/BigWigsMods/packager/blob/master/release.sh); [packager README](https://github.com/BigWigsMods/packager)
- File naming: for non-retail game types the packager appends `-<gametype>` to the file label unless the tag already contains it, so Forever files become `…-forever` (e.g. `ForeverCore-v1.7.4-forever.zip`). [Forever-verified] — [packager release.sh](https://github.com/BigWigsMods/packager/blob/master/release.sh); [legacy-forever .pkgmeta](https://github.com/cjber/legacy-forever/blob/HEAD/.pkgmeta)
- Player-facing echo of that convention: "take the one ending in `-forever` when there is one"; if there is none, the retail file already declares 16001. [Community] — [Kami Labs](https://kami-labs.fr/en/addons-wow-forever/)
- TOC upload IDs: `## X-Curse-Project-ID: 1234`, `## X-WoWI-ID: 5678`, `## X-Wago-ID: he54k6bL` (or flags `-p`, `-w`, `-a`). Keyword substitution: `@project-version@`, `@project-date-iso@`, `@file-abbreviated-hash@`, `@localization(...)@`, etc. [General] — [packager README](https://github.com/BigWigsMods/packager)
- `.pkgmeta` directives: `package-as`, `externals`, `ignore`, `plain-copy`, `move-folders`, `manual-changelog`, `enable-nolib-creation`, `enable-toc-creation`, `required-dependencies`, `optional-dependencies`, `embedded-libraries`, `tools-used`, `license-output`, `changelog-title`, plus `wowi-*` options. [General] — [packager README](https://github.com/BigWigsMods/packager)
- Credentials: README documents `CF_API_TOKEN`, `WOWI_API_TOKEN`, `WAGO_API_TOKEN`, `GH_API_TOKEN`; real Forever workflows (and the long-standing convention) still use `CF_API_KEY` and `GITHUB_OAUTH`. — [packager README](https://github.com/BigWigsMods/packager); [DoesItDie release.yml](https://github.com/swirllyman/DoesItDie/blob/HEAD/.github/workflows/release.yml)

**Release workflow as used by a Forever-only add-on** — [DoesItDie release.yml](https://github.com/swirllyman/DoesItDie/blob/HEAD/.github/workflows/release.yml)
```yaml
name: Package and release
on:
  push:
    tags: ["v*"]
jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    env:
      CF_API_KEY: ${{ secrets.CF_API_KEY }}
      GITHUB_OAUTH: ${{ secrets.GITHUB_TOKEN }}
    steps:
      - uses: actions/checkout@v5
        with:
          fetch-depth: 0
      - uses: BigWigsMods/packager@v2
```
Plain English: when a tag like `v0.2.1` is pushed, check out the full history (needed for the changelog), then run the packager, which zips the add-on, substitutes the version into the TOC, creates a GitHub Release and uploads to CurseForge using the project ID from the TOC and the API key stored as a repository secret. Add `WAGO_API_TOKEN` to also publish to Wago ([BetterBags release.yml](https://github.com/Cidan/BetterBags/blob/HEAD/.github/workflows/release.yml)).

Matching `.pkgmeta` for an add-on kept in a subfolder — [DoesItDie .pkgmeta](https://github.com/swirllyman/DoesItDie/blob/HEAD/.pkgmeta):
```yaml
package-as: DoesItDie
manual-changelog:
    filename: CHANGELOG.md
    markup-type: markdown
move-folders:
    DoesItDie/DoesItDie: DoesItDie
ignore:
    - tools
    - README.md
```
Plain English: name the zip's top folder `DoesItDie`, use the hand-written changelog instead of the commit list, lift the add-on subfolder to the zip root, and leave development files out.

**CurseForge**
- CurseForge has a Forever game-version type: the packager uses `gameVersionTypeID` `88568` for `forever` (others: classic 67408, bcc 73246, wrath 73713, cata 77522, mists 79434, titan 81212, retail 517) and validates the version name against CurseForge's `/api/game/wow/versions`. [Forever-verified in `release.sh`] — [packager release.sh](https://github.com/BigWigsMods/packager/blob/master/release.sh)
- A Forever-only project page shows game version `1.60.1` under the "Forever" flavour (ThiccUI, updated 2026-11-11). [Forever-verified by example] — [ThiccUI on CurseForge](https://www.curseforge.com/wow/addons/thiccui)
- "Tag each file with the Forever game version on CurseForge and on Wago, or users will not find it in the Forever filter." [Community] — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)
- The project description is not updatable via the upload API; one author pastes the README in by hand after changes. [Community] — [ForeverLootSparkles AGENTS.md](https://github.com/svengabr/ForeverLootSparkles/blob/HEAD/AGENTS.md)

**Wago Addons**
- Wago's public game-data API (queried 2026-11-12) lists `"forever": ["1.60.1"]` under `patches`, `"supported_forever_patches": "1.60.1"` under `live_patches`, and TOC suffixes for forever as `["-Camelot","_Camelot","-Forever","_Forever"]`. [Forever-verified] — [Wago game data API](https://addons.wago.io/api/data/game)
- The packager sends Wago a `supported_forever_patches` property derived from the TOC. [Forever-verified in `release.sh`] — [packager release.sh](https://github.com/BigWigsMods/packager/blob/master/release.sh)
- Wago uploads carry no dependency metadata (one author's reason for bundling a core library in the zip). [Community] — [legacy-forever .pkgmeta](https://github.com/cjber/legacy-forever/blob/HEAD/.pkgmeta)

**WoWInterface**
- WoWInterface's compatibility API (queried 2026-11-12) lists only the games `Retail`, `Classic`, `TBC-Classic`, `WOTLK-Classic`, `Cata-Classic` — no Forever entry. [Forever-verified absence] — [WoWInterface compatible.json](https://api.wowinterface.com/addons/compatible.json)
- The packager prints `No WoWInterface game type match for "forever" ... not supported, ignoring` and skips that version (a code comment nearby reads "XXX wowi dead, yo"). [Forever-verified in `release.sh`] — [packager release.sh](https://github.com/BigWigsMods/packager/blob/master/release.sh)

**GitHub Releases**
- The packager creates the GitHub Release itself when given a token with `contents: write`; "nolib" packages are only attached to GitHub releases. — [packager README](https://github.com/BigWigsMods/packager); [DoesItDie release.yml](https://github.com/swirllyman/DoesItDie/blob/HEAD/.github/workflows/release.yml)

**Add-on managers**
- CurseForge app: supports Forever as a selectable installation ("select your Forever live installation"; "puts the files in the right place once Forever is selected as the game"). — [CurseForge blog, 2026-10-25](https://blog.curseforge.com/wow-forever-migrate-addons-beta-to-live/); [Kami Labs](https://kami-labs.fr/en/addons-wow-forever/)
- Manual install remains: extract the add-on folder into the Forever `Interface/AddOns` folder. — [ThiccUI on CurseForge](https://www.curseforge.com/wow/addons/thiccui)

### Inferences
- For a multi-flavour add-on released through the BigWigs packager today, use either one TOC with a comma list including `16001`, or a `_Camelot.toc`; a `_Forever.toc` loads in the client and is recognised by Wago, but is invisible to the packager's TOC scan as of 2026-11-12. For a Forever-only add-on, the unsuffixed TOC sidesteps the question.
- Forever-compatible builds are identified by: CurseForge — game version `1.60.1` in the WoW Forever version type (id 88568); Wago — `supported_forever_patches: ["1.60.1"]`; GitHub — nothing structured, only the `-forever` file-name suffix and the TOC; WoWInterface — not identifiable at all.
- WoWInterface is effectively not a distribution channel for Forever add-ons at this time.

### Gaps
- WowUp and the Wago app: no source found confirming or denying Forever support (two searches returned nothing relevant; WowUp's GitHub releases page yielded no mention). Since Wago's API exposes a `forever` flavour, Wago-app support is plausible but unverified.
- CurseForge's author-facing documentation on tagging Forever files was not found; the upload UI's exact label for the flavour is inferred from project pages ("Forever", "WoW Forever" filter).
- Whether CurseForge moderation treats Forever uploads differently (review time, restrictions) is unknown.
- The packager wiki's full example workflow was not read; snippets above come from real repositories.

---

## 7. Real examples: small open-source add-ons that declare Forever support

### Takeaway
Five small, open-source, actively maintained add-ons with `## Interface: 16001` were inspected (cloned 2026-11-12). They converge on the same template: an unsuffixed TOC, namespace table, no framework, `.pkgmeta` + `BigWigsMods/packager@v2` on tag push, `luacheck` in CI, CurseForge project ID in the TOC.

### Cited Findings

**1. ForeverLootSparkles (svengabr; GPL-3.0; last commit 2026-11-11) — the minimal single-file template** — [repository](https://github.com/svengabr/ForeverLootSparkles)
- Layout: `ForeverLootSparkles.toc`, `ForeverLootSparkles.lua`, `Icon.tga`, `.pkgmeta`, `.luacheckrc`, `CHANGELOG.md`, `README.md`, `LICENSE`, `.github/workflows/{release,test}.yml`, `media/` (store gallery, excluded from zip).
- TOC:
```
## Interface: 16001
## Title: Forever Loot Sparkles
## Notes: Always shows the sparkle effect on lootable quest items. ...
## Author: Ashtar Senior
#@debug@
## Version: dev
#@end-debug@
#@non-debug@
# ## Version: @project-version@
#@end-non-debug@
## IconTexture: Interface\AddOns\ForeverLootSparkles\Icon
## SavedVariables: ForeverLootSparklesDB
## X-License: GPL-3.0-or-later
## X-Curse-Project-ID: 1729017

ForeverLootSparkles.lua
```
Plain English: the `#@debug@` block makes the version read "dev" when run straight from the repository, while the packager strips that block and un-comments the real version line in release zips. Uses `Settings.RegisterVerticalLayoutCategory` for options, defers CVar writes during combat until `PLAYER_REGEN_ENABLED`. Release = tests job (luacheck) then packager.

**2. DoesItDie (swirllyman; MIT; last commit 2026-11-09) — multi-file, localised, add-on in a subfolder** — [repository](https://github.com/swirllyman/DoesItDie)
- Layout: `DoesItDie/` (the add-on: `DoesItDie.toc`, `Locale.lua` + 7 `Locale_xxXX.lua`, `DoesItDie.lua`, `Display.lua`, `Frames.lua`, `Options.lua`, `Nameplates.lua`, `Focus.lua`, `Execute.lua`, `API.lua`), `WillItDieProbe/` (a separate diagnostic add-on, Interface 16001, "checks which combat data addons can read in WoW Forever"), `tools/` (Python tests and scrapers), `.pkgmeta`, `.github/workflows/release.yml`.
- TOC header: `## Interface: 16001`, `Title`, `Notes` + `Notes-deDE`, `IconTexture`, `Author`, `Version: @project-version@`, `SavedVariables: DoesItDieDB`, `X-Curse-Project-ID: 1709161`; file list puts locale files first, then core, then UI modules. — [DoesItDie.toc](https://github.com/swirllyman/DoesItDie/blob/HEAD/DoesItDie/DoesItDie.toc)
- Listed as a Forever add-on in a community list (entry dated 2026-10-28). — [Kami Labs](https://kami-labs.fr/en/addons-wow-forever/)

**3. LegacyForever (cjber; GPL-3.0; last commit 2026-11-12) — dependency on a shared core, compartment button, CurseForge + Wago** — [repository](https://github.com/cjber/legacy-forever)
- Layout: `LegacyForever.toc`, `Core/`, `UI/` (Lua + XML), `Locales/` (10 locale files), `types/` (LuaLS stubs incl. `forever.lua`), `tests/`, `tools/forever_sim/` (stub add-ons for a simulator), `.luarc.json`, `.pkgmeta`, several CI workflows.
- TOC header: `## Interface: 16001`, `Category: Map`, `X-Curse-Project-ID: 1705879`, `X-Wago-ID: b6mzA7KP`, `IconTexture`, `Dependencies: ForeverCore`, `OptionalDeps: ShortestPathForever, QuestieDB, Questie, AtlasLootClassic`, `SavedVariables: LegacyForeverDB`, `LoadSavedVariablesFirst: 1`, `AddonCompartmentFunc: LegacyForever_OnAddonCompartmentClick` (a global assigned in `Core/Core.lua`: `LegacyForever_OnAddonCompartmentClick = OpenMenu`). — [LegacyForever.toc](https://github.com/cjber/legacy-forever/blob/HEAD/LegacyForever.toc)
- `.pkgmeta` uses `optional-dependencies` (CurseForge slugs), an `externals` zip URL and `move-folders` to ship `ForeverCore` as a second top-level folder. — [.pkgmeta](https://github.com/cjber/legacy-forever/blob/HEAD/.pkgmeta)

**4. TautelliniAddons (Tautellini) — multi-add-on monorepo with a template and a written Forever guide** — [repository](https://github.com/Tautellini/TautelliniAddons)
- Layout: `addons/<Name>/` (WorldLight, ClearMap, Breakfast), `templates/ForeverAddonTemplate/` (2 files), `docs/forever-addon-development.md` ("Last checked against client 1.60.1 build 70790 (launch day, 2026-11-04)"), `assets/<Name>/` (CurseForge description + changelog), `tests/`, `.luacheckrc`.
- WorldLight TOC: `## Interface: 16001`, `Title`, `Notes`, `Author`, `IconTexture`, `Version: 1.0.2`, `SavedVariables: WorldLightDB`, then `Compute.lua`, `Settings.lua`, `WorldLight.lua`. — [WorldLight.toc](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/addons/WorldLight/WorldLight.toc)
- No `.pkgmeta` or release workflow found at the guessed paths (404), so releases may be manual; licence not confirmed.

**5. Larger references that added a Forever TOC to an existing multi-flavour add-on**
- BetterBags (Cidan; MIT): `BetterBags_Camelot.toc` with `## Interface: 16001`, `X-Curse-Project-ID`, `X-Wago-ID`, libraries first, and a Camelot-only `core\forever.lua` listed only in this TOC to set `addon.isForever`. — [BetterBags_Camelot.toc](https://github.com/Cidan/BetterBags/blob/HEAD/BetterBags_Camelot.toc)
- ArcHUD3 (nyyr): one TOC per flavour (`ArcHUD3.toc` 120100, `_Vanilla` 11509, `_TBC` 20506, `_Wrath` 30405, `_Cata` 40402, `_MoP` 50504, `_Camelot` 16001), shared Lua, flavour-specific XML file lists (`Rings/Rings-forever.xml`). — [ArcHUD3 AGENTS.md](https://github.com/nyyr/ArcHUD3/blob/HEAD/AGENTS.md); [ArcHUD3_Camelot.toc](https://github.com/nyyr/ArcHUD3/blob/HEAD/ArcHUD3_Camelot.toc)
- Questie: ships both `Questie_Forever.toc` and `Questie_Camelot.toc` for game type `forever`. — [Questie AGENTS.md](https://github.com/Questie/Questie/blob/HEAD/AGENTS.md); [Questie_Forever.toc](https://github.com/Questie/Questie/blob/HEAD/Questie_Forever.toc)

**Other Forever-only repositories surfaced by GitHub search (not inspected)**: `neroxrw/foreverauras` (GPL-2.0, 21 stars), `markgravity/ImprovedForever` (MIT), `Krelock/NodeSight`, `CryptoLogiq/SkillsAlreadyKnown` (MIT), `krilar1/ForeverTools` (MIT), `Atraeau/WoW-Addons` ("WoW: Forever addon dev environment … generated API reference"). — [GitHub repository search "wow forever addon"](https://github.com/search?q=wow+forever+addon&type=repositories)

### Inferences
- A defensible "house template" for a first Forever add-on: `MyAddon/MyAddon.toc` (16001, Title, Notes, Author, `Version: @project-version@`, IconTexture, SavedVariables, `X-Curse-Project-ID`, optionally `X-Wago-ID`), `MyAddon.lua`, `.pkgmeta` (`package-as`, `ignore`, optional `manual-changelog`), `.luacheckrc` (`std = "lua51"`), `.github/workflows/release.yml` (packager on tag), `CHANGELOG.md`, `LICENSE`, `README.md`.
- Many of these repositories contain AI-agent instruction files (`AGENTS.md`, `CLAUDE.md`); several of the "Community" facts above come from such files, which are maintainers' working notes rather than reviewed documentation.

### Gaps
- `neroxrw/foreverauras` and `markgravity/ImprovedForever` were cloned but their output was cut off before inspection; their TOCs are unrecorded.
- CurseForge download counts / popularity of these examples not collected.
- Quartz's default-branch TOC lists `11508,20505,50503,110207` with no 16001, although a "Forever support" pull request (#110) is cited by the Tautellini guide; its merge status was not checked. — [Quartz.toc](https://github.com/Nevcairiel/Quartz/blob/master/Quartz.toc)

---

## 8. Common beginner failures

### Takeaway
The failures are the classic ones plus three Forever-specific traps: putting the add-on in the wrong client folder (or using a Classic add-on), assuming `WOW_PROJECT_MAINLINE` means "modern client", and doing arithmetic on secret values that only appear in combat or instances.

### Cited Findings
- **Wrong client folder / wrong flavour.** Add-ons in the Retail or Classic `Interface` folder do not show up in Forever; Classic Era add-ons do not load on Forever at all. — [Kami Labs](https://kami-labs.fr/en/addons-wow-forever/); [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)
- **Leftover beta add-ons.** At beta start, carried-over add-ons caused Lua errors at login; advice was to uninstall/disable old beta add-ons (two reader comments disputed that add-ons carry over at all). — [Warcraft Tavern, 2026-09-17](https://www.warcrafttavern.com/forever/news/wow-forever-loads-your-retail-addons-keybinds/)
- **Wrong Interface number → "out of date".** If no listed interface value matches, the add-on is flagged out of date; users patch third-party TOCs by adding `16001` to the first line (FlareUI's page instructs exactly this for a bundled borders add-on). — [TOC format](https://warcraft.wiki.gg/wiki/TOC_format); [FlareUI on CurseForge](https://www.curseforge.com/wow/addons/flareui)
- **Shipping only `_Mainline.toc` without 16001** loads on Forever but is marked out of date. — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)
- **Folder/TOC name mismatch or double-nested folder** → add-on not detected. — [Create a WoW AddOn in 15 Minutes](https://warcraft.wiki.gg/wiki/Create_a_WoW_AddOn_in_15_Minutes); [allthings.how](https://allthings.how/world-of-warcraft-forever-how-to-install-addons-without-the-curseforge-app/)
- **Load-order mistakes.** Files run strictly in TOC order; a file that sets a flag must be listed before the file that reads it (BetterBags: `core/forever.lua` "must load after `core/boot.lua` … and before `core/constants.lua`"); libraries go first; a compatibility shim "must precede embedded libraries" (Questie). — [AddOn loading process](https://warcraft.wiki.gg/wiki/AddOn_loading_process); [BetterBags camelot-forever.md](https://github.com/Cidan/BetterBags/blob/HEAD/.claude/rules/camelot-forever.md); [Questie_Forever.toc](https://github.com/Questie/Questie/blob/HEAD/Questie_Forever.toc)
- **Saved variables read too early.** They are not available while files execute (unless `LoadSavedVariablesFirst: 1`); `ADDON_LOADED` for your own add-on name is the earliest safe point; forgetting to check the name argument runs your init for every add-on that loads. — [AddOn loading process](https://warcraft.wiki.gg/wiki/AddOn_loading_process); [Create a WoW AddOn in 15 Minutes](https://warcraft.wiki.gg/wiki/Create_a_WoW_AddOn_in_15_Minutes)
- **Saved variables lost.** Not written on crash or Alt-F4. — [AddOn loading process](https://warcraft.wiki.gg/wiki/AddOn_loading_process)
- **Global leakage.** Any un-`local` variable is shared with every add-on and Blizzard's UI; fix with the namespace table and a `luacheck` whitelist. — [Create a WoW AddOn in 15 Minutes](https://warcraft.wiki.gg/wiki/Create_a_WoW_AddOn_in_15_Minutes); [ForeverLootSparkles .luacheckrc](https://github.com/svengabr/ForeverLootSparkles/blob/HEAD/.luacheckrc)
- **Taint from touching Blizzard frames.** Never overwrite Blizzard functions or frame methods; use `hooksecurefunc`/`HookScript`; protected actions fail in combat, so check `InCombatLockdown()` and defer to `PLAYER_REGEN_ENABLED`; "If a Blizzard panel starts throwing errors after your addon touches it, suspect taint" (`/console taintLog 1`). Real Forever case: a settings predicate tainted Blizzard's options search. — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md); [ForeverLootSparkles AGENTS.md](https://github.com/svengabr/ForeverLootSparkles/blob/HEAD/AGENTS.md)
- **Flavour detection gone wrong.** Build 70170 changed `WOW_PROJECT_ID` to 18; add-ons testing `== WOW_PROJECT_MAINLINE` took Classic code paths and crashed (BetterBags: "table index is nil" at load). `WOW_PROJECT_ID == WOW_PROJECT_CAMELOT` is `nil == nil` → true when neither global exists (test environments, older clients). — [BetterBags camelot-forever.md](https://github.com/Cidan/BetterBags/blob/HEAD/.claude/rules/camelot-forever.md)
- **Secret values.** Arithmetic or comparison on a secret raises an error; it only shows up in combat/encounters/PvP, so testing alone in a city misses it. Pattern: test `issecretvalue(x)` before maths, pass secrets straight to widgets such as `StatusBar:SetValue`. — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)
- **Building on the combat log.** `COMBAT_LOG_EVENT_UNFILTERED` is restricted for add-ons on Forever. — [TautelliniAddons CLAUDE.md](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/CLAUDE.md); [Kami Labs](https://kami-labs.fr/en/addons-wow-forever/)
- **Assuming Classic data or retail layout.** Static Classic item/quest tables are incomplete on Forever; retail bank assumptions are wrong (9 character tabs, no Warband bank). — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md); [BetterBags camelot-forever.md](https://github.com/Cidan/BetterBags/blob/HEAD/.claude/rules/camelot-forever.md)
- **Anchoring to a frame that does not exist on Forever.** Several unit frames were reskinned and children moved; check existence and use `/fstack`. — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md)
- **Publishing without the Forever tag** (file invisible in the Forever filter), or relying on `_Forever.toc` with the BigWigs packager (not scanned). — [TautelliniAddons Forever guide](https://github.com/Tautellini/TautelliniAddons/blob/HEAD/docs/forever-addon-development.md); [packager release.sh](https://github.com/BigWigsMods/packager/blob/master/release.sh)
- **Changing persistent CVars.** CVars persist in `WTF/Config.wtf`; disabling the add-on does not undo them. — [ForeverLootSparkles AGENTS.md](https://github.com/svengabr/ForeverLootSparkles/blob/HEAD/AGENTS.md)
- **Using unverified libraries.** A bundled library may mis-detect Forever (LibUIDropDownMenu case). — [BetterBags camelot-forever.md](https://github.com/Cidan/BetterBags/blob/HEAD/.claude/rules/camelot-forever.md)

### Inferences
- A pre-release checklist follows directly: (1) `/dump select(4, GetBuildInfo())` equals the TOC number; (2) folder name = TOC name, single nesting; (3) `luacheck` clean with an explicit globals whitelist; (4) saved variables touched only from `ADDON_LOADED`; (5) no direct writes to Blizzard frames/functions, combat-guard anything protected; (6) play-test in open world, dungeon and boss fight for secrets; (7) file tagged Forever `1.60.1` on each host.
- The next Forever patch will change the interface number, so expect a wave of "out of date" flags; a tag-and-release pipeline makes the bump a one-line change.

### Gaps
- No first-hand 2026 WoWUIDev Discord guidance or Blizzard forum thread on beginner pitfalls for Forever was retrievable; pitfalls are compiled from the wiki and from add-on maintainers' own notes.
- Frequency of each failure (which is most common in practice) is unknown — no data source.
- Secret-value and taint details are deliberately shallow here; they belong to the secure-execution researcher.
