---
layout: default
title:  "My Learnings from Learning Reinforcement Learnning"
date:   2026-06-12 19:51:44 -0800
categories: reinforcement-learning
permalink: /my-learnings-from-learning-reinforcement-learning
---

# Disillisionment

My journey with reinforcement learning has been full of disillusionments. While it is undoubtedly a difficult subject, the way it is taught in most schools makes it even harder than it needs to be. I hope to get straight to the heart of the matter in this blog post.

## Pedagogy is Overrated
For one thing, I don't think it is useful to teach ideas in the sequence in which they were discovered/invented. We've figured a lot of things out over the years, and it probably makes the most sense to go straight to the stuff that works.

## Everything is a Bag of Tricks
Also, we should be honest with students about the fact that none of what we do in RL has a very strong theoretical grounding - at the end of the day, things proceed largely through trial and error. This is more art than science. Other than a high-level mathematical framework, most of the mathematics we introduce is, in my opinion, simply unneccessary. With that said, I'll quickly go over the high-level framework here, before proceeding to my learnings.

If you go through the papers for landmark algorithms like PPO, you will very quickly find a paragraph/section buried somewhere in the paper that is just an ugly mess of tricks (and hyperparameter values). Calling these 'algorithms' and naming them is unhelpful, in my opnion. What really matters is the combination of tricks that worked well in tandem. I believe we should be teaching students individual tricks, and then present the algorithms as particular instances where collections of tricks have worked well.

What really needs to be learned is how to develop an intuition for which tricks work well together.


# High-Level Mathematical Framework
The overall paradigm of RL is actually quite simple. You have an agent, you have an environment. The state of the environment changes when the agent interacts with it. However, if the agent interacts with the environment correctly, it can also get rewards! And that's the thing the agent is trained to maximize. Examples - game playing, robotics manipulation tasks.

At the end of the day, all the shenanigans you do in RL are towards one goal and one goal only - learning a policy. That is, a mapping from state to action. If the agent learns a good policy, it will be able to get high rewards.

Note how the policy only needs to know the current state to decide the next action; this is where a computer/robot would differentiate itself from a human - the computer/robot looks at every state with fresh eyes, it is not psychologically affected by past bad moves. It only sees the truth - what moves lead to bad results and what moves lead to good ones.

## What makes RL RL?
The fact that you start with no data, and the only way to collect it is to interact with the environment. This is the real crux of what sets RL apart from other paradigms of ML (like supervised and unsupervised learning). Whatever policy you are trying to train is the same policy you will collect data with. This can create a viscious feedback loop where collecting data with a bad policy leads to bad training data, which the agent then never learns from.

This is why so much time is spent in getting agents to explore their environment. Every well-known and practically useful RL algorithm today achieves this by introducing randomization into the actions predicted by its policy. That is, instead of outputting one action given a state, we have the policy output a **distribution** over the action space (introduce notation). Then, we can choose to sample from the distribution with high noise (to encourage exploration) or with low noise (to encourage exploitation). (Explain with examples; robot arm and game playing)

### Distirbutional NNs
Training neural networks that output distributions is tricky, because sampling isn't really a differentiable operation.

## The Unneccessary Dichotomy of On-Policy and Off-Policy Algorithms
I have always found this to be an absurd distinction. In fact, is isn't really a distinction. It is a continnum. You can choose to use a lot of old data (then you are off-policy). Or you can choose to use very little old data (the you are on-policy).

Granted, some methods lend themselves more naturally to one side of the spectrum than the other (for example, PPO is often classified a on-policy because it isn't able to make good use of old data). But that is just a design choice that you make when crafting your code. It isn't some fundamental distinction in the space of RL algorithms, and should not be presented as such. In fact, I would say this distinction should not be taught at all. Maybe just as an afternote saying "the research community like to classify RL method into two categories...".


## You can Optimize Anything
RL taught me a completely new way to think about deep learning. The parameters of a model need not be trained through a loss function (which is calculated with ground truth values). You could train it maximize/minimize any quantity that it outputs!

DDPG does exactly this. DDPG is what is called an actor-critic algorithm. This is just an umbrella term for an approach that has worked well for us. As usual, we try to train a policy (a.k.a, the 'actor'). But we also train something called a value function ('critic'). This value function takes in a state, and outputs the expected reward from that state. (Example with game playing). In DDPG, we do something really clever where we take a (state, action) pair from the replay buffer, pass it to the environment to see the next_state, and then pass it through the valu function to see the value of the next_state. The actor is then trained to maximize the value of the next_state (no loss function or ground truth comparison involved)!

---