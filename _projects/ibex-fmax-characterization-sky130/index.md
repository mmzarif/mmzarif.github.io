---
layout: post
title: Estimating the Max Clock Frequency of a RISC-V Core (Ibex) on SKY130
date: 2026-08-06
description: A pre-PnR fmax characterization of the lowRISC Ibex RISC-V core synthesized with Cadence Genus and SKY130 PDK, using a binary search on the clock period and two clock-uncertainty assumptions to bracket where the design's real silicon maximum frequency will land.
skills:
  - Physical design
  - Static timing analysis
  - RTL synthesis
  - Cadence Genus
  - SKY130 PDK
  - RISC-V
  - Tcl scripting
  - PPA analysis
main-image: https://images.unsplash.com/photo-1518770660439-4636190af475?fm=jpg&q=80&w=1600&auto=format&fit=crop
---

## Project Overview

Here's a thing that tripped me up when I first started doing the physical design flow from scratch: anyone can write RTL at whatever clock frequency they want. Nothing in your SystemVerilog code stops you from running your design at an imaginary high speed. However, while the RTL may pass in simulation, the *actual* speed the design can run at in silicon is set by the technology node it's built on. Hence, the question I wanted to answer was this: **given a (random) design and a tech node, what is the maximum clock frequency the design can actually operate at?**

The design I was working on when this question hit me was the [Ibex RISC-V CPU core](https://github.com/lowRISC/ibex), which is an open source design. While there is a performance metric listed on the Github, it's not originally designed for the SKY130 PDK that I use. Obviously, I could not just guess the performance for my node by looking at the RTL, but I also did not want to run the whole PnR flow at multiple frequencies since that would take up a lot of time. As such, the fastest way to get an honest estimate was to **synthesize the RTL down to a gate-level netlist**, because once every logic gate is mapped to a real SKY130 standard cell, Genus can produce a timing report using the liberty file (which calculates cell delays using input slew and output capacitance). That timing report is what will tell us whether a given clock period is actually achievable.

So the plan was: **binary search** the clock period between a frequency I know will pass and one I know will fail, resynthesizing at each step, until the two converge on the fastest period that still meets timing. Additionally, because this is a pre-PnR estimate with no real clock tree yet, I ran the whole search twice with two different **clock uncertainty** assumptions — one optimistic (0.15ns) and one conservative (0.3ns) — to narrow down where the real number will land once jitter and skew show up for real post clock tree synthesis.

**Design:** `ibex_core` (lowRISC Ibex, [github.com/lowRISC/ibex](https://github.com/lowRISC/ibex)), "small" config.

**Process / library:** SKY130, Cadence SCL 9-track (`sky130_scl_9T_0.0.5`).

**Corner:** `tt_1.8_25` (typical-typical, 1.8V, 25°C).

**Tool:** Cadence Genus 23.12

---

## Background: what timing analysis is actually checking

Before the results make sense, it's worth being clear on what "meets timing" means, because the whole project hinges on one term in the equation.

Static timing analysis (STA) checks every path between two flip-flops and asks whether the signal arrives before the next clock edge needs it. The key quantity is **slack**:

`slack = Required Time − Arrival Time`

- **Arrival Time** works *forward* — it's the real accumulated delay through the gates and wires from the launching flop.
- **Required Time** works *backward* from the clock edge, minus setup time, minus output delay, minus **clock uncertainty**.

Positive slack means the path met timing; negative means it violated. The important detail is that **clock uncertainty only eats into Required Time — it never touches Arrival Time.** It doesn't make the circuit slower; it shrinks the budget the circuit is allowed to spend.

So what is clock uncertainty? Pre-PnR, there is no real clock tree yet, which means two effects can't be measured:

- **Jitter:** cycle-to-cycle wobble in the clock edge (temporal).
- **Skew:** the arrival-time difference between two flops caused by an imbalanced clock tree (spatial).

`clock_uncertainty` is a single placeholder number that stands in for both of these until a real clock tree exists. Which value we pick is a guess, and a big point of this project is to show how much that guess matters.

---

## Methodology

### **Binary search on the clock period** 
The search is bracketed by two bounds: one period known to **PASS** with comfortable slack, and one known to **FAIL**. I resynthesize at the midpoint, check worst-case slack, and use the result to replace whichever bound the midpoint beat — pass → new lower bound, fail → new upper bound — repeating until the bracket closes to below 0.1ns.

### **Full resynthesis at every step** 
Each candidate period is run through the entire `syn_generic → syn_map → syn_opt` flow, not just re-timed against an already-mapped netlist. This matters: gate sizing and technology mapping decisions depend on the target frequency, so re-timing an old netlist would give a dishonest answer. It's the technically correct approach, which is why it's slow; the full sweep took about two hours.

### **Two sweeps, one variable** 
The only thing that changed between the two runs was `clock_uncertainty`:

| Sweep | Clock uncertainty | Stance |
|---|---|---|
| A | 0.15 ns | Optimistic |
| B | 0.30 ns | Conservative |

The fixed Synonpsys Design Constraint (SDC) assumptions held constant across everything: 0.1ns clock transition, 1.5ns input/output delay, INVX4 driving cell, 0.5ns max transition, async reset treated as a false path.

---

## Results

### Sweep A (0.15ns uncertainty)

| Iteration | Period (ns) | Freq (MHz) | Worst Slack (ps) | Status |
|---|---|---|---|---|
| Lower bound | 2.0 | 500.00 | −3531 | FAIL |
| Upper bound | 20.0 | 50.00 | 6466 | PASS |
| 1 | 11.000 | 90.91 | 1 | PASS |
| 2 | 6.500 | 153.85 | 0 | PASS |
| 3 | 4.250 | 235.29 | −1355 | FAIL |
| 4 | 5.375 | 186.05 | −84 | FAIL |
| 5 | 5.938 | 168.41 | 0 | PASS |
| 6 | 5.656 | 176.80 | 0 | PASS |
| 7 | 5.515 | 181.32 | −3 | FAIL |
| 8 | 5.585 | 179.05 | 0 | PASS |

**Converged fmax (optimistic): 5.585ns → ≈179 MHz**

### Sweep B — conservative (0.3ns uncertainty)

For this sweep I reused bounds from prior data instead of retesting from scratch — the lower bound (5.556ns) came from the sensitivity check below, and the upper bound was Sweep A's 20.0ns result adjusted by the 150ps of extra uncertainty margin (well within its 6466ps of slack). Iteration 1 passing as expected confirmed the shortcut held.

| Iteration | Period (ns) | Freq (MHz) | Worst Slack (ps) | Status |
|---|---|---|---|---|
| Lower bound (reused) | 5.556 | 179.99 | −114 | FAIL |
| Upper bound (assumed) | 20.0 | 50.00 | >6316 (est.) | PASS |
| 1 | 12.778 | 78.26 | 10 | PASS |
| 2 | 9.167 | 109.09 | 0 | PASS |
| 3 | 7.361 | 135.85 | 0 | PASS |
| 4 | 6.458 | 154.85 | 0 | PASS |
| 5 | 6.007 | 166.47 | 0 | PASS |
| 6 | 5.781 | 172.98 | −53 | FAIL |
| 7 | 5.894 | 169.66 | 0 | PASS |
| 8 | 5.838 | 171.29 | −14 | FAIL |

**Converged fmax (conservative): 5.894ns → ≈169.7 MHz**

### The sensitivity check — the result that surprised me

To isolate the effect of clock uncertainty from the resynthesis noise you get when the period changes, I fixed the frequency at 180MHz and resynthesized twice, changing *only* the uncertainty:

| Uncertainty (ns) | Period (ns) | Freq (MHz) | Worst Slack (ps) | Status |
|---|---|---|---|---|
| 0.15 | 5.556 | 180.00 | 0 | PASS |
| 0.30 | 5.556 | 180.00 | −114 | FAIL |

Same design, same frequency, same everything except one placeholder number — and it flips from PASS to FAIL. That 114ps swing is the whole story of this project in one table.

### PPA at the conservative fmax (5.894ns / 169.7MHz)

| Area metric | Value |
|---|---|
| Cell count | 10,307 |
| Cell area | 138,353.9 |
| Net area (estimated) | 231,650.2 |
| **Total area** | **370,004.1** |

For reference, the initial uncalibrated run at an arbitrary 125MHz placeholder target synthesized to only 8,061 cells / 266,686 total area — roughly 28% fewer cells. Pushing the timing target tighter forces `syn_opt` to spend real area on larger drive-strength cells and more buffering to keep up.

*(Note on net vs. cell area: net area exceeds cell area here, but that's expected and not a real design property — net area is a statistical wireload-model estimate, since no routing exists yet.)*

Total power came out to **≈26.8mW**, split almost evenly between internal and switching power, with switching narrowly dominating — consistent with a design running near its frequency limit, where toggling activity is high.

### The critical path

```
Startpoint: rf_rdata_a_ecc_i[1]                       (register file read port A)
Endpoint:   ...prefetch_buffer_i_fetch_addr_q_reg[31] (scan-enabled flop)
Data Path:  3932 ps across ~28 logic stages
Slack:      0 ps (MET)
```

The critical path runs from a register-file read, through a long chain of NAND/OAI/adder cells, into the **instruction prefetch buffer's fetch-address increment logic** — functionally, a PC+4-style adder chain. It's a nice reminder that a core's speed limit often comes down to a chained-adder structure rather than the flashy datapath blocks you might expect.

---

## Analysis: why "0ps slack, MET" doesn't mean what it looks like

Look back at both sweep tables and you'll notice nearly every PASS sits at exactly **0ps slack**. That is not a coincidence, and it took me a while to understand why.

`syn_opt` is cost-driven — it stops spending area and power the *moment* a timing target is met. So whenever a target is achievable at all, the optimizer converges right up against the edge of it and stops. This has an important consequence: **slack alone cannot be read as a margin indicator.** A single run reporting "MET, 0ps slack" tells you almost nothing about how much headroom the design has, because the tool would have reported roughly 0ps slack at *any* achievable target you gave it.

The only way to find the true margin is to walk the target itself and watch where it breaks — which is the entire reason to do a sweep rather than trust one run. And once you accept that, the sensitivity result becomes alarming: the optimistic 179MHz number has essentially *zero* real margin against its own key unvalidated input, since a uncertainty change small enough to be a rounding error in a real clock tree wipes it out entirely.

---

## Caveats: this is pre-layout data

Every number here comes from synthesis alone. No placement, no clock tree, no routing. That shapes how much any of it should be trusted:

- **Wire delay is estimated**, not measured — it comes from wireload models based on fanout and design size, not real parasitics from a layout.
- **The clock is treated as ideal** — zero real insertion delay, zero real skew. `clock_uncertainty` is standing in for a clock tree that doesn't exist yet, and neither 0.15ns nor 0.3ns is "the" right answer; they're two points bracketing a plausible range.
- **Single corner only** (tt, 1.8V, 25°C) — no process, voltage, or temperature variation checked. Real signoff needs multi-corner analysis with the ff/ss corners.

Real CTS and routing will almost certainly *erode* these frequencies further, which is why I'd treat both numbers as **optimistic upper bounds**, not commitments.

---

## Conclusion

Under an optimistic clock-uncertainty assumption, Ibex ("small" config) synthesizes to a pre-layout fmax of about **179MHz**. Under a more conservative assumption for the same unmeasured quantity, that drops to about **169.7MHz** — a ~5% reduction driven *entirely* by one placeholder input, with no change to the design at all.

The practical takeaway for me: if I carry this design into Innovus floorplanning, the defensible starting constraint is the **conservative ~169.7MHz**, not the optimistic number and definitely not the arbitrary 150MHz placeholder I started with. It's a much more honest input to the next stage, and I now have the sweep data to justify it.

---

## A correction I made mid-project (and why I'm leaving it in)

An earlier version of this write-up claimed the near-fmax critical path ran through the fast multiplier's carry-save adder tree (`ibex_multdiv_fast`). That was wrong — and it's worth explaining how, because it's an easy trap.

That claim came from a timing report generated at a *different, looser* operating point (the initial 125MHz placeholder run), not at the converged fmax point. The `multdiv_fast` path is real, but it is not the bottleneck at the actual fmax; once I pulled the report at the converged 5.894ns point, the true critical path turned out to be the prefetch buffer's fetch-address logic. I'm calling this out explicitly rather than quietly fixing it, because I'd already talked about the multiplier path as if it were confirmed — a good reminder that a plausible-sounding result generated at one operating point should never be assumed to hold at another without checking the specific report it came from.

---

## Future Improvements

- **Multi-corner signoff.** Rerunning the sweep across ff/ss corners would replace the single-corner estimate with something closer to real derated timing.
- **Carry it into P&R.** Taking the conservative fmax into Innovus for floorplanning and CTS would let me compare this pre-layout estimate against real post-CTS timing — and finally replace the `clock_uncertainty` placeholder with a measured skew number.
