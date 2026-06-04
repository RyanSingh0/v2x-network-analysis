# V2X Wireless Network Performance Analysis

> **METCS 544 · Boston University**

![R](https://img.shields.io/badge/R-4.3-blue)
![R²](https://img.shields.io/badge/Model%20R%C2%B2-0.869-brightgreen)
![Dataset](https://img.shields.io/badge/Dataset-TiHAN--V2X-lightgrey)

---

## Overview

Comprehensive statistical analysis of **Vehicle-to-Everything (V2X) wireless communication performance** using the TiHAN-V2X dataset collected in Hyderabad, India — covering both Vehicle-to-Vehicle (V2V) and Vehicle-to-Infrastructure (V2I) communication scenarios.

**Research questions:**
1. What are the primary factors affecting V2X network throughput and latency?
2. Do V2V and V2I performance differ significantly, and how?
3. Can regression models predict throughput from measurable network features?
4. How much does scenario configuration vs. signal quality drive performance?

---

## Dataset

| Property | Value |
|----------|-------|
| Source | TiHAN-V2X (2024) · IEEE DataPort · DOI: 10.21227/f2kd-9g03 |
| V2V observations | **10,252** across 7 scenarios (S1–S7) |
| V2I observations | **2,956** across 3 scenarios (V2I-S1 to S3) |
| Total | **13,208 observations** |
| Protocol | C-V2X PC5 Mode 4 · 5.9 GHz sidelink · 10 MHz bandwidth · QPSK MCS5 |

**Key metrics per observation:** Throughput (bps), Latency (ms), RSSI (dBm), SNR (dB), Packet Error Rate (PER), Distance (m)

---

## Descriptive Statistics (SOCS)

### Latency
| Link | Mean | Median | SD |
|------|------|--------|----|
| V2V | **0.577 ms** | 0.609 ms | 0.13 ms |
| V2I | **0.624 ms** | 0.640 ms | — |

Both well below the 1 ms URLLC target for vehicular safety. No extreme outliers.

### Throughput
- **V2V:** 1,500–9,800 bps · **Bimodal** (10 Hz scenarios: ~2,000–3,000 bps; 20 Hz scenarios: ~6,000–7,000 bps)
- **V2I:** 1,500–2,900 bps · **Unimodal** (constant 10 Hz rate)

### Signal Quality
| Metric | V2V | V2I |
|--------|-----|-----|
| RSSI mean | ~−306 dBm | ~−257 dBm |
| SNR mean | ~41 dB (range 1–160 dB) | ~90 dB (range 69.5–122.1 dB) |
| PER median | ~5.4% | ~4.0% |

**Key correlation:** Distance vs SNR: r ≈ **−0.81** (V2V) — strong inverse relationship, aligns with path loss models.

---

## Statistical Inference

| Test | Result | Finding |
|------|--------|---------|
| One-sample t-test (latency vs 1ms) | p < 0.0001 | Latency significantly below 1ms URLLC target |
| Two-sample t-test (V2V vs V2I throughput) | p < 0.01 | V2V throughput **14% higher** (driven by 20 Hz scenarios) |
| Two-sample t-test (V2V vs V2I SNR) | p < 0.0001 | V2I SNR 90 dB vs V2V 41 dB — infrastructure link quality far superior |
| Chi-square (Scenario vs Throughput category) | Cramér's V = **0.935** | Near-deterministic relationship — scenario drives throughput |
| One-way ANOVA (Throughput across V2V scenarios) | F ≈ 2144, p < 0.0001 | Scenarios differ massively in throughput |
| ANOVA (SNR + PER across scenarios) | p < 0.0001 | Not just throughput — all performance metrics vary by scenario |

---

## Regression Models

### Multiple Linear Regression — Throughput Prediction

Starting from simple models (SNR, distance, PER individually — all significant but low R²), iteratively added features:

1. Added scenario as categorical predictor → major R² improvement
2. Added polynomial distance term (non-linear relationship confirmed)
3. Added cross-correlation features (threshold ≥ 0.2 applied)

**Final model:**
```
Throughput = β₀ + β₁·Scenario + β₂·Distance + β₃·Distance² + β₄·SNR + ... + ε
```

| Metric | Value |
|--------|-------|
| **Adjusted R²** | **0.869** |
| All predictors | Significant |

The model explains **86.9% of throughput variance** — strong for a real-world wireless dataset.

### Decision Tree (Non-linear exploration)

Built to classify V2V throughput as High/Low using only SNR, distance, and PER (excluding scenario label).

- Primary split: SNR threshold ~21 dB separates low-throughput links
- Test accuracy: **64%** — moderate, constrained by excluding scenario information
- Confirms that SNR < 21 dB reliably predicts poor link quality

---

## Key Findings

1. **Scenario configuration dominates throughput** (Cramér's V = 0.935) — more than signal quality. Message transmission rate (10 Hz vs 20 Hz) is the dominant driver, not SNR or distance.

2. **Distance strongly affects signal quality** (r = −0.81) but has only weak direct effect on throughput until SNR falls below critical thresholds (~10 dB).

3. **V2I consistently outperforms V2V** in SNR and PER, but V2V achieves higher throughput in high-frequency scenarios.

4. **Ultra-low latency achieved** (0.577–0.624 ms) — both link types satisfy URLLC requirements for vehicular safety applications.

5. **Multiple regression R² = 0.869** — practically deployable model for throughput prediction in V2X network planning.

---

## How to Run

```r
install.packages(c("ggplot2", "dplyr", "car", "rpart"))
source("METCS544_Project.R")
```

> **Data:** See data_note.md for TiHAN-V2X dataset download. Requires free IEEE DataPort account.

---

**Aryan Meena** · Boston University · CS544
