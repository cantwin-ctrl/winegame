# winegame

Per-game Wine prefix manager. One prefix per game, zero bloat.

Plain `wine` + `winetricks`, organized. No Steam, no Lutris, no GUI daemons — a single bash script that keeps every game in its own isolated prefix with persistent env vars, DLL overrides, per-run logs, and a shell escape hatch.

## Why

Cracked game folders, itch.io games, old Windows apps — they all want their own Wine environment, and sharing one prefix is how everything breaks. `winegame` gives each game its own `~/.games/<name>/` and makes the common operations one command each.

## Install

```bash
# winegame needs wine + winetricks (both standard packages)
# Debian/Ubuntu:  sudo apt install wine winetricks

# optional but recommended: gamemode for the -g flag
# Debian/Ubuntu:  sudo apt install gamemode

cp winegame ~/.local/bin/          # or anywhere on your PATH
winegame doctor                    # verify everything's in place
```

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
winegame doctor                      check tooling; doctor <name> scans a game
```

## Typical workflow

```bash
winegame new nekopara                     # creates ~/.games/nekopara/
winegame run nekopara /path/to/Game.exe   # first run: prompted to copy the folder in
winegame tricks nekopara vcrun2019 -q     # install a runtime into that prefix
winegame override nekopara dinput8=n,b    # force a cracked DLL to load
winegame run -g nekopara                  # from now on: just this
```

When `run` is given an exe outside the prefix it asks once whether to copy the
game folder in — after that the exe is auto-detected and all you type is
`winegame run <name>`.

## Layout

```
~/.games/<name>/
├── prefix/    # the WINEPREFIX itself
├── game/      # your game files (installed here once)
├── env        # persistent env vars, sourced before every run
└── logs/      # one timestamped log per run
```

Set `WINE_GAMES_DIR` to relocate everything (e.g. another drive).

## Shell completion

```bash
winegame --install-completion   # adds tab completion for bash or zsh
```

## Requirements

- bash 4+
- wine (any modern version)
- winetricks (optional but very useful)
- rsync (optional — installs fall back to `cp`)
- gamemode (optional — powers the `-g` flag)

## License

MIT — do whatever, just don't blame me if a prefix eats your save files.
