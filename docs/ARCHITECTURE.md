# Architecture

## Overview

The design is a simple synchronous scan-decode-display pipeline:

- `PmodKYPD` (top): wires keypad and display blocks
- `Decoder`: scans keypad matrix and outputs decoded nibble
- `DisplayController`: maps nibble to 7-segment pattern

## Block Diagram

```text
             +-------------------+
clk -------->|                   |
JA[7:4] ---->|     Decoder       |---- DecodeOut[3:0] ----+
             |   (scan + decode) |                         |
             +--------+----------+                         v
                      |                             +---------------+
                      +---- Col[3:0] -> JA[3:0] -->|DisplayControl |
                                                    |(hex to 7-seg) |
                                                    +-------+-------+
                                                            |
                                                   an[3:0], seg[6:0]
```

## Module Responsibilities

## 1) `PmodKYPD` (top-level)

- Inputs:
  - `clk`: 100 MHz system clock
  - `JA[7:0]`: keypad interface bus (shared row/column)
- Outputs:
  - `an[3:0]`: seven-segment digit select (active-low)
  - `seg[6:0]`: seven-segment segments (active-low)
- Instantiates:
  - `Decoder`
  - `DisplayController`

## 2) `Decoder`

- Uses a 20-bit free-running scan counter (`sclk`) to create time points.
- At each scan phase:
  - Drives one keypad column low (`Col`).
  - Samples row inputs (`Row`) shortly after.
  - Updates `DecodeOut` when a row is detected low.
- Active-low keypad convention:
  - Column low = selected column.
  - Row low = pressed key on active column.

### Decode Table

| Row | Col1 | Col2 | Col3 | Col4 |
|---|---|---|---|---|
| R1 | `1` | `2` | `3` | `A` |
| R2 | `4` | `5` | `6` | `B` |
| R3 | `7` | `8` | `9` | `C` |
| R4 | `0` | `F` | `E` | `D` |

## 3) `DisplayController`

- Drives fixed anode mask: `1110` (single digit active).
- Performs combinational lookup from 4-bit value to 7-segment pattern.
- Supports full hexadecimal display (`0`-`F`).

## Timing Model

- Clock is constrained to 100 MHz in XDC (`create_clock -period 10.00`).
- Design is fully synchronous to `clk`.
- Existing routed timing report indicates constraints are met.

## Pin/IO Mapping

Key interfaces constrained in `Nexys4DDR_Master.xdc`:

- `clk` on board 100 MHz oscillator pin
- `JA[7:0]` on Pmod JA header
- `an[3:0]`, `seg[6:0]` on onboard 7-segment display

## Design Limitations (Current)

- `JA` is declared as top-level `inout`, producing IO buffering warnings in Vivado.
- No explicit reset path in datapath/control.
- No debounce or key-event qualification.
- Single-digit display only.

## Extension Path

To scale this design:

1. Add reset input and deterministic startup behavior.
2. Add debounce finite-state machine and one-pulse key valid output.
3. Add multi-digit display mux (for history/value + status).
4. Refactor IO to explicit row inputs and column outputs.
5. Add simulation testbench for scan/decode correctness and bounce robustness.
