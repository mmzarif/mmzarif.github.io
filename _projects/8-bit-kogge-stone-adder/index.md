---
layout: post
title: 8-bit Kogge-Stone Adder Design
description: A high-speed custom 8-bit adder utilizing Kogge-Stone architecture to optimize speed and power efficiency, achieving a 3.1 GHz clock frequency.
skills:
- Digital IC Design
- Cadence Virtuoso
- Static Logic Optimization
- Performance Benchmarking
main-image: /project-adder.webp
---

## Project Overview

[cite_start]This project involved the design and simulation of a custom **8-bit Kogge-Stone adder** as part of the ECE 165: Digital Integrated Circuit Design course at UCSD[cite: 1, 2]. [cite_start]The goal was to optimize for high-speed arithmetic performance while maintaining reasonable power consumption[cite: 8].

### Architecture & Design
[cite_start]The Kogge-Stone architecture was selected over a standard Ripple-Carry Adder (RCA) because it utilizes a tree-based structure to calculate carries in $O(\log_2 N)$ time[cite: 18]. 

* [cite_start]**Logic Style:** While dynamic logic was initially attempted, the final design utilized **static logic** to overcome drive strength issues and synchronization errors[cite: 20, 31, 35].
* [cite_start]**Performance:** The final design achieved a maximum clock frequency of **3.1 GHz** at 1.1V, outperforming our previous RCA implementation by 800 MHz[cite: 39].
* [cite_start]**Power Consumption:** The adder required $451.8\ \mu W$ of power during operation at peak frequency[cite: 41].


### Key Contributions
* [cite_start]**Luke Wilson:** Team lead; developed both RCA and Kogge-Stone designs in Cadence[cite: 42].
* [cite_start]**Siona Ahmed:** Simulated the circuits and co-designed individual logic blocks[cite: 43].
* [cite_start]**Mustahsin Zarif:** Provided troubleshooting support and led the technical report writing[cite: 44].

### Technical Specifications
| Parameter | Kogge-Stone Design | Ripple-Carry (Baseline) |
| :--- | :--- | :--- |
| **Max Frequency ($f_{max}$)** | [cite_start]3.1 GHz [cite: 99] | [cite_start]2.3 GHz [cite: 99] |
| **Power @ 1.1V** | [cite_start]$451.8\ \mu W$ [cite: 99] | [cite_start]$318.42\ \mu W$ [cite: 99] |
| **Energy per Op** | [cite_start]$1.457 \times 10^{-13}\ J$ [cite: 99] | [cite_start]$1.39 \times 10^{-13}\ J$ [cite: 99] |
| **Architecture** | [cite_start]Parallel Prefix (Tree) [cite: 11] | [cite_start]Sequential [cite: 14] |

---
[cite_start]*This project was completed for ECE 165 at UCSD, Spring 2025[cite: 50].*
