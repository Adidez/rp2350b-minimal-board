# Gate G1 — PASS (2026-07-07)

Schematic: design/RP2350_80QFN_minimal.kicad_sch @ 452c132.
Independent verify: ERC 0 errors (66 endpoint_off_grid + 6 lib_symbol_mismatch keep-set) ·
39 MPN + 39 LCSC properties · 17 datasheet PDFs + manifest (kicad-happy convention) ·
analyzer: 2 findings total = the known-benign RS-001 pair; DS-001 cleared; netlist membership
75/75 unchanged (agent-verified, s-expression compare).

## Ordering table (live stock 2026-07-07 — RE-VERIFY at order time)

| Refs | MPN | LCSC | Tier | Stock seen | Notes |
|---|---|---|---|---|---|
| C1,C5,C19 10u 0805 | CL21A106KAYNNNE | C15850 | Basic | 12.7M | |
| C2,C8,C11–C18,C20,C21 100n | CL05B104KO5NNNC | C1525 | Basic | 54M | |
| C3,C4 15p C0G | 0402CG150J500NT | C1548 | Basic | 882K | crystal loads |
| C6,C7,C9,C10 4.7u | CL05A475MP5NRNC | C23733 | Basic | 4.8M | |
| L1 3.3µH | AOTA-B201610S3R3-101-T | C42411119 | Extended | 22.5K | exact qualified part; POLARITY DOT |
| R2,R4,R6 1k | 0402WGF1001TCE | C11702 | Basic | 12.6M | |
| R3 33 | 0402WGF330JTCE | C25105 | Basic | 4.8M | |
| R7,R8 27 | 0402WGF270JTCE | C25100 | Extended | 273K | USB series; no Basic 27Ω exists |
| R10 0R | 0402WGF0000TCE | C17168 | Basic | 18.4M | |
| Y1 | ABM8-272-T3 | C20625731 | Extended | 15.7K | CL=10pF qualified |
| U1 | RP2350B **A4** | C9900208890 | Extended | 0 shelf — JLC global-sourcing, procured per order | **reserve early**; alt C42415655 rejected (stepping unverified) |
| U2 | NCP1117ST33T3G | C26537 | Extended | 14.2K | |
| U3 | W25Q128JVSIQ | C97521 | Basic | 110K | |
| J1 | 10103594-0001LF | C428495 | Extended | 7.8K | exact Amphenol |
| J2,J3 2×26 | X6521WV-2x26H-C60D30 | C725898 | Extended | **781** | watch stock |
| J4 | SM03B-SRSS-TB(LF)(SN) | C160403 | Extended | 4.5K | SWD |
| SW1,SW2 | **ALPS SKRPACE010** | C139797 | Extended | 156K | **SUBSTITUTION** for Würth 434133025816 (not carried): land pattern verified exact match (1.05×0.65 @ ±2.075/±1.075), same 4.2×3.2 J-bend; 2.55N vs 1.6N actuation |
| H1–H4 | — | — | — | — | mounting holes, no part |

DNF (excluded from order): C22, R1, R9, R11, R13, U4.

## Datasheet corrections made
- W25Q128JVSIQ: schematic's legacy URL pointed at the JV_DTR family doc — corrected to plain JV.
- NCP1117: onsemi direct blocked → LCSC CDN copy, manifest patched.
