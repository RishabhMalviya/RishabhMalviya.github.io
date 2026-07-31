---
layout: default
title:  "My Learnings from Learning Reinforcement Learning (WIP)"
date:   2026-06-12 19:51:44 -0800
categories: reinforcement-learning
permalink: /my-learnings-from-learning-reinforcement-learning
---

<h1 align="center">A New Pedagogy</h1>

My journey with reinforcement learning has been full of disillusionments. While it is undoubtedly a difficult subject, the way it is taught in most schools makes it even harder than it needs to be. I hope to present a better pedagogy for the subject here.

## Motivating the Mathematical Framework
We should begin by motivating why the mathematical framework is useful, and why each component is neccessary in the first place. For example, when you learn linear algebra for the first time, you can see that it will apply very generally to a wide variety of different spaces of objects (polynomials, audio signals, geometric vectors, higher dimensional vectors) and associated transformations. Similarly, we must make it obvious to students why the RL framework is the way it is by showing that it's seemingly peculiar formulation is precisely what make it so widely applicable to such a wide variety of problems - ranging from turn-based games with discrete action spaces to robotic control with continuous action spaces of multiple dimenstions.

For most students (including me) this takes a lot of time to understand, especially because we've been entrenched in the worlds of supervised and unsupervised learning for a long time. I believe that emphasizing a few specific aspects of the mathematical framework (which aren't given sufficient importance in current curricula) would help make the uniqueness of RL quite apparent.

## Simplifying the Mathematical Framework
The overall paradigm of RL is actually quite simple. You have an agent, you have an environment. The state of the environment changes when the agent interacts with it. However, if the agent interacts with the environment correctly, it can also get rewards! And that's the thing the agent is trained to maximize. Examples - game playing, robotics manipulation tasks.

At the end of the day, all the shenanigans you do in RL are towards one goal and one goal only - learning a policy. That is, a mapping from state to action. If the agent learns a good policy, it will be able to get high rewards. 

### The Data Collection Problem
The fact that you start with no data, and the only way to collect it is to interact with the environment. This is the real crux of what sets RL apart from other paradigms of ML. Your algorithm needs to collect data, and it needs to train a policy with that data too. In most cases, the policy you collect data with is derived from the policy you are trying to train. This can create a viscious feedback loop where collecting data with a bad policy leads to bad training data, which the agent then never learns from.

It would also help to explain how a lot of modern ML systems are actually RL systems. Take recommendation/feed systems for example. Sure, you train them using supervised techniques, but if you take a step back, you'll realize that that is only the first step in the loop. The actual loop is this:
1. You train and deploy a feed/recommendation model (action)
2. You let run/AB test the model in production and gather online metrics (reward).
3. You re-train the algorithm by using everything at your disposal - previous models' performance, seasonality information, data gathered from previous models' user interactions (this is effectively the state)

### Exploration vs Exploitation
The data collection problem naturally leads to the problem of exploration vs exploitation. I'm a little hesitant to say that this is 'fundamental' to RL. But it is so ubiquitous that a modern formulation of RL would probably need to take this into account.

Every well-known and practically useful RL algorithm today achieves exploration by introducing randomization into the actions predicted by its policy. That is, instead of outputting one action given a state, we have the policy output a **distribution** over the action space (introduce notation). Then, we can choose to sample from the distribution with high noise (to encourage exploration) or with low noise (to encourage exploitation). (Explain with examples; robot arm and game playing)

### Markov-ness
We are taught that the policy only needs to know the current state to decide the next action; and that this is where a computer/robot would differentiate itself from a human - the computer/robot looks at every state with fresh eyes, it is not psychologically affected by past bad moves. It only sees the truth - what moves lead to bad results and what moves lead to good ones.

But Markov-ness isn't truly inherent to the framework. Plenty of implementations out there work well because they consider a history of states (for example, early Atari agents took into account the last 4 frames of data).

I think it is more instructive to classify this as part of the feature engineering for LR algorithms. If we engineer the features for the states smartly, then the states should have enough information for the agent to make good decisions about which action it should take. In this sense, the state evolution is 'Markovian', but this is only because we engineer states in such a way (not because RL inherently requires Markovian state evolution).

### Value Functions
In particular, I think it is possible to develop all the theory of RL (*and* to motivate all the major algorithms) *without* first introducing the concept of a value functions. I believe it's only necessary to introduce value functions as part of actor-crtitc algorithms.

The only thing you need to introduce in the beginning is the idea that we want to maximize the expectation of the return (sum of discounted rewards). Defining something similar for each state (i.e., the value function) isn't strictly necessary to understand deep RL. I feel like value functions make a lot of intuitive sense in game-playing, and they have stuck around in teaching curricula because RL was initially developed for game-playing (AlphaGo, AlphaZero, MuZero). 

### The Unneccessary Dichotomy of On-Policy and Off-Policy Algorithms
I have always found this to be an unneccessarily confusing distinction. In fact, is isn't really a distinction, it is a continnum. You can choose to be more off-policy by using a lot of old data (with a large `ReplayBuffer`), or you can choose to be on-policy by using very little old data (with a small `ReplayBuffer`).

Granted, some methods lend themselves more naturally to one side of the spectrum than the other (for example, PPO is often classified as on-policy because the clipping of the importance sampling term essentially renders it unable to make good use of old data). But that is just a design choice that you make when crafting your code. It isn't some fundamental distinction in the space of RL algorithms, and should not be presented as such. In fact, I would say this distinction should not be taught at all. Maybe just as an afternote saying "the research community like to classify RL method into two categories...a number of tricks help make better use of old data (in what the research community dubs 'off-policy' algorithms)".

### Following The Historical Evolution of the Subject is Overrated
For one thing, I don't think it is useful to teach ideas in the sequence in which they were discovered/invented. We've figured a lot of things out over the years, and it probably makes the most sense to go straight to the stuff that works.

For example, we can probably skip tabular Q-learning completely. Once you've established the recursion relation for value functions, just go straight to DQNs and update the rules for them.

### Everything is a Bag of Tricks
Also, we should be honest with students about the fact that at the end of the day, things proceed largely through trial and error. This is more art than science. Other than a high-level mathematical framework, most of the mathematics we introduce is (in my opinion) simply unneccessary.

If you go through the papers for landmark algorithms like PPO/TD-MPC, you will very quickly find a paragraph/section buried somewhere in the paper that condenses a huge mess of tricks (and hyperparameter values for those tricks). Calling these 'algorithms' and naming them is unhelpful, in my opnion. What really matters is the combination of tricks that worked well in tandem. 

So, for example, instead of teaching students SAC, we would teach them about (1) entorpy maximization terms, (2) twin Q-networks, and (3) reparametrization tricks to allow back-prop directly through actor networks. 

I believe we'd be better equipping students by teaching them these individual tricks, and then presenting algorithms as particular instances where collections of tricks have worked well in tandem. What really needs to be learned is why the tricks work, along with an intuition for which tricks work well together.

## Connections to Other Areas of Study
### Generative Model and Distirbutional NNs
Training neural networks that output distributions is tricky, because sampling isn't really a differentiable operation.

### General Theory of Computational Graphs (You can Optimize Anything!)
RL taught me a completely new way to think about deep learning. The parameters of a model need not be trained through a loss function (which is calculated with ground truth values). You could train it to maximize/minimize any quantity that it outputs!

DDPG does exactly this. DDPG is what is called an actor-critic algorithm. This is just an umbrella term for an approach that has worked well for us. As usual, we try to train a policy (a.k.a, the 'actor'). But we also train something called a value function ('critic'). This value function takes in a state, and outputs the expected reward from that state. (Example with game playing). In DDPG, we do something really clever where we take a (state, action) pair from the replay buffer, pass it to the environment to see the next_state, and then pass it through the value function to see the value of the next_state. The actor is then trained to maximize the value of the next_state (no loss function or ground truth comparison involved)!

---