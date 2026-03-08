# Development Guide

## Toolchain

- FPGA Board: Nexys4 DDR (Artix-7)
- Primary project format: Vivado project (`.xpr`)
- Existing run artifacts suggest Vivado 2018.2 environment

## Build Flow

1. Open project: `PMOD_KYPD/PMOD_KYPD.xpr`
2. Verify sources and constraints are present
3. Run synthesis
4. Run implementation
5. Generate bitstream
6. Program board from Hardware Manager

## Source Files

- Top-level:
  - `.../PmodKYPD.vhd`
- Functional logic:
  - `.../Decoder.vhd`
  - `.../DisplayController.vhd`
- Constraints:
  - `.../Nexys4DDR_Master.xdc`

## Coding Notes

- Current code uses:
  - `STD_LOGIC_ARITH`
  - `STD_LOGIC_UNSIGNED`
- Recommended migration:
  - `ieee.numeric_std`
  - `unsigned` for counters and arithmetic

## Verification Status

- Timing constraints in current routed report are met.
- No testbench is committed in repository.
- External IO delays (`set_input_delay`, `set_output_delay`) are not fully constrained.

## Recommended Verification Additions

1. Unit testbench for `Decoder`:
   - column scan order
   - row sampling window
   - full key map coverage
2. Bounce injection test:
   - confirm debounce policy
3. Integration testbench:
   - decode to 7-seg mapping correctness

## Hardware Bring-Up Checklist

1. Confirm Pmod KYPD orientation on JA.
2. Confirm bitstream matches current source/constraints.
3. Validate all 16 keys map correctly (`0`-`F`).
4. Observe for repeated key artifacts due to bounce.

## Refactor Backlog (Suggested)

1. Add reset input and initialize all control registers.
2. Replace magic timing constants with named constants.
3. Parameterize scan frequency via generic.
4. Split physical IO directionality (rows in, cols out).
5. Add CI script for lint/sim/synthesis checks.
