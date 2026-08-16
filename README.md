<div align="center">

![winegame](logo.png)

# 🍷 winegame

**Per-game Wine prefix manager. One prefix per game, zero bloat.**

Plain `wine` + `winetricks`, organized. No Steam, no Lutris, no GUI daemons — a single bash script that keeps every game in its own isolated prefix with persistent env vars, DLL overrides, per-run logs, and a shell escape hatch.

[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
![Bash](https://img.shields.io/badge/language-bash-4EAA25.svg?logo=gnubash&logoColor=white)
![Platform](https://img.shields.io/badge/platform-linux-lightgrey.svg)
![Version](https://img.shields.io/badge/version-1.3.0-blue.svg)

</div>

## Why

Cracked game folders, itch.io games, old Windows apps — they all want their own Wine environment, and sharing one prefix is how everything breaks. `winegame` gives each game its own `~/.games/<name>/` and makes the common operations one command each.

![demo](winegame.png)

## Install

```bash
# winegame needs wine + winetricks (both standard packages)
# Debian/Ubuntu:  sudo apt install wine winetricks

# optional but recommended: gamemode for the -g flag
# Debian/Ubuntu:  sudo apt install gamemode

cp winegame ~/.local/bin/          # or anywhere on your PATH
winegame doctor [name]              verify tooling; with a name: scan a game for missing runtimes
winegame doctor fix                 one-shot: install WineHQ wine + fix video decode (see below)
```

## Quickstart

```bash
winegame new nekopara                     # create a prefix for a game
winegame run nekopara /path/to/Game.exe   # first run: prompted to copy the folder in
winegame tricks nekopara vcrun2019 -q     # install a runtime into that prefix
winegame override nekopara dinput8=n,b    # force a cracked DLL to load
winegame run -g nekopara                  # from now on: just this
```

## Why not just use Bottles?

[Bottles](https://usebottles.com/) is the same idea with a GUI — isolated per-game prefixes, runner switching, dependency installers. If you want a clickable interface and per-game Proton/DXVK runners, use Bottles. It's good at what it does.

`winegame` is for the other half of the room:

- **one bash script** — no Flatpak, no GTK app, no daemon. Works over SSH, in tmux, on a headless box
- **scriptable** — every operation is a command, so it drops into your own tooling
- **zero ceremony** — `winegame new` + `run` and you're in-game; no environment picker, no runner setup
- **transparent** — prefixes are plain folders under `~/.games/`, plain `wine` + `winetricks` underneath. Nothing hidden
- **`doctor fix`** — auto-installs WineHQ staging + the right VA-API driver, fixing the VN movie deadlock class of bug out of the box

GUI people get Bottles. Terminal people get `winegame`. Both fine.

## DXVK / vkd3d-proton (Direct3D → Vulkan)

`winegame dxvk <name>` installs DXVK (d3d8/d3d9/d3d10/d3d11 → Vulkan) and
vkd3d-proton (d3d12 → Vulkan) into a prefix — wine's builtin d3d gets replaced
by the Vulkan translators, which is a large FPS win on most games.

```bash
winegame dxvk <name>          # install both (latest releases, cached in ~/.cache/winegame)
winegame dxvk <name> --dx11   # DXVK only, skip vkd3d-proton (no DX12 games)
winegame dxvk <name> --info   # what's installed
winegame dxvk <name> --remove # delete the DXVK DLLs, restore wine builtins
```

Needs Vulkan drivers (`doctor` now checks). The DLLs land in the prefix's
system32/syswow64 and the right `WINEDLLOVERRIDES` are merged into the game's
`env` file automatically — nothing else to configure.

## Goldberg emulator (run Steam games without Steam)

`winegame steamemu <name>` replaces a game's `steam_api.dll`/`steam_api64.dll`
with the [Goldberg emulator](https://gitlab.com/Mr_Goldberg/goldberg_emulator)
(prebuilt releases via the [gbe_fork](https://github.com/Detanup01/gbe_fork)).

```bash
winegame steamemu <name> [--appid N]  # install; --appid or it prompts / reuses existing steam_appid.txt
winegame steamemu <name> --info       # what's installed
winegame steamemu <name> --remove     # restore the original DLLs
```

- Scans the game's exes (objdump) and only touches DLLs they actually import —
  redist copies in subfolders are left alone. Packed exes fall back to a
  "replace all found" prompt.
- Originals are backed up to `*.bak-goldberg`; `--remove` restores them and
  deletes any `steam_appid.txt` it created.
- If the game imports `steamclient64.dll` but doesn't ship it, the emu's copy
  is dropped next to the exe automatically.
- Saves land in `~/.local/share/GSE Saves/`; per-game tweaks go in a
  `steam_settings/` folder next to the DLL.

## Troubleshooting

**Game hangs on a black/white window when playing a movie** (VN OP videos on Debian/Ubuntu):

Debian's wine build can deadlock on DirectShow video (`winegstreamer`). Fix once, every prefix benefits:

```bash
winegame doctor        # detects bad wine build + missing VA-API driver
winegame doctor fix    # installs WineHQ staging + right VA-API driver (iHD/radeonsi)
```

Then re-create any game that already hung (`winegame new` + `install`). After switching wine versions, run each game once — the prefix auto-upgrades silently on first launch.

## Usage

```
winegame new <name> [--win32]        create a prefix (--win32 for 32-bit only games)
winegame install <name> <folder|exe> copy a game folder into the prefix once
winegame run <name> [exe] [args...]  run a game; exe auto-detected in <name>/game/
                                     flags: -g/--gamemode, --no-copy, -v/--verbose
winegame cfg <name>                  open winecfg (DLL overrides, display, etc.)
winegame setver <name> <ver>         set Windows version (win7|win10|win11|winxp)
winegame tricks <name> [pkgs...]     winetricks; no packages = GUI (e.g. vcrun2019 -q)
winegame override <name> [dll=mode]  set DLL overrides (dinput8=n,b); no args = show
winegame env <name> [KEY=***]         persistent env for the prefix; no args = show
winegame wine <name> [args...]       raw wine passthrough in the prefix
winegame shell <name>                shell with prefix env loaded (escape hatch)
winegame log <name> [-f]             show/tail latest run log
winegame list                        list prefixes
winegame remove <name> [-y]          delete a prefix (asks unless -y)
winegame doctor                      check wine/winetricks tooling + wine build + GPU video driver
winegame doctor fix [name]           install WineHQ wine + right VA-API driver (fixes movie deadlock)
winegame doctor <name>               scan a game for missing runtimes, offer winetricks fixes
winegame dxvk <name> [--dx11]        install DXVK + vkd3d-proton (Vulkan); --remove/--info
winegame steamemu <name> [--appid N] replace steam_api*.dll with the Goldberg emulator; --remove
```

## Layout

```
~/.games/<name>/
├── prefix/    # the WINEPREFIX itself
├── game/      # your game files (installed here once)
├── env        # persistent env vars, exported to the game process on every run
└── logs/      # one timestamped log per run
```

The `env` file is sourced and **exported** into the wine process before every
launch — `LIBVA_DRIVER_NAME`, `WINEDLLOVERRIDES`, anything you put there
actually reaches the game. `winegame new` pre-fills it with the right VA-API
driver for your GPU.

Set `WINE_GAMES_DIR` to relocate everything (e.g. another drive).

## Shell completion

```bash
winegame --install-completion   # adds tab completion for bash or zsh
```

## Requirements

- bash 4+ (every Linux distro ships 5)
- wine (any modern version; WineHQ staging recommended)
- winetricks
- binutils (for objdump — used by `doctor <name>` import scanning)
- curl (downloads for `dxvk` / `steamemu`)
- p7zip-full (unpacks the Goldberg package for `steamemu`)
- zstd (unpacks vkd3d-proton for `dxvk`)
- vulkan drivers (mesa-vulkan-drivers / nvidia-driver — required by DXVK)
- pciutils (for lspci — used by `doctor`/`doctor fix` GPU detection)
- rsync (optional — installs fall back to `cp`)
- gamemode (optional — powers the `-g` flag)

## License

MIT — do whatever, just don't blame me if a prefix eats your save files.
