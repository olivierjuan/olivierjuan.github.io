---
layout: page
title: Gyozas
description: A pure-Python reinforcement learning library for combinatorial optimization, built on PySCIPOpt and Gymnasium.
img: assets/img/gyozas/gyozas.png
importance: 3
category: Machine Learning
related_publications: true
years: "2025 – present"
role: "Creator & Lead Developer"
team_size: "1–2 people"
impact: "Open-source RL framework for MILP research"
stack: "Python · PySCIPOpt · Gymnasium · SCIP"
---

**Gyozas** is an open-source reinforcement learning framework designed for combinatorial optimization research. It provides a clean, Gymnasium-compatible interface to train RL agents that make decisions inside SCIP's branch-and-bound solver — specifically **variable selection** (branching) and **node selection**.

The version 1.0 was released in April 2026.

The library is intentionally lightweight, pip-installable in one line:

```bash
pip install gyozas
```

---

## My role

Creator and Lead Developer. The need came from internal EDF work at the intersection of GenAI and optimization, where Ecole was a blocker — difficult to install in a corporate environment and no longer compatible with recent SCIP releases. I wanted to keep Ecole's user-facing surface (observations, rewards, instance generators), so I reimplemented an Ecole-equivalent in pure Python on top of PySCIPOpt.

---

## Motivation

Leading open-source MILP solvers like SCIP rely on hand-crafted heuristics for branching and node selection. Recent research — including [PlanB&B](https://github.com/abfariah/planbb) {% cite Strang2026 %} and [BBMDP](https://github.com/abfariah/bbmdp) {% cite Strang2025 %} — has shown that learned policies can outperform these defaults.

Until 2023, [Ecole](https://github.com/ds4dm/ecole) was the de facto environment for this line of work. Its last release predates SCIP 8, and running BBMDP and PlanB&B made the consequences concrete: incompatibility with current SCIP releases, and friction whenever a new observation or reward had to be prototyped through Ecole's C++ layer.

Gyozas is the response to that experience. It is designed as a **near drop-in replacement for Ecole** — existing scripts typically port over with a handful of line changes — while keeping the user-facing layer in Python, so observation functions, rewards, and instance generators can be customized without touching C++. SCIP 8+ is supported out of the box, and the library exposes **both the Ecole-style interface and a standard Gymnasium interface** from the same environment.

---

## Key features

- **Dual API** — Ecole-compatible interface for porting existing codebases _and_ a standard Gymnasium `reset()`/`step()` interface for new work
- **SCIP 8+ support** — tracks current SCIP / PySCIPOpt releases, where Ecole stopped at SCIP 7
- **Variable selection or node selection** — one decision type per environment (combined mode on the roadmap)
- **Bipartite graph observations** — LP-feature-based observations following Gasse et al. (2019)
- **Pluggable components** — swap reward functions, observation generators, and instance generators in pure Python, no recompilation
- **Built-in problem generators** — Set Cover, Independent Set, Combinatorial Auction, Facility Location (Ecole-compatible signatures)
- **B&B tree visualization** — inspect solver behavior during and after training

---

## Future work

- Performance comparison with Ecole
- Performance tuning (cffi or numba) where profiling identifies bottlenecks
- Port BBMDP and PlanB&B onto Gyozas as reference use cases
- Provide simultaneous Branching and Node selection environment
- Adding more realistic instance generators

---

## Quick start

**Ecole-compatible API** — drop-in for existing Ecole pipelines:

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

A standard Gymnasium interface is also exposed by the same environment — see the [documentation](https://olivierjuan.github.io/gyozas) for the `gymnasium.make(...)` entry points.

---

## Technical stack

**Dynamics**
: Configuring, PrimalSearch, Branching selection, Node Selection

**Reward functions**
: Node count · Solving time · LP iterations · Bound integrals (primal, dual, primal/dual)

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
