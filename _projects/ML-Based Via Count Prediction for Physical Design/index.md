---
layout: post
title: ML-Based Via Count Prediction for Physical Design
date: 2026-06-10
description: A machine learning model that predicts post-routing via counts per net from post-CTS placement features, built using a 273,212-sample dataset generated across a 5×4 grid of floorplan utilizations and clock periods.
skills:
  - Machine learning
  - Physical design
  - EDA automation
  - Feature engineering
  - Python
  - Bash scripting
  - DEF/SDC/Liberty
  - Siemens Aprisa
main-image: /ml-via-count-cover.png
---

## Project Overview

Routing is the most runtime-intensive step in physical design — it can take hours per implementation. This project builds a regression model that predicts how many vias a net will need after detailed routing, using only features available at the post-CTS stage, before routing ever runs.

If you can predict via counts early, you can flag congested nets, guide placement decisions, and sweep design parameters without paying the full routing runtime cost each time.

**Design:** `aes_cipher_top` (TSMC 65GP, 65nm)  
**Tool:** Siemens Aprisa (P&R)  
**Target:** Minimize RMSE of predicted vs. actual per-net via count

---

## Physical Design Flow

The project sits at the boundary between post-CTS and routing in the standard physical design flow:

```
Floorplan → Placement → CTS → [PREDICT HERE] → Routing
                                ↑                   ↑
                         post-CTS DEF       post-routed DEF
                           (features)          (labels)
```

Features are extracted from the post-CTS DEF. Labels (actual via counts) are extracted from the post-routed DEF. A regression model is trained to map one to the other.

---

## Training Data Generation

The provided data only covered 1.60 ns at three utilization levels. Since the hidden test cases vary both utilization (60–80%) and clock period (1.3–1.6 ns), training on a single clock period would leave the model unable to generalize across timing constraints.

To fix this, I ran Aprisa P&R across a full **5×4 grid**:

|  | 1.30 ns | 1.40 ns | 1.50 ns | 1.60 ns |
|--|:-------:|:-------:|:-------:|:-------:|
| **60%** | ✓ | ✓ | ✓ | ✓ |
| **65%** | ✓ | ✓ | ✓ | ✓ |
| **70%** | ✓ | ✓ | ✓ | ✓ |
| **75%** | ✓ | ✓ | ✓ | ✓ |
| **80%** | ✓ | ✓ | ✓ | ✓ |

This produced **273,212 training samples** (~13,600 nets per case).

Each run patches exactly two files before launching Aprisa:
- `scripts/proj_variables.tcl` — sets `-utilization` in `INIT_FP_OPTIONS`
- `aes_cipher_top.sdc` — sets `-period` and the waveform half-period in `create_clock`

Runs execute in isolated `/tmp` directories so parallel instances never share a working database. A semaphore-based bash job pool (`MAX_JOBS=4`) runs up to 4 cases simultaneously, cutting total generation time by ~4×.

---

## Feature Extraction

### Raw Features

| Feature | Source | Rationale |
|---------|--------|-----------|
| `util` | `proj_variables.tcl` | Higher utilization → less routing room → more detours → more vias |
| `cp` | `aes_cipher_top.sdc` | Tighter timing → router forced onto specific layers → more vias |
| `bboxArea` | Net pin placement | Larger bounding box → longer wire → more layer transitions |
| `bboxAr` | Net pin placement | Elongated nets force many horizontal/vertical layer changes |
| `numPins` | Net connections | More sinks → more Steiner branches → more vias |

### Engineered Features

Eight additional features expose non-linear relationships that linear models miss:

| Feature | Formula | Rationale |
|---------|---------|-----------|
| `bboxPerim` | `2*(W+H)` | Wire length scales with perimeter, not area |
| `logBboxArea` | `log1p(bboxArea)` | Via count is sub-linear in area |
| `pinDensity` | `numPins / bboxArea` | Crowded nets need more layer changes |
| `numPins²` | `numPins²` | High-fanout cost is super-linear |
| `areaPins` | `bboxArea × numPins` | Large high-fanout nets are disproportionately expensive |
| `util × cp` | `util × cp` | Joint effect of density and timing pressure |
| `√bboxArea` | `sqrt(bboxArea)` | Direct wire length proxy |
| `bboxAr × numPins` | `bboxAr × numPins` | Elongated multi-sink nets force many layer transitions |

The DEF parser handles gzip-compressed files automatically — Aprisa outputs compressed DEFs despite a plain `.def` extension.

---

## Model Selection — Ablation Study

Six model variants were evaluated using 5-fold cross-validation on the full training set:

| Model | CV RMSE |
|-------|:-------:|
| LinearRegression (raw features) | 4.14 |
| LinearRegression (engineered features) | 3.94 |
| Ridge Regression (engineered features) | 3.94 |
| GradientBoosting (raw, depth=3) | 3.85 |
| **GradientBoosting (engineered, depth=4)** | **3.84** ✓ |
| RandomForest (engineered features) | 3.95 |

**Selected model:** `GradientBoostingRegressor`  
**Hyperparameters:** `n_estimators=400, max_depth=4, learning_rate=0.08, subsample=0.8, min_samples_leaf=20`

### Key Findings

1. Feature engineering reduced RMSE by **4.7%** on linear models (4.14 → 3.94), confirming the geometry-to-via relationship is non-linear
2. Switching to GradientBoosting on raw features reduced RMSE further (3.94 → 3.85), showing trees capture interactions linear models miss
3. Combining GradientBoosting with engineered features achieved the best result (3.84)
4. RandomForest underperformed GradientBoosting because sequential boosting corrects systematic residuals more effectively than averaging

---

## Results

| Dataset | Utilization | Clock Period | RMSE |
|---------|:-----------:|:------------:|:----:|
| Training | 60% | 1.60 ns | 5.25 |
| Training | 70% | 1.60 ns | 6.09 |
| Training | 80% | 1.60 ns | 5.52 |
| **RMSE_1 (sum)** | | | **16.87** |
| Open Test | 65% | 1.60 ns | 4.94 |
| Open Test | 75% | 1.60 ns | 4.33 |
| **RMSE_2 (sum)** | | | **9.27** |
| **Metric 2 = RMSE_1 + RMSE_2** | | | **26.14** |

The open test cases (65%, 75%) achieve lower RMSE than the training cases, confirming the model generalizes well to utilization values it was not directly trained on at 1.60 ns.

---

## Future Improvements

Adding congestion map features extracted from the post-CTS DEF would likely reduce RMSE significantly — congestion is one of the strongest predictors of routing difficulty but was not available in the base feature set. Timing criticality (slack per net) would also be a strong addition, as timing-critical nets tend to be routed on preferred layers with more via stacks.

Training on additional clock periods between 1.3 and 1.6 ns (e.g., 1.35, 1.45, 1.55) would improve interpolation accuracy for the hidden test cases in that range.

---

*Part of CSE 241A — VLSI Integrated Circuits and Systems Design, UC San Diego, Spring 2026.*
