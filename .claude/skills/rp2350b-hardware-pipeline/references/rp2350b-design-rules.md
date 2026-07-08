# RP2350B Design Rules (verified July 2026)

Authoritative sources, in order: the local reference project `reference/RP2350B-minimal/`
(component values as drawn = qualified), `reference/hardware-design-with-rp2350.pdf`
(§Minimal Design Example, §Power, §Boot), then the links at the bottom. If this file and the
reference schematic ever disagree, the reference schematic wins.

## Silicon stepping — the E9 story (critical when ordering)

- **A2 stepping has erratum E9**: any Bank 0 GPIO with input buffer enabled leaks ~120 µA in the
  intermediate-voltage region and latches ~2.2 V; internal pull-downs cannot hold pins low.
  Workaround on A2: external pull-down ≤ 8.2 kΩ on pins used as inputs, or firmware toggles the
  input buffer between reads.
- **A4 stepping (PCN 2025-07-29) fixes E9**, adds official 5 V-tolerant GPIO spec + security
  fixes. Marking `RP2350B0A4`. A2 was withdrawn from distribution ~Sept 2025 but leftover stock
  exists at assemblers.
- **Rule: order A4 only, and verify at bring-up** via `rp2350_chip_version()` /
  `CHIP_ID.REVISION`. If there is any chance of A2 (e.g. assembler inventory), keep E9
  pull-downs on always-input pins as insurance.

## Power (differs from RP2040 — generic LLM knowledge gets this wrong)

- The core regulator is an **on-chip switching regulator**, not an LDO. It **requires a 3.3 µH
  inductor** — qualified part Abracon **AOTA-B201610S3R3-101-T** — and the inductor has a
  **polarity dot; orient it per the reference layout**.
- `+1V1` (core) and `VREG_AVDD` are internal-VREG rails: ERC reports `power_pin_not_driven` on
  them — expected, do not "fix" by adding phantom sources; a PWR_FLAG annotation is the correct
  silence if needed.
- Decoupling map (each cap hard against its pin): **100 nF at every IOVDD pin (~6 on QFN-80),
  every DVDD pin, USB_OTP_VDD, ADC_AVDD**; bulk **10 µF on IOVDD and DVDD** rails. Copy the
  exact count/values from the reference schematic.
- QFN-80 exposed pad = GND; stitch with thermal vias (the official footprint
  `RP2350-QFN-80-1EP_10x10_P0.4mm_EP3.4x3.4mm_ThermalVias` already has them).

## Boot straps (do not load these nets)

- **QSPI_SS is the BOOTSEL strap**: sampled at boot; low → USB mass-storage bootloader. BOOTSEL
  button = QSPI_SS → GND through ~1 kΩ. Nothing else may drive or heavily load this net at reset.
- **QSPI SD1 selects bootloader interface** inside BOOTSEL: left to the chip's own pull = USB;
  driven high = UART. Do not bias SD1 externally at reset.

## Flash / QSPI

- RP2350B has **no internal flash** — external QSPI NOR is mandatory to boot from flash.
  Qualified reference part: **Winbond W25Q128JVS** (2–16 MB range supported).
- Keep QSPI traces short and matched, straight off the 0.4 mm-pitch pads; route as a group
  before general routing.

## Crystal / clock

- Qualified: **12 MHz Abracon ABM8-272-T3** (CL = 10 pF) with the tuned ~1 kΩ series damping
  resistor and load caps **as drawn in the reference schematic** (values assume 3.3 V IOVDD).
- Do not substitute a crystal with different CL without re-deriving load caps (the JLC
  basic-tier X322512MSB4SI is CL = 20 pF — wrong without rework; pay the extended fee instead).
- Layout: tight loop, ground-guarded, away from USB and switching-VREG inductor.

## USB

- USB-FS (12 Mbps): D+/D− routed together as a pair, short, over solid ground, no stubs.
  Series resistors/terminations per reference schematic.

## QFN-80 mechanics

- 0.4 mm pitch → 0402 passives to break out all 48 GPIOs (8 ADC-capable on Bank 0 high pins).
- The 60-pin (RP2350A) minimal design is NOT this board — always use the QFN-80 variant files.

## Qualified parts + LCSC numbers (verify live stock before ordering)

| Part | Role | LCSC | JLC tier | Note |
|---|---|---|---|---|
| RP2350B **A4** | MCU | **C9900208890** (A4) / C42415655 (verify stepping!) | Extended | Stock volatile — reserve early |
| W25Q128JVSIQ | 16 MB QSPI flash | **C97521** | **Basic** | No feeder fee |
| ABM8-272-T3 | 12 MHz crystal | **C20625731** | Extended | Keep despite $3 fee (CL=10 pF) |
| AOTA-B201610S3R3-101-T | 3.3 µH VREG inductor | search current listing | — | Polarity dot! |

## Sources

- Hardware design with RP2350 (local PDF + https://datasheets.raspberrypi.com/rp2350/hardware-design-with-rp2350.pdf)
- A4 PCN: https://pip.raspberrypi.com/categories/1263-pcn/documents/RP-008771-CC/RP235x-A4-stepping-PCN.pdf
- E9 background: https://hackaday.com/2024/09/20/raspberry-pi-rp2350-e9-erratum-redefined-as-input-mode-leakage-current/
- JLCPCB RP2350 layout guide: https://jlcpcb.com/blog/how-to-design-layout-with-rp2350
