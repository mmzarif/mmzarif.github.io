---
name: ML-Based Via Count Prediction for Physical Design
tools: [Python, scikit-learn, Siemens Aprisa, TSMC 65GP]
image: https://mmzarif.github.io/_projects/ml-via-count/cover.png
description: A machine learning model that predicts post-routing via counts per net from post-CTS placement features, built using a 273,212-sample dataset generated across a 5×4 grid of floorplan utilizations and clock periods.
---

# ML-Based Via Count Prediction for Physical Design

## Project Overview

Routing is the most runtime-intensive step in physical design. This project trains a regression model to predict the number of vias a net will require after detailed routing, using only features available at the post-CTS stage — before routing ever runs.

**Design:** `aes_cipher_top` (TSMC 65GP, 65nm)  
**Tool:** Siemens Aprisa (P&R)  
**Scope:** Per-net via count prediction for nets with 2–50 pins

---

## Skills Used

Machine Learning · Physical Design · EDA Automation · Python · DEF/SDC/Liberty · Siemens Aprisa · scikit-learn · Bash Scripting

---

## Motivation

Running full P&R takes hours per design point. A pre-routing predictor trained on post-CTS data allows designers to:

- Estimate via counts before committing to routing
- Identify potentially congested nets early
- Sweep design parameters (utilization, timing) without running full routing each time

---

## Training Data Generation

The provided example data covered only 1.60 ns at three utilization levels. Since hidden test cases vary both utilization (60–80%) and clock period (1.3–1.6 ns), a single clock period is insufficient for the model to learn the effect of timing constraints on via count.

We generated a **5×4 grid** of Aprisa P&R runs:

|  | 1.30 ns | 1.40 ns | 1.50 ns | 1.60 ns |
|--|:-------:|:-------:|:-------:|:-------:|
| **60%** | ✓ | ✓ | ✓ | ✓ |
| **65%** | ✓ | ✓ | ✓ | ✓ |
| **70%** | ✓ | ✓ | ✓ | ✓ |
| **75%** | ✓ | ✓ | ✓ | ✓ |
| **80%** | ✓ | ✓ | ✓ | ✓ |

**Total: 273,212 training samples** (~13,600 nets per case)

Runs were parallelized using a semaphore-based bash job pool (`MAX_JOBS=4`), reducing total data generation time by ~4×. Each run executes in an isolated `/tmp` directory so concurrent Aprisa instances never interfere with each other.

---

## Feature Extraction

Features are extracted from the post-CTS DEF file. The parser handles gzip-compressed DEF files automatically (Aprisa outputs compressed DEFs despite a plain `.def` extension).

### Raw Features

| Feature | Description |
|---------|-------------|
| `util` | Initial floorplan utilization |
| `cp` | Target clock period (ns) |
| `bboxArea` | Net bounding box area (µm²) |
| `bboxAr` | Net bounding box aspect ratio |
| `numPins` | Number of pins on the net |

### Engineered Features

Eight additional features expose non-linear relationships:

| Feature | Rationale |
|---------|-----------|
| `bboxPerim` | Wire length scales with perimeter, not area |
| `logBboxArea` | Via count is sub-linear in area |
| `pinDensity` | Crowded nets need more layer changes |
| `numPins²` | High-fanout cost is super-linear |
| `areaPins` | Large high-fanout nets are disproportionately expensive |
| `util × cp` | Joint effect of density and timing pressure |
| `√bboxArea` | Direct wire length proxy |
| `bboxAr × numPins` | Elongated multi-sink nets force many layer transitions |

---

## Model Selection — Ablation Study

Six model variants were evaluated using 5-fold cross-validation on the full 273,212-sample training set.

| Model | CV RMSE |
|-------|:-------:|
| LinearRegression (raw) | 4.14 |
| LinearRegression (engineered) | 3.94 |
| Ridge (engineered) | 3.94 |
| GradientBoosting (raw, depth=3) | 3.85 |
| **GradientBoosting (engineered, depth=4)** | **3.84** ✓ |
| RandomForest (engineered) | 3.95 |

**Selected:** `GradientBoostingRegressor`  
**Hyperparameters:** `n_estimators=400, max_depth=4, learning_rate=0.08, subsample=0.8, min_samples_leaf=20`

Key findings:
- Feature engineering reduced RMSE by 4.7% on linear models, confirming non-linear relationships between geometry and via count
- GradientBoosting outperformed RandomForest because sequential boosting better corrects systematic residuals in the via count distribution
- Engineered features provided complementary signal even for tree-based models

---

## Results

| Dataset | Utilization | Clock Period | RMSE |
|---------|:-----------:|:------------:|:----:|
| Training | 60% | 1.60 ns | 5.25 |
| Training | 70% | 1.60 ns | 6.09 |
| Training | 80% | 1.60 ns | 5.52 |
| Open Test | 65% | 1.60 ns | 4.94 |
| Open Test | 75% | 1.60 ns | 4.33 |

The open test cases (65%, 75%) achieve lower RMSE than training cases, confirming the model generalizes well to unseen utilization values.

---

## Pipeline

```
post-CTS DEF  ──► generate_features.py ──► features.csv ──┐
                                                           ├──► training.csv ──► ml_train.py ──► model.pkl
post-routed DEF ► generate_label.py ───► label.csv ───────┘                                         │
                                                                                                     ▼
new post-CTS DEF ──► generate_features.py ──► ml_inference.py ──────────────────────► inference.csv
```
