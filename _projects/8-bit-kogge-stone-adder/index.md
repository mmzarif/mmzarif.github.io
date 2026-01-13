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

[cite_start]This project involved the design and simulation of a custom **8-bit Kogge-Stone adder** as part of the ECE 165: Digital Integrated Circuit Design course at UCSD[cite: 1, 2]. [cite_start]The primary challenge was to optimize speed while maintaining reasonable power levels[cite: 8].

### Architecture & Design
[cite_start]The Kogge-Stone architecture was selected because its tree-based structure allows for carry-look-ahead benefits without the high fan-in issues of large bit adders[cite: 10]. 

* [cite_start]**Logic Style:** We transitioned from dynamic logic to **static logic** to resolve weak drive strengths and clock synchronization issues[cite: 20, 31, 35].
* [cite_start]**Performance:** Our static adder achieved a maximum clock frequency of **3.1 GHz** at a 1.1V power supply[cite: 39].
* [cite_start]**Optimization:** We used the smallest transistor sizing possible to reduce capacitance and delay[cite: 26, 27].

### Schematic Designs

#### 0. The AND, OR gates
[cite_start]Before even designing the adder, we need to create AND and OR gates with equal pull up and pull down strengths, and use them to build a group G block[cite: 21].

![PG Block Diagram](/group_g_and_or.png)
[cite_start]*Figure 0: Schematic of the groujp G Block and AND and OR gates.*

#### 1. The PG Block
[cite_start]The first step in the architecture is translating input bit pairs into Propagate (P) and Generate (G) signals[cite: 22].

![PG Block Diagram](/pg_block.png)
[cite_start]*Figure 1: Schematic of the PG Block used for initial bit translation.*



#### 2. Full 8-bit Adder Hierarchy
[cite_start]The design uses three layers of logic to create all group-PG values efficiently for the 8-bit architecture[cite: 24].

![Static 8-bit Kogge-Stone Adder](/kogge_stone_full_schematic.png)
[cite_start]*Figure 2: Complete schematic of the Static 8-bit Kogge-Stone Adder.*



### Performance Benchmarking
[cite_start]The Kogge-Stone architecture outperformed our baseline Ripple-Carry Adder (RCA) by **800 MHz**[cite: 39].

| Parameter | [cite_start]Kogge-Stone Design [cite: 99] | [cite_start]Ripple-Carry (Baseline) [cite: 99] |
| :--- | :--- | :--- |
| **Max Frequency ($f_{max}$)** | 3.1 GHz | 2.3 GHz |
| **Power @ 1.1V** | $451.8\ \mu W$ | $318.42\ \mu W$ |
| **Energy per Op** | $1.457 \times 10^{-13}\ J$ | $1.39 \times 10^{-13}\ J$ |
| **Critical Path Input** | $A=00000001, B=01111111$ | $A=00000001, B=11111111$ |

### Simulation Results
[cite_start]Validation was performed using transient waveform plots for over 10 input transitions to ensure correct timing behavior[cite: 40].

![Simulation Waveform](/transient_response.png)
[cite_start]*Figure 3: Max frequency simulation results at the critical path.*

---
[cite_start]**Team Members:** Luke Wilson, Siona Ahmed, and Mustahsin Zarif[cite: 42, 43, 44].
