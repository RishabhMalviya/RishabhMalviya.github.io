---
layout: default
title:  "Imitation Learning Deep Dive"
date:   2026-09-23
categories: reinforcement-learning
permalink: /imitation-learning-deep-dive
published: false
---

<h1 align="center">A Deep Dive into Imitation Learning</h1>

---
<br>

## The Three Paradigms of RL
For the purposes of this article, we are going to classify RL algorithms into 3 categories, based on the quality and freshness of data we use in each. Obviously, these are not hard boundaries, and it is possible to combine techniques from different paradigms to create hybrid approaches ([DAgger](https://en.wikipedia.org/wiki/Imitation_learning#DAgger) is one such example, and we will discuss it later in this post).

![RL learning paradigms](/assets/posts/imitation-learning-deep-dive/RL_Learning_Paradigms.png)

### 1. Offline Learning
In offline RL algorithms, the data comes from a past interaction that the agent had with the environment. Usually, this past interaction is done with an untrained version of the agent, so it's likely to contain a lot of bad actions.

I put this at the beginning of the list because the data here is not only stale, but it also likely to be of bad quality. Notice how this places it at the bottom left of the diagram.<br>

### 2. Imitation Learning
In this case, the agent does not gather any data. The training is done with demonstration data gathered from a golden source. This source could be human experts, or it could come from a hand-engineered algorithm.

The data in imitation learning might be static, but it is generally of high quality. However, we will soon see that having exclusively high quality examples in the dataset leads to many interesting problems.<br>

### 3. Online Learning
In these algorithms, the agent is able to interact with the environment *as it trains*. Since the agent must interact with the environment in order to gather training data, we must grapple with one of the most interesting problems in RL - the exploration v/s exploitation trade-off.

It is in this paradigm that we can expect the agent to 'discover' novel strategies. This is because it doesn't merely mimic what human/algorithmic experts do, it autonomously explores new strategies (all in the pursuit of maximizing it's return).<br><br>


## Why Imitation Learning is RL

The final aim of any RL algorithm is to learn a good policy, i.e., a function $$ \pi $$ that can map states to actions:

$$
a_t = \pi(s_t)
$$

such that the chosen actions $$ a_t $$ maximize the expected return on the given task:

$$
\mathbb{E}\left[ \sum_{k=0}^{\infty} \gamma^k R_{t+k+1} \middle| s_t \right]
$$

**NOTE**: I am glossing over details like the discount factor ($$\gamma$$), and the parametrization of the policy (by writing $$\pi$$ instead of $$\pi_{\theta}$$) because they are not essential to the discussion at hand.

In imitation learning (IL), you literally have a dataset of such `state -> action` mappings. So, it would seem that the problem boils down to supervised learning on this dataset. It would also seem that this successfully circumvents the need for any agent-environment interactions. In fact, it would seem that IL removes the need for doing any kind of RL at all!

![IL Training](/assets/posts/imitation-learning-deep-dive/IL_Training.png)

So then...why is imitation learning still considered an RL algorithm?

To answer this question, it helps to how we evaluate RL policies, and where imitation learning is most useful.<br><br>

### Evaluating Imitation Learning Policies

At the end of the day, no matter how you train your RL policy, you have to evaluate it by actually running it in an RL environment. In an RL algorithm where you have some pre-computed trajectories (such as imitation learning), there are two ways of *"running a policy in an RL environment"*:<br>

#### Closed-Loop

This is just normal operation - an agent running exactly as you would expect it to in production:

![ClosedLoop](/assets/posts/imitation-learning-deep-dive/Closed_Loop.png)

The numerical metric is essentially how much reward the policy is able to accumulate.<br>

#### Open-Loop

In open-loop, the 'loop' on the right side of the above diagram is manually overriden by the data in the trajectory on the left side. The way we break the loop is by overriding the states, like this:

![OpenLoop_StateOverride](/assets/posts/imitation-learning-deep-dive/OpenLoop_StateOverride.png)

And if you're thinking "so then...why do we need the environment at all in this case?", you're absolutely right. This form of open-loop evaluation doesn't need an environment; it is equivalent to offline evaluation on a fixed test dataset of `state`-`action` pairs - just like we'd do in supervised learning. In fact, the numerical metric would be something similar to MSE between the policy's predicted actions and the actions we see in the dataset:

![OpenLoop_SupervisedAnalogy](/assets/posts/imitation-learning-deep-dive/OpenLoop_SupervisedAnalogy.png)

The thing to remember here is that in open-loop, the agent does not 'drive'; i.e., it's actions do not determine the next state; they are determined by the data in the trajectory.<br><br>

### Use Cases for Training Imitation Learning Policies
<br>

#### 1. When it's Impossible to Define a Reward Function

The whole field of reinforcement learning is built upon the idea of *rewards*. Every RL algorithm's ultimate objective is to produce an agent that can maximize it's expected (total discounted) reward. But this framework *presumes* that we have a way to calculate the reward given (at a minimum) the following two pieces of information:
1. The state change in the environment
2. The action that produced that state change
In other words, it presumes the existence of a **reward function**:

$$
R_t = R(s_t, a_t, s_{t+1})
$$

![Reward Function](/assets/posts/imitation-learning-deep-dive/Reward_Fn.png)

However, for many real-world scenarios, it's nearly impossible to construct a good reward function. Autonomous driving is a perfect example. For the sake of discussion, let's assume you have a simulator which mirrors what happens in the real-world quite well (a hacked version of GTA, maybe?). 

After thinking really hard about it, you conclude that good driving can be summarized in one succinct sentence: *"Get to where you need to go, without hurting yourself or anyone else on the road"*.

So you let your autononomous driving agent learn with a reward function that:
1. Awards the agent with a large positive reward for reaching the specified destination
2. Penalizes the agent with a large negative reward whenever it touches/hits/crashes into another object

To your dismay, your agent quickly learns to drive the car at a snail-pace, so as to minimize the probability of touching anything on the way to it's destination.

All right, you think, maybe there's more to good driving. You decide to modify the reward function so that it:
1. Awards the agent with a large positive reward for reaching the specified destination *in the least amount of time*
2. Penalizes the agent with a large negative reward whenever it touches/hits/crashes into another object

Now the agent learns to break through red lights, cut past cars and pedestrians, speed through stop signs, and cause all kinds of havoc.

Perhaps all you need to do is to modify the reward function to reward the following of traffic laws. Thinking this, you add a bunch more stuff to your reward function:
1. Awards the agent with a large positive reward for reaching the specified destination in *a reasonable* amount of time *(proportional to the distance between the starting point and the destination)*
2. Penalizes the agent with a large negative reward whenever it touches/hits/crashes into another object
3. Penalizes the agent for driving through a stop sign
4. Penalizes the agent for breaking a red light
5. Awards the agent for driving in the center of it's lane

Now your agent does better, but it fails to capture a lot of nuance and social cues which real drivers naturally exhibit in the real world. For example, because the reward function has awarded your agent for driving exactly in the center of the lane, when a lane next to the agent is blocked (by a vehicle that is loading passengers, for example), your agent will not budge at all to let vehicles in that blocked lane pass through. This causes a lot of frustration to other drivers.

You could continue adding exceptions and cases to your reward function, but this will quickly become intractable. The truth is, good driving isn't something that can really be codified; it's much easier to *demonstrate* good driving than it is to *specify* it. And that's why imitation learning is the superior approach here.

#### 2. When you Need to Bootstrap the RL Process
In it's most raw form, reinforcement learning (in fact, all of machine learning) is a brute force approach.

In pure RL, we start with a randomly initialized policy. The only way to kick-start the learning process for this po;licy is to run it in the environment until it 'accidentally' discovers a good action (which results in reward). This could take a really long time, espexially in sparse reward environments. Wouldn't it be better to bootstrap the agent with good behaviors before you go down the full-blown RL trial-and-error route?

This is another situation where imitation learning is very helpful. In fact, Google DeepMind's [AlphaGo was bootstrapped](https://en.wikipedia.org/wiki/AlphaGo#Algorithm) just like this!





<!-- ### MathJAX Test
The loss is $$ \mathcal{L}(\theta) = -\log p_\theta(a \mid s) $$ for each step.

Display (block) math: put $$...$$ in its own paragraph, with a blank line before and after:

The policy gradient is:

$$
\nabla_\theta J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta}\left[ \sum_t \nabla_\theta \log \pi_\theta(a_t \mid s_t) \, G_t \right]
$$

where $$ G_t $$ is the return.

Numbered equations: use \begin{equation} with \label, then refer to it with \eqref:

$$
\begin{equation}
\label{eq:bellman}
V(s) = \max_a \left[ R(s,a) + \gamma \sum_{s'} P(s' \mid s,a) V(s') \right]
\end{equation}
$$

As shown in \eqref{eq:bellman}, ... -->
