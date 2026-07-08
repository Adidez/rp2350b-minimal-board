# Gate G3 — FINAL: PASS (2026-07-07, post-Stage-2, commit adabefb)

Independent kicad-cli DRC on the v10-converted board (checks/G3_drc.json): **0 errors,
0 unconnected, 0 schematic-parity**; 52 metadata warnings (37 lib_footprint_mismatch keep-embedded,
9 silk_over_copper, 4 silk_overlap, 2 lib_footprint_issues). Routing-tools checkers non-vacuous:
511 segments / 48 vias / 332 pads, 73/73 nets connected, 0 violations. Pad→net equivalence vs
pre-conversion: 332/332 identical. J1/U1 footprint_type_mismatch resolved at ROOT CAUSE: v7
project severity policy → KiCad-10 factory default (ignore); attrs correctly remain SMD for CPL.

---

# Gate G3 Pre-validation — official routing (2026-07-07)

Board: design/RP2350_80QFN_minimal.kicad_pcb (official routing, unmodified). Verdict: **PASS as-is**.

| Check | Tool | Result |
|---|---|---|
| DRC electrical | kicad-cli 10.0.4 --severity-all --refill-zones | 0 electrical (54 metadata/cosmetic only) |
| DRC geometric | check_drc.py @0.15mm (on v10-converted copy) | 0 violations |
| Connectivity | check_connected.py | 73/73 nets (511 segs, 48 vias, 332 pads, 9 zones) |
| Orphan stubs | check_orphan_stubs.py | 11 zone-fed tails (+3V3 ×10, VBUS) — benign, inside own-net pours |
| USB diff pair | geometry analysis | conn side 0.4mm/~0.53mm gap, chip side 0.15mm/~0.23mm; skew 331µm total — fine for USB-FS |
| QSPI/crystal | route-length analysis | QSPI 5.96–26.38mm (10.2mm spread, unmatched — irrelevant at QSPI rates); XIN 11.3mm |
| GND return | zone/segment analysis | full-board B.Cu GND zone; 1 cut under USB (+1V1 jumper, 5mm); QSPI dips carve locally |

## Reference-design quirks — documented, NOT to be "fixed"
- 54 metadata DRC findings incl. J1/U1 footprint-type attribute "errors" (fix attrs in Stage 2 for CPL correctness; rest keep)
- USB pair loosely coupled (~0.5mm gap), not impedance-controlled — normal for FS on the official board
- QSPI unmatched + local B.Cu excursions under itself; +1V1 crosses under USB on B.Cu
- 11 zone-fed stubs; XIN 11.3mm

## Stage-2 obligations discovered
1. Save/convert the PCB to KiCad 10 format during reconcile — REQUIRED for checker validity (see toolchain quirk: v7 tstamp parsing trap)
2. Fix J1/U1 footprint attributes (SMD) — position-file correctness
3. Update-PCB-from-schematic to sync MPN/LCSC fields + U3/U4 footprint rename
