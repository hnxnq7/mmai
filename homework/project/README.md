# Vision vs. Protocol Effects in EEG2Video
## Baseline Audit and Temporal Alignment on SEED-DV

**Rachel Li, Emma Wang, Winston Qian** — MIT · MAS.S60 / 6.S985

**Team repo:** [winstonqian/EEG2Video](https://github.com/winstonqian/EEG2Video)

---

## Overview

EEG-to-video decoding benchmarks like SEED-DV risk confounding genuine visual perception with protocol-induced cognitive artifacts. In the SEED-DV dataset, each concept is presented as a **5-clip run** (cue + 5 consecutive same-concept clips), which may cause classifiers to exploit run-position-dependent neural signals (adaptation, expectation, fatigue) rather than clip-specific visual content.

This project:
1. **Audits** the EEG2Video baseline for data leakage and run-position shortcuts
2. **Establishes** a clean, artifact-free within-subject classification baseline
3. **Proposes** chunk-level latency-aware EEG–video contrastive alignment to enforce genuine dynamic tracking

---

## The Core Question

> Are current EEG-to-video models genuinely tracking within-clip temporal dynamics, or are they relying on static semantics and slow protocol-driven neural drift?

---

## Midterm Results

### Baseline Classification (40-class concept, chance = 2.5%)

| Condition | Top-1 (%) | Top-5 (%) |
|---|---|---|
| DE | 4.37 ± 2.64 | 17.17 ± 5.44 |
| DE_no_data_leak | 4.09 ± 2.62 | 16.84 ± 5.19 |
| DE_no_early_stop | 4.07 ± 2.55 | 16.99 ± 5.34 |
| DE_run2 (repro check) | 4.27 ± 2.57 | 17.19 ± 5.04 |
| PSD | 4.15 ± 2.60 | 17.15 ± 4.89 |
| PSD_no_data_leak | 4.29 ± 2.29 | 17.13 ± 4.87 |
| PSD_no_early_stop | 4.33 ± 2.34 | 16.98 ± 4.58 |
| Raw EEG | 3.79 ± 1.56 | 15.79 ± 3.54 |
| **Chance** | **2.50** | **12.50** |

### Comparison with Liu et al. (2024) Original Baselines

| Method (Liu et al.) | Top-1 (%) | Top-5 (%) |
|---|---|---|
| ShallowNet | 5.59 | 16.93 |
| DeepNet | 4.56 | 14.30 |
| EEGNet | 4.64 | 14.25 |
| Conformer | 4.93 | 15.36 |
| TSConv | 4.92 | 15.05 |
| GLMNet (best) | **6.20** | **17.75** |

Our DE reproduction (4.37%) is directly comparable to DeepNet (4.56%) and EEGNet (4.64%), using the hyperparameters specified in the original paper without additional tuning.

### Key Audit Finding: Run-Position Audit

Accuracy stratified by clip position k ∈ {1,...,5} within each 5-clip concept run:

| Feature | Pos 1 | Pos 2 | Pos 3 | Pos 4 | Pos 5 | Range |
|---|---|---|---|---|---|---|
| DE  | 4.40% | 4.29% | 4.48% | 4.38% | 4.10% | 0.38 pp |
| PSD | 3.84% | 4.39% | 4.29% | 4.13% | 4.31% | 0.55 pp |

**No monotonic trend in either feature type** → the baseline does NOT exploit run-position shortcuts. Models are decoding clip-level content, not temporal anticipation.

### Data Leakage Finding

The original pipeline normalizes globally across train/test splits. After fixing to within-split normalization:
- **PSD:** 13× variance reduction → was sensitive to cross-split contamination
- **DE:** Minor accuracy drop (−0.28%) but training destabilization in some folds
- Accuracy gaps are small → benchmark not fatally compromised, but corrected conditions required for fair comparison

### Within-Chunk Consistency

| Condition | Consistency (%) | Random Baseline (%) |
|---|---|---|
| DE | 24.3 | 17.8 |
| DE_no_data_leak | 27.5 | 17.7 |
| PSD | 24.8 | 17.7 |
| Raw EEG | 27.2 | 24.6 |

Models make more consistent predictions within concept chunks than chance, indicating coarse semantic coherence — but higher consistency doesn't always mean higher accuracy (systematic class confusion).

---

## Architecture

We evaluate 7 architectures: ShallowNet, DeepNet, EEGNet, TSConv, Conformer, GLFNet, GLFNet-MLP. All linear layers are computed dynamically from T and C — eliminating the hardcoded dimension assumptions in the original codebase.

**O(T²) failure:** Attempting standard self-attention over raw EEG (T=400) caused GPU memory overflow (400×400 = 160,000 entries per head). This directly motivated our chunk-level approach below.

---

## Next Steps (Final Report)

### Idea 2: Task-Aware Temporal Attention Pooling
Replace mean pooling with per-task attention heads (concept / color / motion) to reveal which EEG timepoints drive each decoding task. O(T) complexity — avoids the O(T²) bottleneck.

### Idea 3: Chunk-Level Temporal EEG–Video Contrastive Alignment
Partition 2-second clips into K=4 chunks. Align EEG chunks to VideoMAE video chunks with a learned biological latency shift δ ∈ {0, 1, 2} chunk steps (~0–250ms). Evaluated via within-video chunk retrieval (Recall@1, Recall@5).

```
EEG chunks   [e_1] [e_2] [e_3] [e_4]
                ↕δ   ↕δ   ↕δ   ↕δ      ← latency shift
Video chunks [v_1] [v_2] [v_3] [v_4]   ← frozen VideoMAE
```

---

## Dataset

**SEED-DV** — 62-channel EEG at 200 Hz, 1,400 natural video clips (2s), 40 concept categories, 20 subjects, 7 sessions each. Structured as 5-clip runs per concept per block.

---

## Citation

```bibtex
@inproceedings{liu2024eeg2video,
  title     = {EEG2Video: Towards Decoding Dynamic Visual Perception from EEG Signals},
  author    = {Liu, Xuan-Hao and others},
  booktitle = {Advances in Neural Information Processing Systems},
  volume    = {37},
  pages     = {72245--72273},
  year      = {2024}
}
```
