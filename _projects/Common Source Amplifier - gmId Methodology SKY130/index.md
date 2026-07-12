---
layout: post
title: Common Source Amplifier Design using gm/Id Methodology and SKY130 PDK
date: 2026-07-09
description: Sized and simulated a common-source amplifier with PMOS active load in Cadence Virtuoso using the gm/Id methodology and the open-source SKY130 PDK, hitting a 25dB gain / 60MHz bandwidth target.
skills:
  - Analog IC Design
  - Cadence Virtuoso
  - gm/Id Methodology
  - SKY130 PDK
  - SPICE Simulation
  - Common Source Amplifier
  - Open-Source EDA
main-image: /schematic.png
---

## Project Overview

This project sizes and simulates a common-source amplifier with a PMOS active load and current-mirror bias in Cadence Virtuoso, using the **gm/Id methodology** and the **open-source SKYWATER SKY130 PDK**. The goal was to hit a 25dB DC gain / 60MHz unity-gain bandwidth spec into a 2pF load, starting from first-pass hand calculations and converging through simulation.

---

## Background: gm/Id vs. the Square Law

Traditionally, MOSFET saturation current is described by the golden square law:

`I_D = (1/2) * μ * C_ox * (W/L) * (V_GS − V_th)^2 * (1 + λV_DS)`

This holds up reasonably well for hand calculations, but as device dimensions shrink, real transistor behavior increasingly deviates from the square law — especially in weak and moderate inversion, where `gm/Id` is no longer simply `2/V_ov`. The mismatch between the square-law prediction and real device behavior is shown below.

{% include image-gallery.html images="real-device-vs-square-law.png, gmid-vs-square-law.png" height="320" %}

Because of this deviation, sizing directly from the square law becomes unreliable, so this design instead uses the **gm/Id methodology**: pre-characterizing the transistor's gm/Id, gm/gds, and transit frequency across bias points, then using those lookup tables to size devices with far fewer simulation iterations.

For the SKY130 PDK, the lookup tables were generated using [Open-LUT](http://www.open-lut.org), an interactive web-based gm/Id lookup tool. All designs use a minimum channel length of 150nm, above the 130nm process floor, to leave margin for manufacturing error.

{% include image-gallery.html images="open-lut-tables.png" height="380" %}

---

## Circuit: Common Source Amplifier with Active Load

The amplifier is an NMOS common-source stage with a PMOS active load, biased through a current mirror off an ideal current source. The output sees a 2pF capacitive load, which looks like an open circuit at DC.

{% include image-gallery.html images="schematic.png" height="380" %}

The DC gain is set by the total output resistance:

`A_v(0) = gm * R_out = gm / (gds_n + gds_p)`

and the bandwidth (dominant pole, single-stage) reduces to:

`BW = gm / (2π * C_L)`

---

## Design Goals & Parameter Selection

**Target specs:** 25dB DC gain, 60MHz unity-gain bandwidth, 2pF load, output biased at 0.9V DC.

To leave margin for second-order effects, the design target was pushed to 27dB (17.7 V/V) of gain and 66MHz of bandwidth.

From the bandwidth equation: `gm = 66MHz × 2π × 2pF ≈ 0.829mS`

Setting `gds_n = gds_p` for the target gain gives `gds ≈ 23.04µS`, i.e. a `gm/gds` ratio of about 36.

To keep the device in strong inversion (`V_ov > 150mV`), a `gm/Id` of 7 was chosen.

Using Open-LUT with these constraints — plugging in `V_ds`, `gm`, `gds`, `gm/gds`, and `gm/Id` — populates the full sizing table. For the PMOS, `Id` was matched to the NMOS value (118.5µA) since a 1:1 current mirror and ideal current source were assumed, so the table-provided `V_GS` wasn't needed directly.

{% include image-gallery.html images="nmos-parameters.png, pmos-parameters.png" height="320" %}

---

## Results: Iterating to Spec

**First pass — close, but short of spec.** The initial sizing met the 66MHz unity-gain bandwidth target, but the gain fell just short of 25dB. This was expected — the design used a smaller safety margin on gain than on bandwidth, but it's a good illustration of how close first-pass gm/Id sizing can get without any iteration.

{% include image-gallery.html images="bode-plot-not-met.png" height="380" %}

**Fix — upsize channel length.** Scaling up the NMOS channel length (while scaling the PMOS proportionally to preserve the W/L ratio and bias current) increases output resistance and gain. It does move the dominant pole slightly, since gate capacitance grows with L, but simulation showed enough margin to absorb it. The final sizing used a 4µm/265nm W/L ratio, which met both targets.

{% include image-gallery.html images="bode-plot-met.png" height="380" %}

---

## Conclusion

The square law is a reasonable starting point for understanding MOSFET behavior, but it isn't sufficient for *designing* real circuits — channel length modulation and inversion level both shift `gm/Id` away from the ideal `2/V_ov` relationship as devices scale down. The gm/Id methodology, backed by a pre-characterized lookup table like Open-LUT, gives a much more direct path from spec to transistor sizing — hitting the target in essentially one iteration rather than by trial-and-error simulation sweeps.

---

## References

[1] B. Murmann, "Systematic Design of Analog Circuits Using Pre-Computed Lookup Tables," Stanford University, California, 2016.

[2] S. A. Khalid, S. E. Maestre, and K. M. Madrid-Khalid, "Open-LUT: Interactive gm/ID Lookup Tables for MOSFET Sizing in Open-Source PDKs," *2025 10th International Conference on Integrated Circuits and Microsystems (ICICM)*, Hefei, China, 2025, pp. 344–353, doi: 10.1109/ICICM66614.2025.11316023.
