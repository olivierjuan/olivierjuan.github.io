---
layout: page
title: Gyozas
description: A pure-Python reinforcement learning library for combinatorial optimization, built on PySCIPOpt and Gymnasium.
img: assets/img/gyozas/gyozas.png
importance: 3
category: Machine Learning
related_publications: true
years: "2024 – present"
role: "Creator & Lead Developer"
team_size: "1–2 people"
impact: "Open-source RL framework for MILP research"
stack: "Python · PySCIPOpt · Gymnasium · SCIP"
---

**Gyozas** is an open-source reinforcement learning framework designed for combinatorial optimization research. It provides a clean, Gymnasium-compatible interface to train RL agents that make decisions inside SCIP's branch-and-bound solver — specifically **variable selection** (branching) and **node selection**.

The library is intentionally lightweight: pure Python, no C++ compilation, installable in one line:

```bash
pip install gyozas
```

---

## Motivation

State-of-the-art MILP solvers like SCIP rely on hand-crafted heuristics for branching and node selection. Recent research — including [PlanB&B](https://github.com/abfariah/planbb) {% cite Strang2026 %} and [BBMDP](https://github.com/abfariah/bbmdp) {% cite Strang2025 %} — has shown that learned policies can outperform these defaults. Gyozas provides the environment infrastructure needed to run those experiments without the friction of compiling C++ bindings or reimplementing standard RL wrappers.

Unlike [Ecole](https://github.com/ds4dm/ecole) (last updated 2023), Gyozas is actively maintained and targets Python 3.10+ with SCIP 8+.

---

## Key features

- **Gymnasium-style API** — standard `reset()` / `step()` / `render()` interface, compatible with any Gymnasium-based RL framework
- **Dual control modes** — branch on variables _or_ select nodes in the B&B tree
- **Bipartite graph observations** — LP-feature-based observations following established academic practice
- **Pluggable components** — swap reward functions, observation generators, and instance generators independently
- **Built-in problem generators** — Set Cover, Independent Set, Combinatorial Auction, Facility Location
- **B&B tree visualization** — inspect solver behavior during and after training

---

## Quick start

```python
import gyozas

instances = gyozas.SetCoverGenerator(n_rows=100, n_cols=200)
env = gyozas.Environment(
    instance_generator=instances,
    observation_function=gyozas.NodeBipartite(),
    reward_function=gyozas.NNodes(),
)

obs, action_set, reward, done, info = env.reset()
while not done:
    action = action_set[0]
    obs, action_set, reward, done, info = env.step(action)
```

---

## Technical stack

**Core dependencies**
: PySCIPOpt · Gymnasium · Python 3.10+

**Reward functions**
: Node count · Solving time · LP iterations · Bound integrals

**Problem generators**
: Set Cover · Independent Set · Combinatorial Auction · Facility Location

---

## Links

- [GitHub](https://github.com/olivierjuan/gyozas)
- [Documentation](https://olivierjuan.github.io/gyozas)
- [PyPI](https://pypi.org/project/gyozas)

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.liquid repository="olivierjuan/gyozas" %}
</div>
