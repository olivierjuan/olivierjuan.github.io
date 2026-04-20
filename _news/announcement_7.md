---
layout: post
title: "🥟 Gyozas 🥟 released: a pure-Python RL library for combinatorial optimization"
date: 2026-04-19 09:00:00+0100
inline: false
related_posts: false
---

We are releasing [**Gyozas**](https://github.com/olivierjuan/gyozas), a pure-Python reinforcement learning library for combinatorial optimization, built on top of [PySCIPOpt](https://github.com/scipopt/PySCIPOpt) and [Gymnasium](https://gymnasium.farama.org).

Gyozas provides a clean, Gymnasium-compatible interface to train RL agents that control SCIP's branch-and-bound solver — either for **variable selection** (branching) or **node selection**. It ships with bipartite graph observations, pluggable reward functions, and built-in instance generators for standard combinatorial benchmarks (Set Cover, Independent Set, Combinatorial Auction, Facility Location).

```bash
pip install gyozas
```

**Links:** [GitHub](https://github.com/olivierjuan/gyozas) · [Documentation](https://olivierjuan.github.io/gyozas) · [PyPI](https://pypi.org/project/gyozas)
