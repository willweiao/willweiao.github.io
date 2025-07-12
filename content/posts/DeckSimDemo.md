+++
date = '2025-07-11T22:29:42+02:00'
draft = false
title = 'Combating Sparse Rewards in Reinforcement Learning with Simulation-Based Reward Signals'
subtitle = 'How I used card battle simulations to guide RL agents without hand-crafting dense rewards'
+++

This post is a short follow-up to one of my projects—a simple turn-based card battle simulator. I've been a fan of Slay the Spire for a long time, and I've often wondered how its gameplay could be connected to reinforcement learning—things like training self-play agents or building systems that help guide player decisions. This write-up explores some of those ideas, especially how they relate to sparse rewards, and how that line of thinking eventually led to the project itself.


## Introduction: When Winning Isn’t Enough

Reinforcement learning in games is often glamorous in theory and brutal in practice. While the high-level goal—“win the game”—is easy to define, the path to that goal is littered with ambiguity. Especially in strategy games, agents can play dozens or hundreds of actions before knowing if anything they did was useful.

This is the classic *sparse reward* problem: the agent receives no meaningful feedback until the very end. For games like Slay the Spire, where each run consists of dozens of battles, and each battle has multiple rounds of card-play, that final win/loss signal just doesn’t carry enough information to guide learning.

So, instead of rewriting the game environment or inventing hand-crafted reward functions, I asked a different question:

> *Can we use simulation data to build reward signals from the bottom up?*

In this post, I’ll walk through how I built a turn-based card battle simulator and used it to generate **proxy rewards**—statistical signals that help RL agents learn from indirect feedback. This approach helped me sidestep the sparse reward trap, without changing the core rules of the game.

## The Environment: A Custom Card-Battle Simulator

To experiment with this idea, I built a lightweight card-battle simulator inspired by roguelike deckbuilders like Slay the Spire. The goal was to strip down the game to its tactical core—cards, enemies, and turn-based combat—so that I could simulate battles in bulk and analyze how different decisions impact the outcome.

The simulator works like this: each battle pits a player character with a hand of cards against a scripted enemy. On each turn, the player selects one card to play (e.g. Attack, Defend, Buff), while the enemy follows a fixed behavior pattern. After a few rounds, the battle ends in either victory or defeat, and that outcome becomes the reward signal.

While simple, the environment is designed to be modular and extensible. Cards, enemies, and battle rules are all defined as Python objects, so adding new behaviors or mechanics takes only a few lines of code. More importantly, the entire system is built to support batch simulations—I can run thousands of battles in parallel, log detailed information about each action, and use that data to inform strategy decisions.

By keeping the environment minimal but expressive, I was able to focus on what matters most: building a system that’s easy to simulate, analyze, and connect with learning algorithms.

## Why This is a Sparse Reward Setting

Like many game environments, my simulator gives feedback only at the end of each battle. If the agent wins, it gets a reward of +1. If it loses, it gets 0. That’s it—no hints, no nudges, no partial credit along the way.

This setup means that even if the agent makes a smart move on turn 3—like playing a defense card at the perfect time—it won't know it was a good decision unless that one move somehow contributed to winning the entire battle. And since battles often last 10, 15, or even 20 turns, the link between “what you did” and “what you got” becomes extremely blurry.

This causes two big problems in reinforcement learning:

- **Credit assignment**: It’s hard for the agent to figure out which actions were actually responsible for success.

- **Slow convergence**: With so little signal per episode, the agent needs to explore and fail many times before learning even basic patterns.

In short, the environment works fine as a game—but it’s far from friendly to an RL agent trying to learn from scratch.

## Common Approaches to Sparse Rewards 

Sparse rewards are a well-known problem in reinforcement learning, and the literature is full of clever ways to work around them. Here are some of the most common strategies—and how they might apply to a card-battle environment:

- *Reward shaping*: This involves manually designing additional reward signals to guide learning—for example, giving a small bonus whenever the enemy loses HP or the player blocks a large attack. While this can work, it requires deep domain knowledge and careful tuning. In my case, trying to assign numeric value to every card or state transition would have been brittle and time-consuming.

- *Curriculum learning*: Start the agent off in simpler situations—say, battles with fewer enemies or more health—and gradually increase the difficulty as it learns. This approach can be powerful, but it adds complexity to environment design and often requires multiple versions of the game state. I wanted to keep the environment lightweight and consistent across runs.

- *Imitation learning*: Train the agent to mimic expert demonstrations, if such data is available. But in my setting, there is no expert—I’m not a professional card battle strategist, and there’s no large dataset of optimal play logs. Generating high-quality demonstrations would take as much effort as solving the problem directly.

- *Hindsight experience replay (HER)*: Often used in goal-conditioned environments, this technique lets agents learn from failed episodes by pretending the outcome was the intended goal. Unfortunately, HER doesn’t translate well to games like mine, where there isn’t a natural goal space to reshape.

That’s why I chose a different path: instead of injecting knowledge or manually rewriting the reward structure, I used massive simulation to uncover patterns that already exist in the environment. By collecting data on what tends to work over thousands of games, I was able to create a dense, data-driven signal—one that’s cheap to compute, grounded in outcomes, and easy to plug into an RL training loop.

## My Approach: Simulation as a Proxy Reward Generator

Now I want to show my approach to this issue as in the project. Instead of rewriting the environment or inventing new reward functions, I leaned into the strength of simulation. I wrote a script that could run thousands of battles automatically using a simple, semi-random policy: choose a legal card at each turn with a slight bias toward stronger effects, and let the battle play out.

Each simulation logged not just the final result (win or lose), but also which cards were played at each step of the battle. By collecting this data across thousands of games, I was able to build a statistical map: how often was a particular card played in a battle that ended in victory?

The idea was simple: **if a card is frequently present in winning trajectories, maybe it’s a good card**. More specifically, maybe playing that card in a certain context increases your chances of winning.

This gave me a new kind of reward signal—not handcrafted, but learned through simulation. I called it a proxy reward, because it doesn't come from the environment directly, but rather from analyzing patterns in the outcomes of many episodes.

## Applying Proxy Rewards to Dense Up the Signal

With these proxy rewards in hand, I could now give the agent more frequent feedback. During training, whenever the agent played a card, I could look up its proxy score—say, a 0.62 win rate from past simulations—and use that as an estimated value for the action.

In effect, this replaces the sparse +1/0 terminal reward with a dense stream of per-action feedback, derived from data.

This small change makes a big difference. Instead of learning from success or failure once every 20 steps, the agent can now adjust its policy every single turn. It gets an approximate—but meaningful—signal about which decisions are worth repeating.

Of course, this reward signal is noisy. It doesn’t guarantee that a certain card caused the win. But in practice, it helps the agent avoid the early flailing stage where it plays random actions and sees no reward for dozens of episodes. The proxy reward gives it a direction, even if the signal isn’t perfect.

## Limitations of this Method

While proxy rewards based on simulation data offer a practical way to reduce reward sparsity, they come with important limitations.

First, **correlation is not causation**. Just because a card frequently appears in winning trajectories doesn’t mean playing it causes the win. Some cards might be commonly used simply because they are always available in strong decks, or because they appear late in battles that are already likely to be won. This kind of statistical bias can mislead the agent into overvaluing certain actions.

Second, my current implementation treats card usage in isolation—without modeling context or sequence. That is, it assumes a card’s effectiveness is independent of when and how it’s played. In reality, the value of a card often depends heavily on the current state (e.g. energy, enemy intent, player HP) and the surrounding play sequence. Without capturing that nuance, the proxy reward may miss key strategic interactions.

Third, the proxy reward system relies on a fixed policy for simulation, which may not reflect optimal or even reasonable play. If the underlying simulations are noisy or unrepresentative, the reward estimates will be skewed. For example, in my simulations I assume enemies follow fixed intent every turn, however in real *Slay the Spire* game, each enemy's intent is constructed in a way of Markov chain, i.e. they usually don't have a fixed sequence of intents, instead they have different probability changing between possible moves. Over time, the system might need to re-simulate using improved policies (e.g. the agent’s own evolving policy) to stay relevant.

## Outlook: Training Full Tower-Run Agents

So far, I’ve focused on single battles—but most roguelike deckbuilders are much more than that. A full tower run includes many layers of decision-making: drafting new cards, choosing which path to take, deciding what to buy in shops, and managing risk over a long sequence of fights. All of these choices are currently made without direct reward signals until the very end of the run, making sparse feedback an even bigger problem at the macro level.

The same idea behind proxy rewards can be extended to these broader decisions. For example:

- We could simulate multiple tower runs and measure which card draft patterns tend to correlate with later success.

- We could estimate the long-term value of different shop purchases based on their impact on win rates.

- We could log path choices and outcomes to model risk-reward trade-offs in map traversal.

Ultimately, this opens the door to more structured learning approaches, such as hierarchical RL, where a high-level policy chooses strategic goals (e.g. “build a scaling deck”), and a low-level policy executes tactics (e.g. which card to play this turn). By combining proxy rewards at both levels, agents could start to learn multi-phase, long-horizon strategies without the need for fully shaped rewards or handcrafted rules.

## Conclusion & Reflections

Sparse rewards can make reinforcement learning frustratingly slow and brittle—especially in games where good decisions only pay off long after they're made. But with enough simulation data and a bit of statistical thinking, it’s possible to construct reward signals from the ground up, rather than relying on expert rules or intrusive environment modifications.

In this project, I used thousands of simulated battles to estimate the impact of each card on win probability, then used those estimates as proxy rewards to help the agent learn. This approach is scalable, model-free, and well-suited to environments where exploration is cheap but rewards are rare.

Directly, it can be used to check if a deck is strong enough to beat a specific enemy, where both deck and enemies can be freely customized in my project. Although many functions in real *Slay the spire* game haven't been implemented in my project—such as potions, relics, or dynamic enemy scaling—the core combat mechanics are largely complete. The simulation captures the essence of turn-based decision-making, including card effects, energy management, enemy intents, and win/loss conditions, making the results meaningful and relevant for strategic analysis or early-stage agent training.

More importantly, it shows that learning doesn't have to stop just because the environment is quiet. If we take the time to simulate, log, and analyze what works—even in noisy or indirect ways—we can still guide our agents toward good strategies.
