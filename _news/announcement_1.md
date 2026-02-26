---
layout: post
title: "PlanB&B accepted at AAAI 2026"
date: 2026-01-15 09:00:00+0100
inline: false
related_posts: false
---

Our paper **Planning in Branch-and-Bound: Model-Based Reinforcement Learning for Exact Combinatorial Optimization** has been accepted at **AAAI 2026**!

This work introduces [PlanB&B](../projects/planbb/), the first model-based reinforcement learning (MBRL) agent for exact combinatorial optimization. Building on the principled MDP formulation of [BBMDP](../projects/bbmdp/) (NeurIPS 2025), PlanB&B goes one step further by learning an internal model of branch-and-bound dynamics and using Monte Carlo Tree Search (Gumbel Search) to plan improved branching decisions — without any additional LP solves at inference time.

Key results on four standard MILP benchmarks:
- **2–3× improvement** over prior RL baselines in both tree size and solving time
- **First RL agent to surpass imitation learning** trained on strong branching
- Up to **50% tree size reduction** with additional search budget at inference
- Discovers **genuinely novel branching strategies**, actively diverging from expert patterns

**Authors:** Paul Strang, Zacharie Alès, Côme Bissuel, Olivier Juan, Safia Kedad-Sidhoum, Emmanuel Rachelson

**Links:** [Project page](../projects/planbb/) · [arXiv](https://arxiv.org/abs/2511.09219) · [OpenReview](https://openreview.net/forum?id=8481hfKVsr)
