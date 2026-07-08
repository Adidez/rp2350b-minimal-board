---
name: rp2350b-hardware-pipeline
description: >-
  Gated hardware-design pipeline for this RP2350B (QFN-80) KiCad project. Use this skill for ANY
  design work in this repo — creating or editing the schematic, adding circuits/sensors/interfaces,
  placement, routing, ERC/DRC, generating gerbers/BOM/CPL, ordering from JLCPCB, or board bring-up.
  Also use it when the user says things like "add X to the board", "route the USB", "is this ready
  for fab?", "check my schematic", or mentions RP2350/RP2350B/Pico2 hardware at all — even if they
  don't name a pipeline stage. It encodes the RP2350B-specific design rules (VREG inductor, E9/A4
  stepping, boot straps, decoupling map) that generic knowledge gets wrong.
---

# RP2350B Hardware Pipeline

You are working on a real board that will be fabricated and must boot. Every stage below has a
gate; do not advance past a failing gate. The reason for the gates: schematic errors cost minutes
to fix now, a respin costs ~2 weeks and real money after fab.

## Ground rules (why they exist)

- **Source of truth for RP2350B circuit values** is the official reference project at
  `reference/RP2350B-minimal/` and the guide `reference/hardware-design-with-rp2350.pdf`
  (§Minimal Design Example). Never invent component values from memory — RP2350 differs from
  RP2040 in ways that trained-in knowledge gets wrong (the core regulator is a *switcher* needing
  an inductor, not an LDO). When in doubt, open the reference schematic and copy its values.
- **Separation of powers**: kicad-mcp-pro and the routing tools are *actuators*; `kicad-cli`
  ERC/DRC is *ground truth*; kicad-happy skills are the *independent reviewer*. Never let a tool
  self-certify its own output — always close a stage with kicad-cli + a kicad-happy review.
- **Consult `references/rp2350b-design-rules.md` before touching** power, flash/QSPI, USB,
  crystal, or any GPIO strap. Consult `references/toolchain.md` for exact verified commands and
  tool selection.
- Work on git branches; commit after every passing gate so any stage can be rolled back.

## Stage 0 — Baseline (do once)

1. Copy `reference/RP2350B-minimal/` → `design/` (never edit the reference in place).
2. Migrate KiCad 7 → 10: ensure global lib tables exist (fresh installs have none — copy from
   the app bundle's `SharedSupport/template/`), add PWR_FLAG to the internal-VREG rails
   (`+1V1`, `VREG_AVDD`), re-link footprints renamed by v10 (instance properties only).
   Do NOT grid-snap `endpoint_off_grid` warnings (snapping risks disconnecting nets that
   connect today) and do NOT update cached lib_symbols (reference values are law) — both
   warning classes are kept and documented.
3. **Gate G0**: `kicad-cli sch erc` → 0 errors; residual warnings limited to the documented
   keep-set (`endpoint_off_grid`, `lib_symbol_mismatch`). Prove any schematic edit with a
   netlist-equivalence diff. Run kicad-happy's analyzer and store the report in `checks/`
   (note: its RS-001 doesn't credit PWR_FLAG — benign noise on the VREG rails; kicad-cli is
   the gate authority). See `references/toolchain.md` § surgery quirks before editing.

## Stage 1 — Schematic work

- Edit via kicad-mcp-pro `sch_*` tools (headless, no GUI needed) or direct `.kicad_sch`
  s-expression edits for surgical changes. Keep hierarchical structure; new subsystems go on
  their own sheet.
- For every new part: resolve a real MPN + LCSC number (kicad-happy `lcsc`/`digikey` skills),
  populate MPN/LCSC fields immediately, and pull the datasheet into `datasheets/` so reviews are
  "verified against datasheet", not "per symbol".
- After each meaningful change: `kicad-cli sch erc` (must stay at 0 errors) — cheap, run it often.
- **Gate G1**: ERC 0 errors · kicad-happy `kicad` skill review clean (power tree, decoupling,
  nets) · decoupling map satisfied per design rules · boot straps (QSPI_SS, QSPI SD1) unloaded ·
  every part has MPN + LCSC + datasheet.

## Stage 2 — Placement

- Update PCB from schematic, then place with intent before any routing. Priority order:
  (1) crystal loop tight against XIN/XOUT, away from USB; (2) QSPI flash close to the chip,
  short matched runs off the 0.4 mm-pitch pads; (3) VREG inductor + caps per reference layout;
  (4) 100 nF decouplers hard against their pins; (5) USB connector → clean short diff-pair path.
- Live placement via kicad-mcp-pro `pcb_*` (requires KiCad GUI open with IPC API enabled) or
  file-based via the routing tools.
- **Gate G2**: run `/plan-pcb-routing` on the board — the plan must not flag placement blockers
  (unroutable fanout, diff-pair crossings, missing room for GND vias).

## Stage 3 — Routing

- Critical nets first, by hand or AI-guided, in this order: USB D+/D− as a coupled pair
  (`route_diff.py`), QSPI group, crystal. Then power. Then let `grid_router` batch-route the
  boring nets. Full-board autoroute is banned — cleanup costs more than it saves.
- Use `/identify-diff-pairs` and `/find-high-speed-nets` to seed the plan; add GND return vias
  per their recommendation.
- **Gate G3**: `kicad-cli pcb drc` → 0 errors · `/review-routed-board` clean · connectivity check
  passes (no orphan stubs).

## Stage 4 — Fab package

- Exports via `kicad-cli` (gerbers, drill, position); BOM/CPL in JLCPCB format via kicad-happy
  `bom` + `jlcpcb` skills; run kicad-happy's fab release gate last.
- Parts: order **A4-stepping silicon only** and verify live stock before committing — exact LCSC
  numbers and the stepping story are in `references/rp2350b-design-rules.md`.
- **Gate G4**: fab release gate PASS · BOM 100% LCSC-resolved · gerber visual spot-check.

## Stage 5 — Bring-up

- First power: check 3V3, then 1V1 (proves the switching VREG + inductor). Hold BOOTSEL
  (QSPI_SS→GND) while plugging USB → RP2350 must enumerate as mass storage.
- Flash a UF2; confirm stepping in firmware (`rp2350_chip_version()` — must report A4).
- Any boot failure: re-read the straps + flash sections of the design rules before touching
  hardware.

## Escalation

3 failed attempts at any gate → stop, write down the failing evidence (ERC/DRC output, review
findings), and present the human with the options: different approach / relax the constraint /
show the failing area. Never loop silently.
