# AGENTS.md

Guidance for agents working in this repository. Chromium Client is a browser Minecraft 26.2 client (Eaglercraft / TeaVM) plus a single-file launcher. There is no package manager, bundler, linter, or test runner here.

## Run

Open the HTML files in a browser. Nothing in this repo builds them.

- Launcher: `Chromium Launcher.html` (loads `presets-info.js`).
- Standard engine: `Chromium-Client-vanilla.html`.
- Shaders engine (Sodium + Iris): `Chromium-Client-shaders.html`.

`presets-info.js` sets `window.CHROMIUM_BUILT` (`vanilla`, `performance`, `shaders` timestamps) and `window.CHROMIUM_FAILED`. A missing timestamp makes `isBuilt` refuse that engine. There is no `Chromium-Client-performance.html` in this tree. `buildFor` returns `shaders` only when the instance mods include `iris`; otherwise `vanilla`. The old Sodium-only path is Chromium video settings on the Standard engine (`chromium_sodium` in the launch hash), not the Performance file.

`chromium-packs.js` (`window.__chromiumPackData`, six built-in resource packs) is a payload blob. Do not rewrite it.

Live site API: `location` host matching `chromiumclient.com` uses `/api`; any other host uses `https://chromiumclient.com/api`. `localStorage["chromium.api.dev"]` overrides that base. `ONLINE_SITE` is true only for `http:` / `https:`.

## Do not load whole client files

`*.html` is Git LFS (`filter=lfs diff=lfs merge=lfs -text`).

| File | About |
| --- | --- |
| `Chromium Launcher.html` | 2.3 MB. Editable UI and logic. |
| `Chromium-Client-vanilla.html` | ~234 MB. Inline brotli WASM, assets. |
| `Chromium-Client-shaders.html` | ~186 MB. Same, plus sounds. |
| `chromium-packs.js` | ~17 MB, one long line. |

Search with ripgrep. Never read a client HTML or `chromium-packs.js` end to end, and never reformat or rewrite those files. A client change belongs in the boot block (`<!-- chromium-boot-v1 -->` … `<!-- chromium-boot-end -->`, lines 1–184, shared by both client pages) or in a located script after the `application/octet-stream` payloads. Leave `eag-inline-*` script bodies untouched.

## Layout

Everything the player sees is inline in `Chromium Launcher.html`: CSS, then scripts marked by `/* ===== Chromium Launcher: … ===== */`.

- Core (instances, mods, launch, streak, servers) starts near the `KEY` / `launchUrl` block.
- Accounts, sync, presence, friends.
- Shell and pages (home, instances, mods, servers, settings).
- Social (notifications, messages, parties, wardrobe, clips, store / VIP).

`presets-info.js` is the only external script the launcher loads.

Client pages: boot screen, then inline payloads (`eag-inline-decoder`, `eag-inline-wasm-br`, `eag-inline-mesh-wasm-br`, `eag-inline-server-wasm-br`, `eag-inline-assets`, and on shaders `eag-inline-sounds`), then the TeaVM Wasm-GC shell. Boot progress watches those node ids and `window.__log`, `window.__eaglerBoot`, `window.__chromiumLoad`.

## Launch contract

`launchUrl` opens `Chromium-Client-<build>.html` with a hash the boot screen and the game both read. Keep these names:

`mods`, `launcher`, `perf` (`chromium.perf.v1`, `"1"` / `"0"`; default on for Chrome OS, `deviceMemory <= 4`, or `hardwareConcurrency <= 2`), `join`, `joinName`, `c1`, `c2`, `cx`, `cy`, `iname`.

`mods` is the optional mod id list. `sodium` and `iris` are not passed as ids; they choose the engine. `chromium_fps` is always added. `chromium_fps_low` is added on low-end devices. `chromiumsocial` is added unless in-game chat is off (`SOCIAL_MOD.soon` is toggled with `/*@SOON@*/`).

`CATALOG` in the launcher is the mod list (`id`, `name`, `soon`). `available()` is `BY_ID[id] && !soon`. `LOCKED` entries stay in the catalog but are not selectable yet.

## Storage

Persist with `ls` / `load` / `save` (they swallow quota errors). These keys are synced when an account is signed in (`SYNC_KEYS`); renaming one desyncs existing browsers:

`chromium.instances.v1`, `chromium.selected.v1`, `chromium.servers.v1`, `chromium.perf.v1`, `chromium.background.v2`, `chromium.background.image.v1`.

Also used locally: `chromium.tab.v1`, `chromium.streak.v1`, `chromium.played.v1`, `chromium.account.v1`, `chromium.account.sync.v1`, `chromium.topServers`, `chromium.api.dev`.

Account calls go through `api(method, path, body)` to `API`. Password stretching happens in the browser (`stretch`); only the stretched value is sent. Ranks: `owner`, `staff`, `youtuber`, `tiktok`, `twitch`, `partner`, `vip`, `tester`. `isStaff` is owner or staff. `isOwner` is owner.

## UI conventions

Vanilla DOM and ES5 (`var`, `function`). No modules, no JSX. Escape untrusted strings with `esc` before inserting HTML.

Appearance: Glass (frosted panels, Manrope) is the default look; Classic turns that block off (Appearance > Style). Match nearby CSS section comments (`/* ---------- … ---------- */`) instead of adding a new framework.

The launcher toasts with `toast(msg, kind)` (`kind === "err"` stays longer). Boot tips and status strings are player-facing; keep that voice.
