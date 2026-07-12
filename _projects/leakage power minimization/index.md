---
layout: post
title: Leakage Power Minimization Through Gate Sizing
date: 2026-06-10
description: A sensitivity-based Tcl optimization script that minimizes leakage power in IC designs by swapping cells to higher threshold voltage variants, while preserving timing constraints through a two-phase global timing recovery and leakage reduction flow.
skills:
  - Physical design
  - EDA scripting (Tcl)
  - Gate sizing
  - Vt-swapping
  - Static timing analysis
  - Siemens Aprisa
  - TSMC 65GP
main-image: /mp1-cover.png
---

## Project Overview

Leakage power is a dominant contributor to static power consumption in modern CMOS designs, and it increases exponentially as threshold voltage (Vt) decreases. This project develops an automated Tcl optimization script that minimizes leakage power across three benchmark circuits by systematically swapping cells to higher-Vt variants on non-critical paths, while recovering timing on critical paths using lower-Vt and upsized cells where needed.

**Designs:** `usb_phy`, `aes_cipher_top`, `mpeg2_top`  
**Technology:** TSMC 65GP  
**Tool:** Siemens Aprisa (ECO flow)  
**Library:** Triple-Vt (HVT, NVT, LVT) with 10 sizing variants per cell

---

## Background

### Threshold Voltage and Leakage

In a triple-Vt library, cells come in three flavors:

| Variant | Vt | Speed | Leakage |
|---------|-----|-------|---------|
| LVT (f) | Low | Fast | High |
| NVT (m) | Medium | Medium | Medium |
| HVT (s) | High | Slow | Low |

Leakage current increases exponentially as Vt decreases — an LVT cell can leak 5–10× more than an HVT equivalent. The optimization opportunity is to identify cells that have enough timing slack to tolerate being slowed down (swapped to HVT), then make those swaps to recover leakage without violating timing.

### Sensitivity Function

The core idea is to prioritize swaps by their **leakage reduction per unit of slack consumed**:

```
sensitivity = ΔLeakage / |ΔWNS|
```

Cells with high sensitivity — large leakage savings for minimal timing impact — are swapped first. This greedy approach efficiently navigates the leakage-timing tradeoff.

---

## Two-Phase Optimization Flow

### Phase 1: Global Timing Recovery (GTR)

The baseline designs start with negative WNS (setup violations). Before leakage reduction can begin, timing must be brought to a feasible region. Phase 1 iterates over violating cells and:

1. Attempts Vt downgrade (HVT → NVT → LVT) to speed up the critical path
2. If Vt downgrade is insufficient, upsizes the cell to a stronger drive strength variant
3. Calls `compute_timing -full_update` after each swap to track WNS progress
4. Stops when WNS ≥ -1ps

Starting WNS values before Phase 1:

| Design | Starting WNS |
|--------|-------------|
| `usb_phy` | -21 ps |
| `aes_cipher_top` | -87 ps |
| `mpeg2_top` | -96 ps |

### Phase 2: Leakage Reduction

With timing feasible, Phase 2 greedily swaps non-critical cells toward HVT:

1. Sort all cells by sensitivity (leakage reduction / slack impact)
2. For each candidate cell, attempt Vt upgrade (LVT → NVT → HVT)
3. Accept the swap if WNS remains above the guard threshold (-3ps)
4. Reject and revert if the swap would violate the guard
5. Repeat until no further beneficial swaps remain

The -3ps guard provides margin against post-ECO timing degradation, which was measured at approximately 6-7ps in practice.

---

## Implementation

The script is written entirely in Aprisa Tcl and uses the following key API calls:

| API | Purpose |
|-----|---------|
| `ApWorstSlack clk` | Get current WNS |
| `ApCellSlack inst` | Get per-cell slack |
| `ApCellLeak inst` | Get per-cell leakage |
| `ApSwapCell inst cell` | Perform cell swap |
| `getNextVtUp / getNextVtDown` | Navigate Vt variants |
| `getNextSizeUp / getNextSizeDown` | Navigate size variants |
| `sortCells` | Sort instance list by attribute |
| `compute_timing -full_update` | Recompute timing after swap |

A custom `RecoverTimingFast` procedure was written to replace the built-in `RecoverTiming`, which iterated Vt in HVT-first order (wrong for timing recovery) and called `compute_timing` on every single attempt regardless of outcome. The custom version iterates LVT-first and batches timing updates, significantly reducing runtime.

---

## Results

### usb_phy

| Metric | Before | After |
|--------|--------|-------|
| Leakage Power | baseline | **9.137% reduction** |
| WNS | -21 ps | -9.870 ps |

The leakage reduction was achieved while keeping WNS within the acceptable timing penalty tier for grading. The -3ps guard threshold for Phase 2 was set based on empirical observation of ~6-7ps post-ECO timing degradation on this design.

---

## Key Implementation Challenges

**`getNextVtUp` / `getNextVtDown` behavior at limits** — these functions return the same cell name rather than a sentinel value when no further Vt variants exist. Explicit `ne ""` guards and same-name checks are required in while loops to avoid infinite loops.

**`ApCellSlack` takes only instance name** — no clock argument is accepted, unlike some other timing API calls. Passing a clock argument causes a silent error.

**`get_ta_paths -slack_lesser_than` is unsupported in Aprisa** — this flag exists in other EDA tools but not in Aprisa. Timing-critical cell filtering must be done by querying `ApCellSlack` per cell instead.

**`ap_cmds.tcl` must be sourced first** — the helper procs (`getNextVtUp`, `sortCells`, etc.) are defined in `ap_cmds.tcl`, which is loaded by `run_eco.tcl` before `size.tcl` is called. Running `size.tcl` standalone without sourcing `ap_cmds.tcl` first results in undefined procedure errors.

---

## Future Improvements

Extending the sensitivity function to account for fanout would improve swap ordering — high-fanout cells have disproportionate timing impact and should be penalized more heavily in the sensitivity calculation. Adding a hold-timing check after each swap would also prevent the optimization from inadvertently introducing hold violations on paths that gain slack during the leakage reduction phase.

---

*Part of CSE 241A — VLSI Integrated Circuits and Systems Design, UC San Diego, Spring 2026.*
