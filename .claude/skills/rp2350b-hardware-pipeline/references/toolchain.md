# Toolchain — verified commands (this machine, July 2026)

macOS Apple Silicon · KiCad 10.0.4 · Python 3.14.2 · Claude Code 2.1.204

## kicad-cli (ground truth for ERC/DRC/exports)

Not on PATH — use the bundle path (or add an alias):

```bash
KCLI=/Applications/KiCad/KiCad.app/Contents/MacOS/kicad-cli
$KCLI sch erc  --output out.rpt --severity-all design/RP2350_80QFN_minimal.kicad_sch
$KCLI pcb drc  --output out.rpt design/RP2350_80QFN_minimal.kicad_pcb
$KCLI pcb export gerbers --output fab/gerbers/ design/*.kicad_pcb
$KCLI pcb export drill   --output fab/gerbers/ design/*.kicad_pcb
$KCLI pcb export pos     --output fab/cpl.csv --format csv --units mm design/*.kicad_pcb
```

KiCad 10's official libs ship the RP2350B symbol (`MCU_RaspberryPi:RP2350B`, 81 pins) and QFN-80
footprint natively.

## kicad-mcp-pro (actuator MCP — 368 tools, registered as `kicad-pro`)

- Registered user-scope: `uvx kicad-mcp-pro --transport stdio`, env `KICAD_MCP_PROFILE=schematic_only`.
- Profile controls which tools load. Switch by editing the env in `~/.claude.json` (server
  `kicad-pro`): `schematic_only` → `pcb_only` → `manufacturing` (or `full`). Restart session after.
- `sch_*` tools are **headless** (edit files directly). `pcb_*` live tools need the KiCad GUI
  open with **Preferences → Plugins → Enable IPC API** checked.
- Trust model: its gates/SI/PI outputs are first-pass heuristics — always confirm with kicad-cli
  and a kicad-happy review (it's a 5-week-old AI-built surface; treat as intern, verify as senior).

## kicad-happy v1.3.2 (independent reviewer — installed Claude Code plugin)

- 12 skills auto-load each session: `kicad` (schematic/PCB analysis), `bom`, `datasheets`,
  `lcsc`/`digikey`/`mouser`/`element14`, `jlcpcb`/`pcbway`, `emc`, `spice`, `kidoc`.
- Scripts also runnable directly (pure stdlib), e.g.:
  `python3 ~/Desktop/work/skill-review/kicad-happy/skills/kicad/scripts/analyze_schematic.py <sch> --text`
- Datasheets convention: PDFs into `./datasheets/`, MPN fields populated → findings upgrade from
  "per symbol" to "verified against datasheet".

## KiCadRoutingTools v0.17.x (routing layer — repo at `tools/KiCadRoutingTools/`)

- Rust router installed: `tools/KiCadRoutingTools/rust_router/grid_router.so` (prebuilt arm64).
- **Scripts must run with CWD = `tools/KiCadRoutingTools/`** (they use relative imports):
  `route.py`, `route_diff.py` (coupled diff pairs), `qfn_fanout.py`, `add_gnd_vias.py`,
  `check_drc.py`, `check_connected.py`, `analyze_power_paths.py`.
- 9 skills symlinked into `.claude/skills/`: `/plan-pcb-routing`, `/identify-diff-pairs`,
  `/find-high-speed-nets`, `/analyze-power-nets`, `/recommend-stackup`,
  `/recommend-plane-mappings`, `/review-routed-board`, `/diagnose-routing-failures`,
  `/stress-test-router`.

## Layout of this project

```
reference/   RP2350B-minimal/ (official, read-only) · hardware-design-with-rp2350.pdf · Minimal-KiCAD.zip
design/      working copy of the KiCad project (Stage 0 creates it)
datasheets/  per-MPN PDFs for verified reviews
fab/         gerbers, BOM, CPL, release-gate reports
tools/       KiCadRoutingTools
```

## .kicad_sch surgery quirks (learned Stage 0, 2026-07-07 — read before editing schematics)

- KiCad 10's `power:PWR_FLAG` pin name is **empty** (v7 syntax: `(name "~")`), not "pw" — using
  the old name adds a lib_symbol_mismatch warning.
- ERC JSON `pos` values are **mm/100** (1.1938 → 119.38 mm). Not inches.
- Power symbols (`#PWR`/`#FLG`) never appear in exported netlists — prove PWR_FLAG placement by
  the error vanishing + unchanged net membership, not by looking for #FLG nodes.
- Local labels get a `/` prefix in netlist names (`VREG_AVDD` → `/VREG_AVDD`).
- v7 `ki_description` compares equal to v10 `Description` in the library-mismatch checker.
- Footprint strings can exist in 3 places; fix instance properties only — the cached
  `lib_symbols` copy stays stale on purpose (keep-cached policy).
- kicad-happy RS-001 does not credit PWR_FLAG symbols (upstream limitation) — its +1V1/VREG_AVDD
  findings are permanent benign noise in this project; kicad-cli ERC is the gate authority.

## Deliberately NOT in the stack (evaluated + rejected July 2026)

- BeckhamLabsLLC/kicad-jlcpcb (dead parts backend), colaco1123/K-AI (abandoned, ToS risk),
  lamaalrajih/kicad-mcp (read-only, stale), circuit-synth (ERC-dirty output, dormant),
  atopile (no .kicad_sch, no RP2350 package), skidl-skills (netlist dead-end; SKiDL venv kept at
  `~/Desktop/work/skill-review/venv-skidl` for netlist experiments), Bouni JLCPCB plugin
  (redundant), full-board autorouting (cleanup > savings).
- Optional, not installed: Diode `pcb`/Zener (Anthropic-trained schematic codegen — installer
  pending user approval), KiBot (adopt when CI exists), Freerouting MCP (fallback router).
