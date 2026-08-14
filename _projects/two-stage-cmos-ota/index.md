---
layout: post
title: Two-Stage CMOS Operational Transconductance Amplifier (OTA)
date: 2026-06-11
description: Designed a two-stage OTA (folded cascode + common source) for ECE 164 at UC San Diego with teammate Lucas Johnson, using gm/Id sizing and Miller compensation to hit 70dB gain and 60MHz+ bandwidth in Cadence Virtuoso.
skills:
  - Analog IC Design
  - Folded Cascode
  - Common Source
  - gm/Id Methodology
  - Miller Compensation
  - Cadence Virtuoso
  - SPICE Simulation
  - Team Project

main-image: /power-dissipation.png
---

## Project Overview

This was a two-person team project for **ECE 164 (Analog IC Design)** at UC San Diego, done with teammate **Lucas Johnson**. We designed a two-stage operational transconductance amplifier (OTA) by cascading a **folded cascode** first stage with a **common source** second stage, each loaded with an RC network, and compensated with Miller compensation. The design was sized by hand using the gm/Id methodology, then implemented and verified in Cadence Virtuoso.

---

## Design Outline

Any complex system can be broken into fundamental building blocks — our two-stage amplifier was a folded cascode (FC) cascaded with a common source (CS) stage, each an operational transconductance amplifier (OTA) with an RC load. After solving the half-circuits for each stage, we could size transistors, set transconductances, choose gm/Id ratios, and budget current.

Since FC gain scales as `(gm·ro)²` while CS gain scales only as `gm·ro`, we partitioned the target gain roughly **FC : CS ≈ 50dB : 20dB** — keeping in mind that a low CS gain could hurt phase margin and cost extra power later.

The large resistive network inside the FC doesn't contribute to amplification, so we lowered the current through the M4 → M5 → M6 branches to raise `Ro1` (and therefore gain) without spending extra power. M3 and M9, the actual amplifying transistors, needed higher current to boost gain, so we budgeted current for them separately and ratioed the bias network 1:1 (W/L) against the main branches.

For M3 and M9 we targeted a **gm/Id > 12** (max 13.3, to stay solidly in strong inversion). Starting from a guessed 50µA, we sized the FC with `Lp,min = 600n`, `Ln,min = 450n`, and the CS with `Ln,min = 180n` (so the "magic battery" length worked out to `3×Lp,min = 4×Ln,min = 1.8µm`). NMOS current mirrors used 1µm lengths to avoid second-order distortion of the mirrored currents; PMOS mirrors were tested at 600nm and matched well.

{% include image-gallery.html images="schematic-page1-left.png, schematic-page1-right.png" height="480" %}

{% include image-gallery.html images="schematic-page2-left.png, schematic-page2-right.png" height="480" %}

---

## Design Parameter Calculations

We started from the phase margin equation, targeting **71.5° phase margin** to intentionally overdesign the compensation:

`PM = 90 − tan⁻¹((gm3/gm9)/(CL/Cc))`, with `gm3 = (1/5)·gm9`

Solving `71.5 = 90 − tan⁻¹((1/5)(2pF/Cc))` gives **Cc ≈ 1.195pF**.

From `GBW = gm3/Cc`, targeting 60MHz gives **gm3 ≈ 0.6mS**, and from the `gm3 = (1/5)gm9` assumption, **gm9 ≈ 3mS**.

### Folded Cascode Sizing

Using the gm/Id methodology and a provided current-density MATLAB script, we chose **gm/Id = 12** for M3 to get maximum transconductance at the lowest current while still clearing the 150mV minimum overdrive voltage. From `ID3 = (gm3·Vov)/2`, we got **ID3 = 50µA**, then found width from `W = ID/(ID/W)` off the script. The rest of the FC was sized the same way, assuming `ID4 = 1.5×ID3 = 75µA`.

- **M1, M2, M4, M7** (tail transistors): gm/Id = 9, for low current draw while staying close to deep saturation for good current mirroring.
- **M5**: started at gm/Id = 9.
- **M6**: `VDS6,min = Vov6 + Vov7`, and `VDS6 = Vov + Vtn`. Assuming Vtn ≈ 310mV, we rounded Vov6a up to 350mV, giving **gm/Id = 5.7**.

We originally targeted **55dB** of gain from the FC stage alone.

### Common Source Sizing

The CS stage targeted **25dB** (overdesigned on purpose). M9 was sized for 50µA at **gm/Id = 13.3**; M8 originally used an M-factor of 1 to bring 50µA into the CS stage.

### Bias Circuit Sizing

The bias network mirrored 50µA throughout before the amplifying stages' M-factors were adjusted. Current mirrors used a **4µm/1µm** ratio to avoid second-order effects, with `Mb1 = 16µm` sized so that `gm = 1/Rbias`. We swept `R` in the full schematic until 50µA flowed through the bias resistor.

An important realization: even after re-sizing the FC and CS, **VCAS stays the same as long as gm/Id and Vov are held constant** — which let us re-tune the circuit without it breaking.

{% include image-gallery.html images="magic-battery-pmos.png, magic-battery-nmos.png" height="380" %}

---

## Transistor Sizing

After implementing the folded cascode in Cadence, we re-tuned widths to push every transistor into saturation at the desired branch currents — raising M1's width to get ~100µA through M1/M2, then realizing that *lowering* branch current would raise `Rout` (and gain) in the FC, so we cut the widths of M4–M6 by 8× (same lengths), dropping to ~7µA through those branches.

That re-sizing shifted the DC midpoint between the FC output and the CS input, which pushed the CS into saturation. We swept M9's width to keep the output DC voltage above 400mV (inside our common-mode output range), then swept M8's width to hit **70dB** gain — landing on an M-factor of 0.65 and only ~32µA through the CS stage, well under the original current budget.

| Transistor | Initial W | Initial L | Revised W | Revised L |
|---|---|---|---|---|
| M1 | 50µm | 600nm | 76µm | 600nm |
| M2 | 71µm | 1.8µm | 54.5µm | 450nm |
| M3 | 50µm | 600nm | 38µm | 600nm |
| M4 | 38µm | 600nm | 4.75µm | 600nm |
| M5 | 27µm | 450nm | 3.4µm | 450nm |
| M6 | 2.33µm | 450nm | 290nm | 450nm |
| M7 | 8.67µm | 450nm | 5.63µm | 450nm |
| M8 | 38µm | 600nm | 14.11µm | 340nm |
| M9 | 3.25µm | 450nm | 3.6µm | 450nm |

---

## Power Dissipation

With calculated currents through every branch of the bias network, FC, and CS:

`P = ((4 × 50µA) + (2 × 100µA) + (2 × 6.25µA) + 32µA) × 1.8V ≈ 800µW`

Our actual simulated power came out to **960µW** — the gap is likely from discrepancies between hand calculations and the SPICE models.

{% include image-gallery.html images="power-dissipation.png" height="380" %}

---

## Adjusting Poles & Final Results

After hitting our power and gain targets, we adjusted the second pole using our earlier Cc value and by sweeping Rc to **13.5kΩ**. Final simulated performance overachieved spec:

| Metric | Value |
|---|---|
| Open-loop gain (A0) | 72.16 dB |
| Unity-gain bandwidth (UGBW) | 86.76 MHz |
| Gain-bandwidth product (GBW) | 35.68 MHz |
| Phase margin | 74.04° |
| Gain margin | 14.03 dB |
| 3dB bandwidth | 8.782 kHz |
| Power | 960.1 µW |

{% include image-gallery.html images="bode-gain.png, bode-phase.png" height="420" %}

---

## Transient Analysis & Output Common-Mode Range

We also verified transient behavior at the input and output nodes, and swept output common-mode range (OCMR) against input common-mode range (ICMR) to confirm the amplifier stayed in its valid operating region across the intended input swing.

{% include image-gallery.html images="transient-vin.png, transient-vout.png" height="380" %}

{% include image-gallery.html images="ocmr-vs-icmr.png" height="420" %}

---

## Comments and Conclusions

The first real challenge was setting branch currents — we knew our overall current budget and target gm/Id ratios, but still had to choose either `gm` or `Id` and iterate from there, which took real trial and error.

Second, for the CS stage, `Av2 = gm2(ro,up || ro,down)`. Without knowing λ exactly, we saw firsthand that pushing current up to raise `gm` also drops `ro`, so gain doesn't scale linearly with current the way a naive read of the equation suggests.

Third — and this one cost us a re-design — we originally sized the CS stage around a gate voltage that gave our desired current in isolation. Once we connected the FC stage ahead of it, we realized the CS's gate voltage was actually *set by* the FC's output, not the other way around, so the CS had to be re-sized around that constraint.

**Future improvements:** add a startup circuit to the constant-gm bias network so it can never latch into an off-state, and make fuller use of the M-factor (up to 20) to shrink the bias branches as small as possible.
