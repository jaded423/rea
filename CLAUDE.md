# rea — fork/pin of morluto/rea (Reverse Engineer Anything)

> **Stack:** T2 leaf under `mcp` (third-party MCP, not in-house). Upstream: https://github.com/morluto/rea (MIT). Fork: `jaded423/rea` (GitHub) + `jaded/rea` (Gitea). Born 2026-10-13.

Upstream's own agent notes are in `AGENTS.md`; the user docs are in `docs/` and `README.md`. This file holds only what is ours.

## What it is

One MCP server + CLI (`rea`) that lets a coding agent inspect software it did not write: native binaries (PE/ELF/Mach-O), Electron/JS apps, .NET, APKs, WASM, pcaps, firmware. Native analysis needs an engine: Ghidra (free), Hopper or IDA (paid). It rebuilds behaviour and logic, not original source.

First job: Super Collapse! II (GameHouse, 2002, Windows PE) for a Linux/iOS recreation. Plan file: `~/.claude/plans/global.super-collapse-2.pot-plan.md`.

## Where it runs — engine on `lab` (VM103), CLI on both, user-local, no sudo

**2026-10-13 evening: the Ghidra engine moved to `lab`.** The first real binary (an 870 KB x86 PE) OOM-killed Ghidra's java on multi-verse (3.9 GB total, two other Claude sessions resident; `dmesg` 18:16). `lab` has 4 vCPU / 7.9 GB. The multi-verse `rea` MCP registration now runs REA **on lab over ssh** (`~/.claude.json`: `ssh lab env GHIDRA_INSTALL_DIR=… JAVA_HOME=… PATH=… /home/j/.local/bin/rea mcp`; the previous local registration is in `~/.claude.json.rea-lab.backup`). The MCP speaks stdio, so ssh is a transparent transport; targets must be paths **on lab** (copy the file over first, e.g. `~/sc2/`).

Lab install (same recipe, 2026-10-13): Node 24 tarball at `~/.local/opt/node`, `rea-agents@6.4.0` under `~/.local`, Ghidra + JDK rsynced from multi-verse into `~/.local/opt/`. Lab's PATH does not include `~/.local/bin`; every command sets `PATH=$HOME/.local/opt/node/bin:$HOME/.local/bin:$PATH` plus the two engine vars. CLI from here: `ssh lab 'export GHIDRA_INSTALL_DIR=$HOME/.local/opt/ghidra JAVA_HOME=$HOME/.local/opt/jdk21 PATH=$HOME/.local/opt/node/bin:$HOME/.local/bin:$PATH; rea analyze ~/sc2/relapse.exe --json'`.

### multi-verse (VM123) — CLI + skill only now

| Thing | Path |
|---|---|
| Runtime (npm, pinned) | `~/.local/lib/node_modules/rea-agents` → `~/.local/bin/rea` (`npm_config_prefix=$HOME/.local npm install --global rea-agents@<ver>`) |
| MCP registration | `~/.claude.json` → `mcpServers.rea` = `/home/j/.local/bin/rea mcp` (written by `rea setup --client claude_code --yes`; backup `~/.claude.json.rea.backup`) |
| Skill | `~/.claude/skills/reverse-engineer-anything/` (vendored by setup; refresh with `rea setup --skill-client claude_code`) |
| Ghidra 12.1.x | `~/.local/opt/ghidra` (`GHIDRA_INSTALL_DIR`) |
| JDK 21 (Temurin) | `~/.local/opt/jdk21` (`JAVA_HOME`) |
| Installer for the two above | `~/.local/opt/rea-tools-install.sh` (idempotent; undo = `rm -rf ~/.local/opt/{ghidra,jdk21,dl}`) |
| REA state/cache | `~/.rea` (`rea uninstall --purge-data` removes it) |

Why user-local: multi-verse holds no secrets and gets no sudo installs by default; everything here is removable with `rm -rf` and leaves pacman untouched. Switch to `pacman -S ghidra jdk21-openjdk` later if preferred (then re-run `rea setup` so the registration picks up the new `GHIDRA_INSTALL_DIR`).

Sizing: multi-verse is 2 vCPU / 3 GB RAM / 512 MB swap: fine for `rea doctor` and tiny inputs only. Measured 2026-10-13: `rea analyze` on a 40 KB ELF (`/usr/bin/true`) = 37 s, peak +740 MB over idle (idle was 2.5 GB used with Claude sessions running), exit 0. A 2002-era PE of a few MB should fit; watch `free -m` on the first real run. A big binary → run Ghidra on `lab` (VM103, 4 vCPU / 7 GB) instead; REA needs the engine local to it.

## Daily use

- CLI runs need the engine env on the command (the MCP registration carries it, the shell does not): `GHIDRA_INSTALL_DIR=$HOME/.local/opt/ghidra JAVA_HOME=$HOME/.local/opt/jdk21 rea analyze <file> --json`. Without it: `provider_unavailable`.
- Health: `rea doctor --json` (providers, registration, skill). Dry-run any setup change: `rea setup --client claude_code --yes --dry-run --json`.
- CLI: `rea --help`; native commands and snapshots → `docs/cli.md`. Inputs are untrusted downloads: keep each in its own empty dir, pass paths as arguments.
- Update: bump the pin (`npm_config_prefix=$HOME/.local npm install --global rea-agents@<new>`), then `rea setup --client claude_code --yes`.

## The fork

`main` tracks upstream; our additions are this file and `TODO.md` only. Sync: `git fetch upstream && git merge upstream/main`. Patches, if ever needed, go on a branch and get a PR upstream; the npm pin is what actually runs.
