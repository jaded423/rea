# rea TODO

- [ ] Create the remotes and push `main` (added 2026-10-13). Needs the Pocket (gh auth + Gitea token are not on multi-verse): `gh repo fork morluto/rea --clone=false --org '' ` as jaded423 (or tap Fork on github.com/morluto/rea), create `jaded/rea` on Gitea per `~/.claude/skills/newdir/SKILL.md` Step 3, then `cd ~/projects/rea && git push -u origin main`.
      verify: cd ~/projects/rea && git fetch origin -q && git status -sb | head -1
- [x] 2026-10-13 Ghidra provider green (added 2026-10-13) — done: Ghidra 12.1.4 + JDK 21 user-local, exec bits restored (zipfile drops them), `rea setup` saved `GHIDRA_INSTALL_DIR`/`JAVA_HOME` into the registration; doctor shows ghidra available=True. After `~/.local/opt/rea-tools-install.sh` finishes: `GHIDRA_INSTALL_DIR=~/.local/opt/ghidra JAVA_HOME=~/.local/opt/jdk21 rea setup --client claude_code --yes --json` so the registration carries the env, then restart Claude.
      verify: rea doctor --json | python3 -c 'import sys,json; d=json.load(sys.stdin); print([p["available"] for p in d["providerInspections"] if p["id"]=="ghidra"])'
- [ ] First real run: Super Collapse! II binary (added 2026-10-13). Blocked on Joshua supplying his copy; see the plan file.
