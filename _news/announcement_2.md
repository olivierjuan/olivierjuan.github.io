---
layout: post
title: "BBMDP accepted at NeurIPS 2025"
date: 2025-10-22 09:00:00+0100
inline: false
related_posts: false
---

Our paper **A Markov Decision Process for Variable Selection in Branch & Bound** has been accepted at **NeurIPS 2025**!

This work introduces [BBMDP](../projects/bbmdp/), a principled vanilla MDP formulation for the branching problem in branch-and-bound (B&B) solvers. Unlike prior RL approaches that relied on approximate TreeMDP formulations — which do not satisfy standard Markov properties — BBMDP models the full state of the search tree, enabling the use of standard RL algorithms with theoretical convergence guarantees.

Our agent, DQN-BBMDP, is evaluated on four standard MILP benchmarks and achieves:
- **27% reduction** in node count vs. prior RL state-of-the-art (DQN-Retro)
- **38% reduction** in solving time
- Strong transfer performance on larger, unseen instances

**Authors:** Paul Strang, Zacharie Alès, Côme Bissuel, Olivier Juan, Safia Kedad-Sidhoum, Emmanuel Rachelson

**Links:** [Project page](../projects/bbmdp/) · [arXiv](https://arxiv.org/abs/2510.19348) · [OpenReview](https://openreview.net/forum?id=05Svr0k5C9) · [Code](https://github.com/abfariah/bbmdp)
