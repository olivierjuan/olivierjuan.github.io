---
layout: page
title: "PlanB&B: Model-Based RL for Branch-and-Bound"
description: "First model-based RL agent for exact combinatorial optimization — uses MCTS over a learned B&B dynamics model to discover branching strategies that surpass expert heuristics."
img: assets/img/planbb/teaser.png
importance: 1
category: Machine Learning
related_publications: true
---

**Authors:** Paul Strang, Zacharie Alès, Côme Bissuel, Olivier Juan, Safia Kedad-Sidhoum, Emmanuel Rachelson

**Published:** AAAI 2026 — arXiv:2511.09219

**Affiliations:** EDF R&D, ENSTA Paris, CNAM Paris, ISAE-SUPAERO

---

### Abstract

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
    Mixed-Integer Linear Programming (MILP) lies at the heart of many critical industrial problems. Modern solvers rely on Branch-and-Bound (B&B), whose performance is tightly coupled to the branching heuristic used to select which variable to split on at each node.<br/><br/>

    While reinforcement learning (RL) has shown promise for learning branching policies, prior approaches are either limited to imitation learning or use model-free RL, which is sample-inefficient and unable to look ahead. Building on the rigorous MDP formulation introduced in {% cite Strang2025 %}, this work introduces <strong>PlanB&B</strong> {% cite Strang2026 %}, the first <em>model-based</em> RL agent for exact combinatorial optimization.
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/planbb/teaser.png" title="PlanB&B planning in B&B" class="img-fluid rounded z-depth-1" %}
        <div class="caption">Planning 3 steps ahead in B&B over a learned latent model. The networks h, f, and g enable simulating subtree rollouts from the current node without any LP solve.</div>
    </div>
</div>

---

### The Innovation: PlanB&B

PlanB&B learns an **internal model of B&B dynamics** and uses **Monte Carlo Tree Search (MCTS)** to plan improved branching decisions — analogous to how MuZero masters board games, but applied for the first time to exact combinatorial optimization.

The agent is built around three neural networks:

- **Representation network (h)** — encodes a B&B node observation into a compact latent state
- **Dynamics network (g)** — predicts the latent representation of child nodes after a branching decision, without solving any LP
- **Prediction network (f)** — outputs a branching policy prior, a subtree value estimate, and a branchability score

At inference time, **Gumbel Search** (a variant of MCTS) simulates multiple trajectories through this learned model to produce a refined branching policy, all without touching the actual B&B tree.

$$\pi^* = \arg\min_{\pi} \; \mathbb{E}_{P \sim p_0} \left[ \left| \mathrm{BB}_{(\pi, \rho)}(P) \right| \right]$$

---

### Training

PlanB&B is trained on K-step subtree trajectories with four complementary loss terms:

| Loss | Role |
|------|------|
| **Policy loss** | Match search-improved policy targets from Gumbel Search |
| **Value loss** | Predict n-step returns via HL-Gauss classification |
| **Branchability loss** | Distinguish branchable nodes from pruned ones |
| **Tree consistency loss** | Enforce hierarchical consistency between real and imagined subtrees (SimSiam-inspired) |

---

### Results

Evaluated on four standard MILP benchmarks from the Ecole library: **Set Covering (SC)**, **Combinatorial Auctions (CA)**, **Maximum Independent Set (MIS)**, and **Multiple Knapsack (MK)**. Scores are normalized so that **lower is better**, with PlanB&B set as the reference (100).

| Method | Norm. Node Count | Norm. Solving Time |
|--------|----------------:|-------------------:|
| **PlanB&B** | **100** | **100** |
| DQN-tMDP | 207 | 188 |
| DQN-Retro | 208 | 241 |
| PG-tMDP | 272 | 254 |

| **Non-RL** | **Norm. Node Count** | **Norm. Solving Time** |
| IL★ (strong branching imitation) | 96 | 116 |
| IL (DFS) | 130 | 131 |
| SCIP (default) | 58 | 363 |

> **Note:** SCIP has fewer nodes due to aggressive presolving, but is 3.6× slower in wall time.

Key findings:
- **2–3× improvement over RL baselines** in both tree size and solving time
- **First RL agent to surpass imitation learning** trained on strong branching (DFS variant)
- With additional search budget at inference, PlanB&B achieves up to **50% tree size reduction** on transfer instances
- Discovers **genuinely novel branching strategies** — actively diverging from expert patterns while achieving superior performance

---

### Resources

- **arXiv**: [arXiv:2511.09219](https://arxiv.org/abs/2511.09219)
- **OpenReview**: [AAAI 2026 paper](https://openreview.net/forum?id=8481hfKVsr)
- **Predecessor (BBMDP @ NeurIPS 2025)**: [project page](../bbmdp/)

---

### Citation

If you use this work, please cite:

```bibtex
@inproceedings{strang2026planbb,
  title     = {Planning in {Branch-and-Bound}: {Model-Based} {Reinforcement Learning}
               for {Exact Combinatorial Optimization}},
  author    = {Strang, Paul and Al{\`e}s, Zacharie and Bissuel, C{\^o}me and Juan, Olivier
               and Kedad-Sidhoum, Safia and Rachelson, Emmanuel},
  booktitle = {Association for the Advancement of Artificial Intelligence (AAAI)},
  year      = {2026}
}
```
