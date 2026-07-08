# CPL Rotation Notes — JLCPCB upload preview (MANDATORY human check)

Generated with Stage 4 fab package (Gate G4), 2026-07-07.
Source: `fab/cpl_jlcpcb.csv` (kicad-cli pos export, front side, mm, rotations
normalized 0–360). **No rotation offsets were pre-applied.**

## Why no offsets were baked in

Per the kicad-happy `jlcpcb` skill guidance: JLCPCB's pick-and-place uses
different rotation conventions than KiCad for some footprint families, but
JLCPCB also maintains its own per-LCSC-part orientation database and its
order-review step renders every part on the board. Blindly adding the
"typical" offsets on top of that database risks double-rotation. This is our
**first assembly order** of this board — the skill's process is: verify in
the upload preview now, then record any corrections needed and apply them to
future CPL exports.

**At upload time: open the JLCPCB assembly preview and visually confirm pin-1
/ polarity of every part below against `checks/pcb_top.png` (approved 3D
render) before paying.**

## Parts to eyeball, with expected-offset hints (from the jlcpcb skill table)

| Ref | Package | CPL rotation | Skill's typical JLCPCB offset | What to check in preview |
|-----|---------|--------------|-------------------------------|--------------------------|
| U1 | QFN-80 (RP2350B) | 0.0 | QFN: +90° | Pin-1 dot top-left when text upright; 0.4 mm pitch — any rotation error is fatal |
| U2 | SOT-223 (NCP1117) | 270.0 | SOT-223: +180° | Tab (pin 2/Vout) toward board edge as in render |
| U3 | SOIC-8 (W25Q128JVS) | 0.0 | SOIC-8: +90° or +270° | Pin-1 dot orientation vs silkscreen dot |
| J1 | USB Micro-B (Amphenol 10103594-0001LF) | 180.0 | USB connectors: varies | Receptacle opening must face off-board edge |
| L1 | L_pol_2016 (AOTA-B201610S3R3-101-T) | 0.0 | not in table — **polarized inductor** | **POLARITY DOT** must match reference layout (G1 note); VREG switcher inductor |
| Y1 | Crystal 3225 4-pin (ABM8-272-T3) | 0.0 | not in table | Pin-1 corner vs footprint; 4-pad crystal shorts GND if rotated 90° |

Lower-risk but glance anyway: SW1/SW2 (270/90), J4 (JST SH, 0.0) — connector
keying visible in preview; passives are symmetric.

## Other CPL conventions

- Coordinates are KiCad pos-file convention (absolute board origin, Y
  negative-down); JLCPCB auto-aligns the CPL to the gerbers in its preview —
  if parts land off-board in the preview, use JLCPCB's "align" tools, do not
  hand-edit coordinates.
- J2/J3 (2×26 0.100" headers) are through-hole: they appear in BOM+CPL for
  Standard (THT-capable) assembly; delete both lines from BOM+CPL instead if
  ordering Economic/SMD-only and hand-soldering the headers.
- After this first order: record every correction JLCPCB's preview required
  into this file and pre-apply them in the next CPL export (skill guidance).
