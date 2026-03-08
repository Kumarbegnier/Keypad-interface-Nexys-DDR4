# Keypad Interface on Nexys4 DDR (VHDL)

Interface a Digilent Pmod KYPD (4x4 keypad) with the Nexys4 DDR FPGA board and display pressed keys on the 7-segment display in hexadecimal format (`0`-`F`).

This repository contains a Vivado project with VHDL source, pin constraints, and generated implementation artifacts.

## Features

- 4x4 keypad scanning through JA Pmod header
- Hex key decode (`0`-`F`)
- 7-segment output display
- Vivado project ready (`.xpr`) with generated bitstream included

## Repository Layout

- `PMOD_KYPD/PMOD_KYPD.xpr`: Vivado project
- `PMOD_KYPD/PMOD_KYPD.srcs/sources_1/imports/PmodKYPD_Source/PmodKYPD.vhd`: Top module
- `PMOD_KYPD/PMOD_KYPD.srcs/sources_1/imports/PmodKYPD_Source/Decoder.vhd`: Keypad scan + decode logic
- `PMOD_KYPD/PMOD_KYPD.srcs/sources_1/imports/PmodKYPD_Source/DisplayController.vhd`: 7-segment mapping
- `PMOD_KYPD/PMOD_KYPD.srcs/constrs_1/imports/nexys4ddr_master_xdc/Nexys4DDR_Master.xdc`: Board constraints
- `PMOD_KYPD/PMOD_KYPD.runs/impl_1/PmodKYPD.bit`: Generated bitstream

## System Architecture

High-level data flow:

1. `Decoder` drives keypad columns sequentially (`Col` active-low).
2. `Decoder` samples row inputs (`Row` active-low) at specific scan instants.
3. Row/column combination is converted into a 4-bit hex value.
4. `DisplayController` maps hex value to 7-segment pattern.

Detailed architecture: [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)

## Key Mapping (Hex)

The keypad is decoded to:

| Keypad Row/Column | Hex Output |
|---|---|
| R1C1..R1C4 | `1 2 3 A` |
| R2C1..R2C4 | `4 5 6 B` |
| R3C1..R3C4 | `7 8 9 C` |
| R4C1..R4C4 | `0 F E D` |

## Hardware Setup

1. Board: Digilent Nexys4 DDR (Artix-7).
2. Keypad: Pmod KYPD connected to JA.
3. Clock: On-board 100 MHz clock (`clk`).
4. Display: On-board 7-segment display (`an[3:0]`, `seg[6:0]`).

JA pin constraints are already enabled in the XDC.

## Build and Program (Vivado)

1. Open `PMOD_KYPD/PMOD_KYPD.xpr` in Vivado (project artifacts indicate Vivado 2018.2 was used).
2. Run `Synthesis`.
3. Run `Implementation`.
4. Run `Generate Bitstream`.
5. Open `Hardware Manager` and program the FPGA.

You can also directly use the generated bitstream:

- `PMOD_KYPD/PMOD_KYPD.runs/impl_1/PmodKYPD.bit`

## Behavior Notes

- Current display controller enables only one 7-segment digit (`anode = "1110"`).
- Keypad scan is timer-based and active-low.
- No debounce logic is implemented in the current version.

## Timing/Implementation Snapshot

From existing implementation report:

- Device: `xc7a100t-csg324-1`
- Setup/hold timing met (no failing endpoints)
- Positive setup slack reported in `PmodKYPD_timing_summary_routed.rpt`

## Known Limitations

- `JA` is modeled as `inout` bus at top-level; Vivado reports IO buffering warnings for this style.
- Internal scan counter has no explicit reset port.
- No simulation testbench is included in this repository.
- IO delay constraints for external interfaces are not fully specified.

## Recommended Improvements

- Add explicit reset and counter initialization.
- Add keypad debounce + key-press strobe output.
- Replace non-standard VHDL arithmetic packages with `ieee.numeric_std`.
- Split keypad pins into explicit row inputs and column outputs (or instantiate explicit IOBUF primitives).
- Add a self-checking testbench for decoder logic.

## Development Notes

Developer-focused guidance (coding style, constraints, verification, and extension ideas):

- [`docs/DEVELOPMENT.md`](docs/DEVELOPMENT.md)

## License

No license file is currently present in this repository. Add a `LICENSE` file before public redistribution if needed.
