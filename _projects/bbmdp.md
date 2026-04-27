---
layout: page
title: "BBMDP: A Markov Decision Process for Variable Selection in Branch & Bound"
description: >
  A principled vanilla MDP formulation for learning optimal branching strategies in MILP solvers,
  unlocking k-step RL algorithms previously incompatible with the TreeMDP framework.
  New state-of-the-art among RL agents on the Ecole benchmark. NeurIPS 2025.
img: assets/img/bbmdp/results.png
importance: 2
category: Machine Learning
related_publications: strang2025bbmdp
years: "2023 – 2025"
role: "Industrial Supervisor (CIFRE EDF)"
team_size: "6 co-authors"
impact: "NeurIPS 2025 · SOTA on Ecole benchmark"
stack: "RL · Python · PyTorch · PySCIPOpt · SCIP"
---

**Authors:** Paul Strang, Zacharie Alès, Côme Bissuel, Olivier Juan, Safia Kedad-Sidhoum, Emmanuel Rachelson

**Published:** NeurIPS 2025 · [arXiv:2510.19348](https://arxiv.org/abs/2510.19348) — October 22, 2025

**Affiliations:** EDF R&D · ENSTA Paris (Institut Polytechnique de Paris) · CNAM Paris (CEDRIC) · ISAE-SUPAERO

---

## My role

Industrial co-supervisor (CIFRE EDF) and problem setter. I defined the EDF application context and research direction, and provided support to the implementation.

---

## Context & Motivation

Reinforcement Learning is a natural fit for Branch and Bound (B&B): branching decisions are sequential, the objective is global, and the agent can improve over time by solving repeated instances of the same problem family. Yet, despite a decade of effort, **RL branching agents have consistently fallen short of Imitation Learning (IL) approaches**, which simply learn to mimic the expensive Strong Branching (SB) expert.

A key reason is theoretical: prior RL approaches relied on **TreeMDP**, an MDP-_inspired_ but not MDP-_compliant_ formulation. TreeMDP defines a branching action as producing _two_ next states (the two child nodes), which violates the fundamental Markov property of standard MDPs. This forces ad hoc Bellman updates and ad hoc convergence proofs, and — more critically — **breaks compatibility with multi-step RL algorithms** that are central to state-of-the-art deep RL.

This paper asks: can we define a _proper_ MDP for B&B branching, and does it help in practice?

---

## The Problem with TreeMDP

<div class="row justify-content-sm-center">
  <div class="col-sm-10 mt-3 mt-md-0">
    {% include figure.liquid loading="eager"
       path="assets/img/bbmdp/structure.png"
       title="BBMDP vs TreeMDP: 1-step and k-step Bellman equations"
       class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  At 1-step, BBMDP and TreeMDP yield identical Bellman updates (a, b). But for k-step returns,
  they diverge: BBMDP produces a valid linear trajectory (c), while TreeMDP's k-step target
  expands into 2<sup>k</sup> pseudo-states (d) that cannot be produced by any real DFS execution.
</div>

In TreeMDP, the state is the current B&B node $$s_i = (P_i, x^*_{LP,i}, \bar{x}_i)$$ and a branching action yields _two_ child states $$(s^-_i, s^+_i)$$. This is not a stochastic process on a state variable — it is a tree process — and therefore not a Markov Decision Process. The consequences are concrete:

- **k-step TD learning is broken**: applying the tree Bellman operator k times yields a target that sums $$2^k$$ pseudo-states, a trajectory that cannot be produced by any actual DFS execution of B&B. The resulting targets are biased approximations of true B&B dynamics.
- **MCTS is incompatible**: TreeMDP's UCT criterion balances exploration between $$s^-_i$$ and $$s^+_i$$, but since branching on $$a_i$$ _necessarily_ adds both children to the tree, this comparison is meaningless.
- **Ad hoc theory**: every RL algorithm (TD(0), value iteration, policy gradient) requires a custom convergence proof within TreeMDP, limiting reuse of the broader RL literature.

---

## BBMDP: A Principled Formulation

BBMDP defines variable selection as a standard deterministic MDP $$(S, A, \mathcal{T}, p_0, R)$$:

| Component          | BBMDP                                                     | TreeMDP                                       |
| ------------------ | --------------------------------------------------------- | --------------------------------------------- |
| **State** $$s_t$$  | Full B&B tree $$(V_t, E_t, \bar{x}_t)$$                   | Current node $$(P_i, x^*_{LP,i}, \bar{x}_i)$$ |
| **Action** $$a_t$$ | Integer variable index $$\in \mathcal{I}$$                | Integer variable index $$\in \mathcal{I}$$    |
| **Transition**     | $$s_{t+1} = \kappa_\rho(s_t, a_t)$$ — a single next state | Two child states $$(s^-_i, s^+_i)$$           |
| **Reward**         | $$R(s,a) = -2$$ (two nodes added per branching)           | $$-1$$ or $$-2$$ per child                    |
| **Is an MDP?**     | **Yes**                                                   | No                                            |

The key insight is that the **state must include the entire B&B tree**, not just the current node. This is the minimal information needed for the Markov property to hold: the future trajectory depends on all open nodes and the current incumbent solution, not just the node being branched on right now.

### Why DFS is still needed

Even with a proper MDP, the value of a subtree rooted at open node $$o_i$$ depends on which other subtrees are explored before it (they may improve the incumbent $$\bar{x}$$, enabling earlier pruning). The only node selection policy $$\rho$$ that isolates subtrees from each other — making subtree size predictable from local information — is **Depth-First Search**. This recovers the computational tractability of TreeMDP without sacrificing MDP correctness.

Under DFS-BBMDP, **Proposition 3.1** establishes the refined Bellman equation:

$$\bar{V}^\pi(o_1, \bar{x}_{o_1}) = -2 + \bar{V}^\pi(o'_1, \bar{x}_{o'_1}) + \bar{V}^\pi(o'_2, \bar{x}_{o'_2})$$

where $$o'_1$$ and $$o'_2$$ are the two children of the branched node. This is structurally identical to the 1-step TreeMDP equation — explaining why TD(0) TreeMDP and BBMDP are equivalent — but it lives inside a proper MDP, unlocking multi-step extensions:

$$\bar{V}^\pi(o_1, \bar{x}_{o_1}) = -2k + \sum_{i=1}^{k+1} \bar{V}^\pi\!\left(o^{(k)}_i, \bar{x}_{o^{(k)}_i}\right)$$

This k-step target follows a real DFS trajectory of length $$k$$, visiting $$k+1$$ open nodes — not a fictitious tree of $$2^k$$ branches.

---

## DQN-BBMDP Agent

### State Representation

Following Gasse et al. [2019], B&B nodes are represented as **bipartite constraint–variable graphs** $$G = (V_G, C_G, E_G)$$, processed by a Graph Convolutional Network (GCN). DQN-BBMDP enriches this with the additional node features introduced by Parsonson et al. [2022].

### HL-Gauss Classification Loss

Drawing on Farebrother et al. [2024], BBMDP replaces the standard MSE regression loss with a **HL-Gauss cross-entropy loss** that encodes Q-values as categorical histogram distributions. This is non-trivial in the B&B setting: value functions span several orders of magnitude ($$Z_v = [-10^6, -2]$$). The solution is to partition the log-scale support $$\psi(Z_v)$$ with $$\psi(z) = \log_2(-z)$$, placing bin centres at $$\zeta_i = i$$ and converting back via $$\psi^{-1}(z) = -2^z$$.

### Training Pipeline (Rainbow-DQN)

The full agent combines:

- **k-step return** ($$k=3$$) temporal difference
- **Double DQN** for stable target estimation
- **Prioritized Experience Replay (PER)**
- **Boltzmann + ε-greedy** exploration

```
for t = 0, …, N-1:
  1. Draw instance P ~ p₀.
  2. Solve P with ε-greedy / Boltzmann policy qθ_t.
  3. Collect k-step transitions (sᵢ, aᵢ, Σrᵢ₊ⱼ, sᵢ₊ₖ) → replay buffer.
  4. Update θ via HL-Gauss loss on batches from the replay buffer.
```

---

## Experiments

### Benchmarks & Setup

Evaluated on four standard MILP benchmarks from the **Ecole** library, using SCIP 8.0.3 as backend (DFS, no cuts beyond root, no restarts). Models are trained on fixed-dimension instances and evaluated on both same-size **test instances** and larger **transfer instances**.

| Benchmark               | Train / Test           | Transfer                |
| ----------------------- | ---------------------- | ----------------------- |
| Set Covering            | 500 items, 1000 sets   | 1000 items, 1000 sets   |
| Combinatorial Auction   | 100 items, 500 bids    | 200 items, 1000 bids    |
| Maximum Independent Set | 500 nodes              | 1000 nodes              |
| Multiple Knapsack       | 100 items, 6 knapsacks | 100 items, 12 knapsacks |

### Results

<div class="row justify-content-sm-center">
  <div class="col-sm-8 mt-3 mt-md-0">
    {% include figure.liquid loading="eager"
       path="assets/img/bbmdp/results.png"
       title="Normalized scores across the Ecole benchmark"
       class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Normalized scores (log scale) aggregated across all four benchmarks on transfer instances.
  DQN-BBMDP (score 100) dominates all prior RL agents and narrows the gap with IL.
</div>

On **test instances**, DQN-BBMDP achieves best aggregate performance among all RL agents:
**−27% nodes and −38% solving time** vs. the previous RL state-of-the-art (DQN-Retro).

On **transfer instances** (harder, out-of-distribution size), the advantage is even larger — BBMDP is the **first RL agent to robustly generalise to higher-dimensional instances**, outperforming SCIP on 3 out of 4 benchmarks. This aligns with the theoretical advantage of a principled MDP formulation over TreeMDP.

Full results (geometrical means over 100 test instances, 5 seeds):

| Method        | Norm. Score (Test) | Norm. Score (Transfer) |
| ------------- | ------------------ | ---------------------- |
| Random        | 995                | 5555                   |
| PG-tMDP       | 233                | 298                    |
| DQN-tMDP      | 151                | 439                    |
| DQN-Retro     | 137                | 494                    |
| **DQN-BBMDP** | **100**            | **100**                |
| IL            | 82                 | 39                     |

> Note: normalized scores are relative to DQN-BBMDP (lower is better). IL remains the reference on in-distribution test instances, but BBMDP closes the gap significantly and surpasses IL on transfer for several benchmarks.

### Ablation Study

| Configuration             | k=1 nodes    | k=3 nodes       |
| ------------------------- | ------------ | --------------- |
| DQN-BBMDP (full)          | 158.9        | **152.3 (−4%)** |
| w/o HL-Gauss              | 175.8 (+10%) | 172.3 (+8%)     |
| w/o BBMDP (= DQN-TreeMDP) | 158.9 (+0%)  | 178.9 (+13%)    |
| w/o DFS                   | 156.2 (−2%)  | 150.1 (−5%)     |

Key findings:

- **HL-Gauss** contributes the largest single gain (+10% without it).
- **k-step returns improve BBMDP but hurt TreeMDP** (+13% at k=3), directly confirming the theoretical claim that TreeMDP k-step targets are inconsistent with real B&B dynamics.
- DFS is not restrictive in practice, contrary to prior claims.

---

## Theoretical Contributions

**Proposition 3.1** — Under DFS-BBMDP, the global optimal branching policy $$\pi^*$$ satisfies:

$$\pi^*(s) = \arg\max_{a \in A}\ Q^*(s, a) = \arg\max_{a \in A}\ \bar{Q}^*(o_1, \bar{x}_{o_1}, a)$$

i.e., optimising locally over the current subtree is _equivalent_ to global optimality. This result extends the subtree consistency property of Etheve et al. [2020] to the rigorous MDP setting, while also enabling k-step returns, TD(λ), and MCTS-based policy improvement — none of which were compatible with TreeMDP.

---

## Limitations & Future Directions

- **RL still lags IL on pure node count** on some benchmarks; the generalization gap is closing but not yet closed.
- **DFS requirement**: while shown not to be restrictive in practice here, future work could explore removing this constraint.
- **Convergence not reached** on multiple knapsack after 200k steps, suggesting further gains with longer training.

Promising extensions opened by the principled BBMDP formulation:

- **MCTS-based policy improvement** (e.g., AlphaZero-style) — now theoretically compatible.
- **TD(λ) and full Rainbow** — directly applicable without custom convergence proofs.
- **Cut selection and primal search** — BBMDP generalises to other B&B heuristics by adapting the action set and reward.
- **Distribution shift / domain adaptation** — the MDP foundation provides a cleaner basis for addressing out-of-distribution generalisation.

---

## Resources

**arXiv:** [arXiv:2510.19348](https://arxiv.org/abs/2510.19348)

**Code & pretrained models:**

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.liquid repository="abfariah/bbmdp" %}
</div>

---

## Citation

```bibtex
@inproceedings{strang2025bbmdp,
  title     = {A Markov Decision Process for Variable Selection in Branch \& Bound},
  author    = {Strang, Paul and Al\`es, Zacharie and Bissuel, C\^ome and
               Juan, Olivier and Kedad-Sidhoum, Safia and Rachelson, Emmanuel},
  booktitle = {Advances in Neural Information Processing Systems (NeurIPS)},
  year      = {2025}
}
```
