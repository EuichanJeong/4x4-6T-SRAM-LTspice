# 4×4 6T SRAM Array Design in LTspice

![4×4 SRAM Array](images/03_4x4_SRAM_array.png)

## Overview

This project explores the transistor-level design and simulation of a **6T SRAM cell** and its expansion into a functional **4×4 SRAM array** using LTspice.

The design was developed progressively from a single SRAM cell to a reusable hierarchical cell, a 2×2 validation array, and finally a 16-cell 4×4 memory array.

The main goals were to understand:

* CMOS-based SRAM storage
* Cross-coupled inverter operation
* Differential bitlines
* Wordline-controlled access
* Multi-bit parallel write operations
* Data retention
* Row isolation
* Hierarchical SRAM array design

---

## Final Architecture

The final array contains:

* **16 6T SRAM cells**
* **4 rows × 4 columns**
* **4 wordlines**
* **4 differential bitline pairs**
* **16 total stored bits**
* **1.2 V supply voltage**

Each row stores one 4-bit word.

| Row   | Stored Data |
| ----- | ----------- |
| Row 0 | `1010`      |
| Row 1 | `0110`      |
| Row 2 | `1100`      |
| Row 3 | `0011`      |

---

## 6T SRAM Cell

The SRAM cell consists of:

* 2 PMOS pull-up transistors
* 2 NMOS pull-down transistors
* 2 NMOS access transistors
* Internal storage nodes `Q` and `QB`
* Differential bitlines `BL` and `BLB`
* Wordline `WL`

Two cross-coupled CMOS inverters create a bistable storage structure capable of storing one binary bit.

![6T SRAM Cell](images/01_single_6T_SRAM_cell.png)

The individual cell was first tested independently to verify write and hold behavior before being reused in larger arrays.

---

## Hierarchical SRAM Cell

After validating the transistor-level SRAM cell, the design was converted into a reusable LTspice hierarchical symbol.

The external ports are:

* `VDD`
* `WL`
* `BL`
* `BLB`

The internal storage nodes `Q` and `QB` remain inside the SRAM cell.

This hierarchical approach allowed the same verified 6T SRAM circuit to be reused throughout the 2×2 and 4×4 arrays without manually rebuilding the six-transistor circuit for each memory cell.

---

## 2×2 SRAM Array

A 2×2 SRAM array was constructed as an intermediate validation step.

Cells in the same row share a common wordline, while cells in the same column share a differential bitline pair.

The tested patterns were:

| Row   | Stored Data |
| ----- | ----------- |
| Row 0 | `10`        |
| Row 1 | `01`        |

This stage was used to verify:

* Multi-bit writing
* Shared wordline operation
* Shared differential bitlines
* Row isolation
* Data retention

![2×2 SRAM Array](images/02_2x2_SRAM_array.png)

---

## 4×4 SRAM Array

The validated 2×2 structure was expanded into a 4×4 array containing 16 SRAM cells.

The final array uses:

* `WL0` – `WL3`
* `BL0 / BLB0`
* `BL1 / BLB1`
* `BL2 / BLB2`
* `BL3 / BLB3`

Each wordline selects one complete row of four SRAM cells, allowing one 4-bit word to be written in parallel.

---

## Write Timing

The four rows were written sequentially using separate wordline pulses.

| Wordline | Approximate Active Interval |
| -------- | --------------------------- |
| WL0      | 1–3 ns                      |
| WL1      | 5–7 ns                      |
| WL2      | 9–11 ns                     |
| WL3      | 13–15 ns                    |

Piecewise-linear voltage sources were used on the bitlines so that a different 4-bit data pattern was applied before each row was selected.

---

## Simulation Results

### Row 0 — `1010`

![Row 0 — 1010](images/04_row0_1010_waveform.png)

### Row 1 — `0110`

![Row 1 — 0110](images/05_row1_0110_waveform.png)

### Row 2 — `1100`

![Row 2 — 1100](images/06_row2_1100_waveform.png)

### Row 3 — `0011`

![Row 3 — 0011](images/07_row3_0011_waveform.png)

The simulations confirmed that each row stored its intended 4-bit word and retained that value after its corresponding wordline was disabled.

Previously written rows also maintained their stored data while later rows were being written.

---

## Verified Behaviors

The project verified:

* Logic `0` and logic `1` storage
* SRAM write operation
* Data retention after the wordline is disabled
* Multi-bit parallel writing
* Differential bitline operation
* Independent row selection
* Row isolation
* 2×2 SRAM array operation
* 4×4 SRAM array operation
* Sequential storage of multiple 4-bit words

---

## Design Iterations and Debugging

Several design iterations were required while scaling the circuit from a single SRAM cell to the final 4×4 array.

### SRAM Cell Simplification

The original test circuit contained additional circuitry used during early experiments. Before creating the hierarchical SRAM block, the design was reduced to the core six-transistor SRAM storage cell.

This separated the SRAM storage element from external test circuitry.

### Bitline Organization

During early array construction, the bitlines were initially organized incorrectly.

The final structure was corrected so that:

* Wordlines are shared horizontally across each row.
* Bitlines and complementary bitlines are shared vertically across each column.

### Hierarchical Node Observation

After converting the SRAM cell into a hierarchical LTspice block, the internal `Q` and `QB` nodes were initially not visible in the waveform viewer.

Subcircuit node-voltage saving was enabled so that the internal state of each SRAM cell could be observed directly.

### Write Verification

Some cells initially started in the same state as the intended write value, making it difficult to determine whether an actual write transition had occurred.

Opposite initial conditions were therefore applied during validation to confirm that the write operation could actively change the stored state.

### 4×4 Bitline Control

Early 4×4 simulations produced unintended or identical cell states when the bitlines were not independently driven for each write interval.

Piecewise-linear bitline sources were introduced so that each row received the intended 4-bit pattern during its wordline activation period.

---

## Relationship to Practical SRAM

This project represents a simplified version of a practical SRAM memory array.

In a real SRAM macro, additional peripheral circuitry would normally include:

* Row decoder
* Column multiplexer
* Bitline precharge circuit
* Write driver
* Differential sense amplifier
* Timing and control logic

A simplified write path is:

```text
Input Data
    ↓
Write Driver
    ↓
BL / BLB
    ↓
Selected Wordline
    ↓
SRAM Cell
```

A practical read path would be:

```text
Selected Wordline
    ↓
SRAM Cell
    ↓
BL / BLB Differential Signal
    ↓
Sense Amplifier
    ↓
Digital Output
```

SRAM is commonly used in high-speed on-chip memories such as processor caches, embedded memory blocks, and local buffers.

---

## Limitations

This project focuses on transistor-level SRAM behavior and small-array organization.

The following were outside the scope of the final implementation:

* Full row decoder
* Column decoder
* Column multiplexer
* Dedicated write driver
* Complete differential sense amplifier
* Full bitline precharge network
* Physical transistor layout
* DRC / LVS verification
* Process-voltage-temperature analysis
* Statistical mismatch analysis
* Large-array timing and power optimization

The MOSFET models used in LTspice are simplified and are intended primarily for functional and educational transistor-level simulation.

---

## Future Work

Possible extensions include:

* Differential sense amplifier design
* Bitline precharge circuitry
* Row decoder implementation
* Write-driver circuitry
* Static noise margin analysis
* Read/write delay measurement
* Static and dynamic power analysis
* Transistor sizing optimization
* Process and temperature variation analysis
* Larger SRAM arrays
* Physical CMOS layout
* DRC and LVS verification

---

## Repository Structure

```text
4x4-6T-SRAM-LTspice/
│
├── README.md
│
├── images/
│   ├── 01_single_6T_SRAM_cell.png
│   ├── 02_2x2_SRAM_array.png
│   ├── 03_4x4_SRAM_array.png
│   ├── 04_row0_1010_waveform.png
│   ├── 05_row1_0110_waveform.png
│   ├── 06_row2_1100_waveform.png
│   ├── 07_row3_0011_waveform.png
│   └── sram_github_images.zip
│
├── ltspice/
│   ├── SRAM_Cell.asc
│   ├── SRAM_Cell.asy
│   ├── 2x2_SRAM_ARRAY.asc
│   └── 4x4_SRAM_ARRAY.asc
│
└── report/
    └── 4x4_6T_SRAM_Array_Report.pdf
```

---

## Project Report

A detailed explanation of the design process, simulation methodology, results, limitations, and future work is available in the full project report:

[View the Full Project Report](report/4x4_6T_SRAM_Array_Report.pdf)

---

## Tools

* LTspice 26
* Transistor-level CMOS simulation
* Hierarchical schematic design
* Transient analysis

---

## Summary

This project demonstrates the progression from an individual CMOS SRAM storage cell to a functional small-scale memory array.

A transistor-level 6T SRAM cell was designed and simulated, converted into a reusable hierarchical block, and expanded first into a 2×2 validation array and finally into a 4×4 SRAM array.

The final 16-cell array successfully stored four independent 4-bit words:

* `1010`
* `0110`
* `1100`
* `0011`

The stored values remained stable after each wordline was disabled and while subsequent rows were being written.
