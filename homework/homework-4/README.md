# Homework 4 — Reinforcement Learning for Vision-Language Models (GRPO)

**Rachel Li** · MAS.S60 / 6.S985 · Spring 2026

**Code:** [rachel_li_homework_4_grpo_vlms.py](./rachel_li_homework_4_grpo_vlms.py)

---

## Overview

Fine-tuned a vision-language model using Group Relative Policy Optimization (GRPO), a critic-free RL algorithm. Implemented rule-based reward functions for format and accuracy, trained on visual reasoning tasks, and analyzed the effect of RL fine-tuning on model behavior.

---

## Part 1 — Reading & Reflection

### GRPO vs PPO

Standard PPO requires a learned value function (critic) $V_\phi(s)$ to estimate advantages. GRPO eliminates the critic entirely: for each prompt, it samples $G$ completions and uses group reward statistics as a self-supervised baseline:

$$\hat{A}_i = \frac{r_i - \text{mean}(\mathbf{r})}{\text{std}(\mathbf{r}) + \varepsilon}$$

**Trade-offs of removing the critic:**
- ✅ Simpler training — no second model, fewer hyperparameters
- ❌ No mid-sequence bootstrapping (must wait for full rollout)
- ❌ High-variance baseline with small $G$
- ❌ Signal collapse if all completions receive identical reward

### Reward Model Types

| Type | Strength | Risk |
|------|----------|------|
| Learned reward model | Handles nuanced, subjective tasks | Reward hacking via distributional shift (Goodhart's Law) |
| Rule-based (exact-match) | Interpretable, robust to adversarial outputs | Brittle to paraphrase; rewards form over substance |

In this homework, format + accuracy rewards are combined to mitigate both failure modes.

---

## Part 2 — Implementation

**Reward functions:**
- `format_reward`: checks for correct `<think>...</think><answer>...</answer>` structure
- `accuracy_reward`: exact-match on final answer extracted from the structured output

**Training setup:** GRPO with $G=8$ completions per prompt, KL penalty against reference policy, cosine LR schedule.

**Key finding:** RL fine-tuning improves format compliance significantly in early training, with accuracy gains emerging more gradually as the model learns to reason within the enforced structure.

---

## Part 3 — Analysis

- Models fine-tuned with combined format + accuracy reward outperform accuracy-only variants on held-out visual reasoning tasks
- Format reward acts as a "scaffolding" signal early in training — the model learns output structure before it learns to reason correctly within it
- Reward hacking observed when using accuracy-only reward: model produces valid-format outputs with plausible but incorrect answers that avoid the zero-reward case
