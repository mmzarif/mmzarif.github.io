---
layout: post
title: 8-bit Kogge-Stone Adder Design
description: A high-speed custom 8-bit adder utilizing Kogge-Stone architecture to optimize speed and power efficiency, achieving a 3.1 GHz clock frequency.
skills:
- Digital IC Design
- Cadence Virtuoso
- Static Logic Optimization
- Performance Benchmarking
main-image: /kogge_stone_full_schematic.png
---

## Project Overview

This project involved the design and simulation of a custom **8-bit Kogge-Stone adder** as part of the ECE 165: Digital Integrated Circuit Design course at UCSD. The primary challenge was to optimize speed while maintaining reasonable power levels.

### Architecture & Design

The Kogge-Stone architecture was selected because its tree-based structure allows for carry-look-ahead benefits without the high fan-in issues of large multi-bit adders.

**Logic Style:** We transitioned from dynamic logic to static logic to resolve weak drive strengths and clock synchronization issues.

**Performance:** Our static adder achieved a maximum clock frequency of **3.1 GHz** at a 1.1V power supply.

**Optimization:** We used the smallest transistor sizing possible to reduce capacitance and delay.
### Schematic Designs

#### 0. The AND, OR Gates

Before even designing the adder, we needed to create AND and OR gates with equal pull-up and pull-down strengths, and use them to build a group G block.

![group_g_and_or.png](/group_g_and_or.png)
*Figure 0: Schematic of the group G Block and AND and OR gates.*

#### 1. The PG Block

The first step in the architecture is translating input bit pairs into Propagate (P) and Generate (G) signals.

![PG Block Diagram](/pg_block.png)
*Figure 1: Schematic of the PG Block used for initial bit translation.*

#### 2. Full 8-bit Adder Hierarchy

The design uses three layers of logic to create all group-PG values efficiently for the 8-bit architecture.
![kogge_stone_full_schematic.png](/kogge_stone_full_schematic.png)
*Figure 2: Complete schematic of the Static 8-bit Kogge-Stone Adder.*

### Performance Benchmarking

The Kogge-Stone architecture outperformed our baseline Ripple-Carry Adder (RCA) by **800 MHz**.

| Parameter | Kogge-Stone Design | Ripple-Carry (Baseline) |
|:---|:---|:---|
| **Max Frequency (f_max)** | 3.1 GHz | 2.3 GHz |
| **Power @ 1.1V** | 451.8 µW | 318.42 µW |
| **Energy per Op** | 1.457 × 10⁻¹³ J | 1.39 × 10⁻¹³ J |
| **Critical Path Input** | A=00000001, B=01111111 | A=00000001, B=11111111 |
### Simulation Results

Validation was performed using transient waveform plots for over 10 input transitions to ensure correct timing behavior.

![transient_response.png](/transient_response.png)
*Figure 3: Max frequency simulation results at the critical path.*

---

**Team Members:** Luke Wilson, Siona Ahmed, and Mustahsin Zarif.

pplication of analog electronics principles while creating an engaging interactive display system.*
