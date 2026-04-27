---
layout: page
title: "Influence Branching for Learning to Solve MIPs Online"
description: >
  A graph-oriented variable selection strategy combined with Thompson sampling
  to learn branching heuristics online across sequences of similar MIP instances.
  Submitted to the 20th Mixed Integer Program Workshop computational competition.
importance: 5
category: Machine Learning
related_publications: Strang2025a
years: "2023 – 2025"
role: "Industrial Supervisor & Problem Setter"
team_size: "6 co-authors"
impact: "MIPcc23 competition · 3–6% speedup over SCIP"
stack: "Python · PySCIPOpt · Thompson Sampling · SCIP"
---

**Authors:** Paul Strang, Zacharie Alès, Côme Bissuel, Olivier Juan, Safia Kedad-Sidhoum, Emmanuel Rachelson

**Preprint:** [arXiv:2510.04273](https://arxiv.org/abs/2510.04273) — October 2025

**Affiliations:** EDF R&D · ENSTA Paris (Institut Polytechnique de Paris) · CNAM Paris (CEDRIC) · ISAE-SUPAERO

**Context:** Entry to the [MIPcc23](https://www.mixedinteger.org/2023/) — 20th Mixed Integer Program Workshop computational competition.

---

## My role

Industrial co-supervisor (CIFRE EDF) and problem setter. I defined the EDF application context and research direction, and provided guidance on the problem formulation.

---

## Context & Motivation

Most machine learning approaches to MIP solving are designed offline: a model is trained on a large set of instances and deployed without further adaptation. The MIPcc23 competition posed a different challenge — **online learning**: given a stream of 50 instances drawn from the same distribution, adapt the solver strategy across the sequence to minimize total solving time.

This online setting rules out standard RL training loops (too expensive per instance) and supervised learning (no labelled data). It naturally maps to a **multi-armed bandit** problem: at each instance, pick a branching strategy, observe the solving score, and update beliefs.

The key open question is: **what branching strategies are worth putting in the action set?** We introduce _Influence Branching_, a lightweight graph-based heuristic that leverages the MIP's structure to make early branching decisions, and show it can be meaningfully optimized online.

---

## Influence Branching

### Core Idea

Rather than learning a full branching policy end-to-end, influence branching captures structural information from the MIP formulation through an **influence graph**: a weighted graph where edge $$(i,j)$$ measures how much variables $$i$$ and $$j$$ interact through shared constraints.

The branching rule selects, at each node within the first $$k$$ levels of the B&B tree, the variable with the **maximum total influence** — the variable most entangled with others. At deeper nodes, control returns to SCIP's default heuristic.

### The Six Influence Models

Six formulas quantify how constraint $$l$$ contributes to the local influence of the pair $$(i,j)$$:

| Model           | Formula                                           | Intuition                                  |
| --------------- | ------------------------------------------------- | ------------------------------------------ |
| **Count**       | $$\mathbf{1}_{A_{li}} \cdot \mathbf{1}_{A_{lj}}$$ | Raw co-occurrence in constraints           |
| **Binary**      | Normalized count by total co-occurrences          | Scale-invariant co-occurrence              |
| **Dual**        | Count weighted by dual value $$\vert y_l^*\vert$$ | Active, tight constraints matter more      |
| **Countdual**   | Count restricted to active dual constraints       | Sparse dual-guided selection               |
| **Auxiliary**   | Weighted by distance to variable bounds           | Prefers fractional, near-bound variables   |
| **Adversarial** | Coefficient ratio weighted by dual activity       | Captures adversarial variable interactions |

### Normalization

To handle problem rescaling (different units, magnitudes), the method normalizes the constraint matrix $$A$$, right-hand side $$b$$, and objective $$c$$ before computing influence weights. This makes the heuristic invariant to affine rescaling of any variable or constraint — a practical requirement often ignored in academic branching literature.

### Integration with SCIP

Implemented as a `pyscipopt.BranchRule`, influence branching intercepts branching decisions at depths $$0, \ldots, k-1$$ and delegates to SCIP defaults for deeper nodes. This focus on the **root neighbourhood** keeps per-instance overhead negligible while still shaping the early structure of the B&B tree.

---

## Online Learning via Multi-Armed Bandits

### Action Space

With only 50 instances per series, the sample budget constrains the size of the action set. The five best-performing $$(g, k)$$ pairs (influence model $$g$$, depth limit $$k$$) were pre-selected by sweeping over public datasets:

$$\{(\text{count},1),\ (\text{count},5),\ (\text{countdual},2),\ (\text{binary},3),\ (\text{dual},3)\}$$

### Thompson Sampling

Each arm $$a$$ maintains a normal reward model $$\mathcal{N}(\hat{\mu}_a, \sigma^2)$$ with fixed $$\sigma = 0.2$$. At each instance, arms are sampled independently and the arm with the lowest sample is selected (minimizing solving score). After solving, the empirical mean $$\hat{\mu}_a$$ is updated by Bayesian averaging.

**Performance metric:** $$f_{s,i} = \text{reltime} + \text{gap at time limit} + \text{infeasibility penalty}$$

where reltime is solving time relative to default SCIP. Negative speedup = improvement over SCIP.

### Thompson Sampling vs UCB2

On 10,000 shuffled trial runs, Thompson sampling converges to the oracle arm faster than UCB2, achieving **convergence scores of 64–75% across all public series** — meaning the algorithm recovers more than half the theoretical speedup of always picking the best arm from the start.

---

## Experimental Results

Results averaged over 2,000 simulated runs on the public competition series (negative = faster than SCIP):

| Series                   | Speedup            |
| ------------------------ | ------------------ |
| bnd series 1             | −0.031 ± 0.009     |
| bnd series 2             | −0.037 ± 0.020     |
| obj series 1             | −0.022 ± 0.006     |
| obj series 2             | **−0.052 ± 0.022** |
| rhs series 1             | −0.048 ± 0.027     |
| rhs series 2             | +0.001 ± 0.000     |
| mat rhs bnd obj series 1 | **−0.061 ± 0.021** |

**Key findings:**

- Largest gains on hard instances with large B&B trees (10,000+ nodes): early branching decisions have high leverage when the tree is deep.
- No improvement on `rhs series 2`: these instances are easy enough that SCIP's default strategy leaves little room for improvement.
- The most general series (`mat rhs bnd obj`) — where the constraint matrix, RHS, and objective all vary — yields the strongest speedup, suggesting the method generalizes beyond fixed-structure distributions.
- Even suboptimal arm selections yield meaningful speedups, making the algorithm robust to slow convergence.

---

## Relation to Other Work

This paper is the earliest in a series of EDF CIFRE contributions applying machine learning to Branch & Bound:

| Paper                               | Venue        | Key contribution                                           |
| ----------------------------------- | ------------ | ---------------------------------------------------------- |
| **Influence Branching** (this work) | arXiv 2025   | Online bandits + graph heuristic for MIPcc23               |
| [BBMDP](../bbmdp/)                  | NeurIPS 2025 | Principled MDP formulation for B&B, enabling multi-step RL |
| [PlanB&B](../planbb/)               | AAAI 2026    | Model-based RL with MCTS for B&B                           |

Influence branching can be seen as a fast, interpretable heuristic baseline — one that does not require neural networks, GPU training, or offline datasets — against which the heavier RL agents of BBMDP and PlanB&B are benchmarked.

---

## Citation

```bibtex
@misc{Strang2025a,
  author        = {Strang, Paul and Al{\`e}s, Zacharie and Bissuel, C{\^o}me and
                   Juan, Olivier and Kedad-Sidhoum, Safia and Rachelson, Emmanuel},
  title         = {Influence branching for learning to solve mixed-integer programs online},
  year          = {2025},
  archiveprefix = {arXiv},
  eprint        = {2510.04273},
  primaryclass  = {cs.LG},
  doi           = {10.48550/arxiv.2510.04273},
}
```
