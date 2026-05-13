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
3. **Proposes and validates LATA** — a chunk-level latency-aware EEG–video contrastive alignment module

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
| PSD | 4.15 ± 2.60 | 17.15 ± 4.89 |
| PSD_no_data_leak | 4.29 ± 2.29 | 17.13 ± 4.87 |
| Raw EEG | 3.79 ± 1.56 | 15.79 ± 3.54 |
| **Chance** | **2.50** | **12.50** |

### Key Audit Findings

**Run-position audit:** No monotonic trend in accuracy across clip positions k ∈ {1,...,5} (range < 0.55 pp for both DE and PSD) → the baseline does **not** exploit run-position shortcuts. Models decode clip-level content, not temporal anticipation.

**Data leakage:** The original pipeline normalizes globally across train/test splits. After fixing to within-split normalization: PSD shows 13× variance reduction; accuracy gaps are small → benchmark not fatally compromised, but corrected conditions required for fair comparison.

---

## Final Results — LATA: Latency-Aware Temporal Alignment

### Method

Standard cross-attention assumes time $t$ in EEG corresponds to time $t$ in video. This is physically wrong: neural responses always **lag** the driving stimulus by a biological transit delay δ (P100 ~100ms, P300 ~300ms). LATA learns this delay end-to-end.

**Module design:** LATA maintains a trainable logit vector $\boldsymbol{\ell} \in \mathbb{R}^{\Delta+1}$ over candidate delays $\delta \in \{0,...,\Delta\}$:

$$\mathbf{w} = \mathrm{softmax}(\boldsymbol{\ell}), \qquad \tilde{\mathbf{v}}_k = \sum_{\delta=0}^{\Delta} w_\delta\, \mathbf{v}_{k-\delta}$$

Trained with a **latency-aware InfoNCE loss** — gradients flow through $\mathbf{w}$ differentiably, driving delay logits toward the true biological lag with no supervision on δ.

### Synthetic Validation

LATA correctly recovers ground-truth delays in all 4 tested cases:

| δ_true | Peak of learned **w** | Correct? |
|--------|----------------------|---------|
| 0 | 0 | ✅ |
| 1 | 1 | ✅ |
| 2 | 2 | ✅ |
| 3 | 3 | ✅ |

### SEED-DV Results (20 subjects)

| Metric | Value |
|--------|-------|
| Subjects peaking at δ*=2 (~810ms) | **14 / 20 (70%)** |
| Subjects peaking at δ*=1 (~500ms) | 6 / 20 (30%) |
| Subjects peaking at δ=0 or δ=3 | 0 / 20 |
| Population mean E[δ] | **1.58 ± 0.05 chunks ≈ 790ms** |
| Mean learned distribution **w̄** | [0.197, 0.268, 0.289, 0.246] |

The convergence of 70% of subjects to δ*=2 — with no subject at δ=0 (no lag) or δ=3 (1500ms) — provides strong evidence LATA is recovering a genuine biological signal. The preferred 500–1000ms window is consistent with late positive ERP components (P300, N400, late positive complex). The near-identical distributions across subjects (SD of E[δ] = 25ms) confirm the learned latency reflects a **population-level property** of the visual EEG response, not subject-specific noise.

---

## Architecture

We evaluate 7 architectures: ShallowNet, DeepNet, EEGNet, TSConv, Conformer, GLFNet, GLFNet-MLP. All linear layers computed dynamically from T and C — eliminating hardcoded dimension assumptions in the original codebase.

**O(T²) failure:** Standard self-attention over raw EEG (T=400) causes GPU memory overflow. This directly motivated the chunk-level approach.

---

## Dataset

**SEED-DV** — 62-channel EEG at 200 Hz, 1,400 natural video clips (2s), 40 concept categories, 20 subjects, 7 sessions each. Structured as 5-clip runs per concept per block.

---

## Repository Structure

```
project/
├── README.md
├── lata/
│   ├── lata.py                      ← LATA module (plug-and-play PyTorch)
│   ├── train_lata_seeddv.py         ← training script (all 20 subjects)
│   ├── synthetic_validation.py      ← synthetic delay recovery experiment
│   ├── lata_synthetic_validation.png
│   ├── lata_all_subjects_results.png
│   └── lata_seeddv_results.png
```

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
