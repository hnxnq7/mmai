# MAS.S60 / 6.S985 — Multimodal AI (2026)
### Rachel Li · MIT · [lirachel@mit.edu](mailto:lirachel@mit.edu)

---

## About Me

Hi, I'm Rachel! I'm an MIT student interested in the intersection of neuroscience and AI — specifically, how we can build systems that decode and reconstruct human perception from brain signals. This repo is my living lab notebook for the Multimodal AI course, tracking ideas, experiments, and results across the semester.

---

## Final Project

**Vision vs. Protocol Effects in EEG2Video: Baseline Audit and Temporal Alignment on SEED-DV**

*with Emma Wang and Winston Qian*

> Can EEG-to-video decoding models genuinely track dynamic visual content, or are they exploiting structured experimental protocols? We audit the EEG2Video framework for run-position artifacts and data leakage, then propose chunk-level latency-aware contrastive alignment to enforce genuine dynamic tracking.

- **GitHub (team repo):** [winstonqian/EEG2Video](https://github.com/winstonqian/EEG2Video)
- **Project README:** [project/](./project/)

### Key Midterm Results

| Condition | Top-1 Acc | Top-5 Acc |
|---|---|---|
| DE (our reproduction) | 4.37% ± 2.64% | 17.17% ± 5.44% |
| GLMNet (Liu et al. 2024 best) | 6.20% | 17.75% |
| Chance (1/40) | 2.50% | 12.50% |

**Run-position audit result** — accuracy across clip positions 1–5:
- DE: [4.40%, 4.29%, 4.48%, 4.38%, 4.10%] ✓ flat — no shortcut artifact
- PSD: [3.84%, 4.39%, 4.29%, 4.13%, 4.31%] ✓ flat — no shortcut artifact

---

## Homework

- [Homework 1](./homework/homework-1/) — Multimodal Dataset Exploration
- [Homework 2](./homework/homework-2/) — *coming soon*
- [Homework 3](./homework/homework-3/) — *coming soon*

---

## Course Notes & Explorations

Side experiments, paper notes, and ideas I'm exploring throughout the semester will live here as I go.

---

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">
<img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a>
This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.
