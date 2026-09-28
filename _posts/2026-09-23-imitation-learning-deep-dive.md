---
layout: default
title:  "Imitation Learning Deep Dive"
date:   2026-09-23
categories: reinforcement-learning
permalink: /imitation-learning-deep-dive
---

<h1 align="center">Why Imitation Learning Isn't Just Supervised Learning</h1>

---
<br>

# Let's Start With the Basics: Behavior Cloning
Most basic algorithm is BC. In BC, you literally have a dataset of such `state -> action` mappings. So, it would seem that the problem boils down to supervised learning on this dataset.

![IL Training](/assets/posts/imitation-learning-deep-dive/IL_Training.png)

## Pick-Place-v3
For this blog, we will use demonstrations from the oracle in pick-place-v3:

- Pick-Place-v3 Oracle video
- Pick-Place-v3 Oracle success rate
- Pick-Place-v3 Oracle puck pick-up code v/s video visualization
<br><br>

## Training a BC Agent
It learns to mimic individual state-action pairs very well.

- Show loss curve
- Show offline accuracy


## Evaluating The Trained BC Agent
- Show success rate of trained BC Agent (it should be low)


# Why BC Doesn't Work: Covariate Shift
As you read about BC, you'll see people saying that it doesn't work because of covariate shift. Which is taken straight from non-RL ML literature and just means that the input distributions are mis-matched between training and deployment.

So maybe, it's just a case of 'the real world is messy', and that messiness isn't captured in the training data. Let's test this hypothesis with our current setup.

## Testing the "real world is messy" Hypothesis

- Inject noise into observations, show that the oracle fails (success rate falls + Video)
- Train BC Agent with noisy observations show that it's better than the oracle (Video)
- But BC Agent (trained with noise) still has <100% success rate

## Why?
Because in RL, the source of the covariate shift isn't just dist mis-match because you're unable to capture all the nuances of the real-world. In fact, most demonstrations come from experts operating in a real environment (as in autonomous driving), so real-world messiness (like sensor noise) is sufficiently captured. 

Covariate shift in an RL setting is inherent because RL is a sequential decision making paradigm. Let's dig deeper into the differences between training and evaluation settings:

### Closed-Loop vs Open-Loop
- Closed-Loop Diagram
- Open-Loop Diagram

## Visualizing the Difference Between Open and Closed Loop Runs
Let's see this in the Pick-Place-V3 environment. During closed-loop eval, the agent will run into states that the oracle never runs into. Especially around the hard boundary (where the gripper closes when it is an exact z-distance above the puck).
- Visualize histogram of puck-gripper-xy-distance when gripper=~0.2 - for oracle
- Visualize histogram of puck-gripper-xy-distance when gripper=~0.2 - for BC Agent (trained without noisy states)
- Visualize histogram of puck-gripper-xy-distance when gripper=~0.2 - for BC Agent (trained with noisy states)

## The Real Source of Covariate Shift in BC
The core issue is that expert demonstrations never contain information about what to do if the agent messes up. It never learns to recover from bad states.
- Show classic car crash diagram for BC (from ivy league lectures)

# Solving Covariate Shift When you Have an Oracle: DAgger
Only works if you have an oracle. In this case we do...so let's see if DAgger actually works!
- Visualize performance of DAgger agent on closed-loop eval (success rate)

# Solving it When You Don't Have an Oracle: Diffusion Policies

# Solving it When You Don't Have an Oracle: Loss Function is on states reached instead of actions executed
