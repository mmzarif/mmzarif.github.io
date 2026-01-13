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

This project involved the design and simulation of a custom **8-bit Kogge-Stone adder** as part of the ECE 165: Digital Integrated Circuit Design course at UCSD[cite: 1, 2]. [cite_start]The primary challenge was to optimize speed while maintaining reasonable power levels.

### Architecture & Design
The Kogge-Stone architecture was selected because its tree-based structure allows for carry-look-ahead benefits without the high fan-in issues of large bit adder. 

* **Logic Style:** We transitioned from dynamic logic to **static logic** to resolve weak drive strengths and clock synchronization issues.
* **Performance:** Our static adder achieved a maximum clock frequency of **3.1 GHz** at a 1.1V power supply.
* **Optimization:** We used the smallest transistor sizing possible to reduce capacitance and delay.

### Schematic Designs

#### 0. The AND, OR gates
Before even designing the adder, we need to create AND and OR gates with equal pull up and pull down strengths, and use them to build a group G block.

![PG Block Diagram](/group_g_and_or.png)
*Figure 0: Schematic of the group G Block and AND and OR gates.*

#### 1. The PG Block
The first step in the architecture is translating input bit pairs into Propagate (P) and Generate (G) signals.

![PG Block Diagram](/pg_block.png)
*Figure 1: Schematic of the PG Block used for initial bit translation.*



#### 2. Full 8-bit Adder Hierarchy
The design uses three layers of logic to create all group-PG values efficiently for the 8-bit architecture.

![Static 8-bit Kogge-Stone Adder](/kogge_stone_full_schematic.png)
*Figure 2: Complete schematic of the Static 8-bit Kogge-Stone Adder.*



### Performance Benchmarking
The Kogge-Stone architecture outperformed our baseline Ripple-Carry Adder (RCA) by **800 MHz**.

| Parameter | Kogge-Stone Design | Ripple-Carry (Baseline) |
| :--- | :--- | :--- |
| **Max Frequency ($f_{max}$)** | 3.1 GHz | 2.3 GHz |
| **Power @ 1.1V** | $451.8\ \mu W$ | $318.42\ \mu W$ |
| **Energy per Op** | $1.457 \times 10^{-13}\ J$ | $1.39 \times 10^{-13}\ J$ |
| **Critical Path Input** | $A=00000001, B=01111111$ | $A=00000001, B=11111111$ |

### Simulation Results
Validation was performed using transient waveform plots for over 10 input transitions to ensure correct timing behavior.

![Simulation Waveform](/transient_response.png)
*Figure 3: Max frequency simulation results at the critical path.*

---
**Team Members:** Luke Wilson, Siona Ahmed, and Mustahsin Zarif.
