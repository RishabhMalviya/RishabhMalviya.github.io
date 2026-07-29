---
layout: page
title: BSP
description: Body Schema Pretraining — a reward-free, curiosity-driven pre-training framework that makes downstream robotic locomotion RL substantially more sample-efficient.
img: assets/img/projects/bsp-overall-idea.png
importance: 2
github: https://github.com/RishabhMalviya/bsp
---

**Body Schema Pretraining (BSP)** is a two-stage framework for sample-efficient reinforcement learning in robotic control. The core idea starts from a simple structural observation: **a robot's own body dynamics are task-agnostic and fixed.** The mapping from torques to accelerations, the joint coupling, the inertial structure — none of it changes when the task changes from _stand_ to _walk_ to _run_. Yet a vanilla RL agent re-learns all of it from scratch, jointly with the task, for every new problem.

BSP factors that body-dynamics learning _out_ of RL and completes it **once, ahead of time**:

1. **Pre-train** a `DynamicsPredictor` Transformer (DPT) on a single robot body, reward-free, using **curiosity-driven exploration** and a **masked-language-modeling (MLM)-style** reconstruction loss.
2. **Re-use** that same pre-trained Transformer as the **actor backbone** for task-specific training on downstream tasks that share the same body.

The hypothesis is that this body-first prior greatly increases the sample efficiency of downstream task-specific training compared to training from scratch.

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/bsp-overall-idea.png" title="Pre-train on a robot body, then fine-tune on many downstream tasks" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

The environments come from the **DeepMind Control Suite** (via Shimmy + Gymnasium), which conveniently exposes several tasks that share an identical robot body — e.g. `humanoid` and `walker`, each with `stand` / `walk` / `run`.

This project was built for Stanford's graduate course on RL for Robotics, [CS224R](https://cs224r.stanford.edu/).

## Stage 1 — Curiosity-driven pre-training

In standard RL, the environment hands the agent an extrinsic reward signal. BSP pre-training is **reward-free** in the task sense. Instead, it follows an [Intrinsic Curiosity Module (ICM)](https://arxiv.org/abs/1705.05363)-style scheme: the reward signal is **intrinsic** and equal to the DPT's own **next-state prediction error**. The exploration agent is therefore rewarded for visiting transitions the DPT models poorly, pushing it to maximally cover the body-dynamics manifold. Because the DPT is trained on the collected trajectories at the same time, the agent and the predictor improve together in a single joint loop.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/bsp-basic-rl-setup.png" title="Basic RL setup" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/bsp-curiosity-loop.png" title="Curiosity pre-training loop" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left, the standard RL loop with an extrinsic reward. Right, the curiosity loop — the intrinsic reward <em>is</em> the DPT's next-state prediction loss.
</div>

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        <img src="https://raw.githubusercontent.com/RishabhMalviya/bsp/main/readme_assets/curiosity-pretraining.gif" alt="Evaluation run for curiosity-driven pre-training" class="img-fluid rounded z-depth-1" loading="lazy" />
    </div>
</div>
<div class="caption">
    An evaluation run during curiosity-driven pre-training — the exploration agent drives the body around to surface novel, hard-to-predict transitions.
</div>

### The MLM-style objective

The DPT is an **encoder-only Transformer** that consumes interleaved sequences of `(state, action)` tokens, `τ₁:ₗ = (s₁, a₁, s₂, a₂, …, sₗ, aₗ)`. Trajectories are sampled from the replay buffer, a random subset of token positions is **masked**, and the model is trained to **reconstruct the masked tokens** from the surrounding context (BERT-style MLM), under an MSE loss.

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/bsp-dpt-pretraining.png" title="DPT pre-training" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Notable implementation choices during pre-training:

- **SimNorm latents** — a soft simplex-projection over groups of the per-position output, producing soft-discretized latents (as in DreamerV3 / TD-MPC2) for better-conditioned representations.
- **Welford running normalization** of observations, since DM Control components (positions, velocities, contact forces) live on very different scales.
- **Variable-length history sampling** biased toward longer windows, so the DPT is exposed to the short contexts it will encounter at the start of fine-tuning.
- **Action-smoothness (jerk) + entropy regularization** on the exploration agent, to stop it from "gaming" the intrinsic reward with high-frequency oscillations instead of genuine exploration.
- **Explicit dead-end truncation**, so uninformative frozen states don't poison the replay buffer.

## Stage 2 — Task-specific fine-tuning

For downstream training, the reconstruction head is removed and an **action head** is attached: the per-position DPT outputs are average-pooled and passed through a `tanh` to produce an action distribution. The same pre-trained Transformer now serves as the **actor** inside a standard actor-critic algorithm (PPO), with a separate MLP critic.

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/bsp-dpt-finetuning.png" title="DPT fine-tuning" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

The pre-trained DPT is then fine-tuned on each downstream task individually — `stand`, `walk`, `run` — each sharing the body it was pre-trained on.

## Results

The central comparison is PPO with a **pre-trained** DPT actor vs. PPO with a **randomly initialized** DPT actor, on `walker-stand`, under identical fine-tuning hyperparameters. The pre-trained actor (green) consistently reaches higher reward, and faster, than training from scratch (blue).

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/bsp-results.png" title="Final performance comparison" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

An additional engineering finding: the **choice of exploration algorithm during pre-training matters at least as much as the representation objective**. Because the intrinsic reward is non-stationary — it tracks the DPT's evolving error — DDPG's deterministic/off-policy assumptions destabilized the loop, whereas **PPO's stochastic, trust-region updates** tolerated it well. The final pipeline is PPO end-to-end.
