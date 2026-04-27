---
layout: page
title: "PlanB&B: Model-Based Reinforcement Learning for Branch and Bound"
description: >
  First model-based RL agent for exact combinatorial optimization.
  PlanB&B learns an internal model of B&B dynamics and uses Gumbel Search (MCTS)
  to discover branching strategies that surpass both prior RL agents and imitation learning.
  AAAI 2026.
img: assets/img/planbb/results.png
importance: 1
category: Machine Learning
related_publications: strang2026planbb
years: "2024 – 2026"
role: "Industrial Supervisor (CIFRE EDF)"
team_size: "6 co-authors"
impact: "AAAI 2026 · Surpasses IL & prior RL agents"
stack: "MCTS · RL · Python · PyTorch · SCIP"
---

**Authors:** Paul Strang, Zacharie Alès, Côme Bissuel, Olivier Juan, Safia Kedad-Sidhoum, Emmanuel Rachelson

**Published:** AAAI 2026 · [arXiv:2511.09219](https://arxiv.org/abs/2511.09219) — November 25, 2025

**Affiliations:** EDF R&D · ENSTA Paris (Institut Polytechnique de Paris) · CNAM Paris (CEDRIC) · ISAE-SUPAERO

**Predecessor:** This work builds directly on the BBMDP formulation introduced at [NeurIPS 2025](../bbmdp/).

---

## My role

Industrial co-supervisor (CIFRE EDF) and problem setter. I defined the EDF application context and research direction, and provided support to the implementation.

---

## Context & Motivation

Despite a decade of effort applying machine learning to Branch and Bound (B&B), reinforcement learning agents have consistently underperformed Imitation Learning (IL). IL learns to mimic the expensive Strong Branching (SB) expert at lower cost, but is fundamentally capped by the quality of that expert. RL, in theory, is bounded only by the maximum achievable performance — yet it has never surpassed IL on standard benchmarks.

A key reason is **exploration and credit assignment**: B&B episodes are long, rewards sparse, and the combinatorial complexity of MILP exacerbates sample inefficiency. The analogy with board games is instructive — RL also struggled in Go and Chess until _model-based planning_ arrived. AlphaZero and MuZero showed that **learning an internal model of the environment and using it for look-ahead search (MCTS) dramatically improves both sample efficiency and final performance**.

This paper asks: can the same paradigm be applied to exact combinatorial optimization?

---

## PlanB&B: Planning in Branch and Bound

<div class="row">
  <div class="col-sm-6 mt-3 mt-md-0">
    {% include figure.liquid loading="eager"
       path="assets/img/planbb/teaser.png"
       title="PlanB&B: planning 3 steps ahead over a learned latent model"
       class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm-6 mt-3 mt-md-0">
    {% include figure.liquid loading="eager"
       path="assets/img/planbb/results.png"
       title="Aggregate normalized solving time across the Ecole benchmark"
       class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Left: PlanB&B simulates k-step subtree rollouts in latent space using networks h, f, g — no LP solve required.
  Right: aggregate normalized solving time (log scale) across four Ecole benchmarks. PlanB&B (100) is the first
  RL agent to surpass IL-DFS and approach IL★.
</div>

PlanB&B adapts the **MuZero** framework to the B&B setting. Its learned model consists of three interconnected networks:

| Network                  | Input → Output                                                  | Role                                                        |
| ------------------------ | --------------------------------------------------------------- | ----------------------------------------------------------- |
| **Representation** $$h$$ | B&B node $$(o, \bar{x}_t) \rightarrow \hat{o} \in \mathcal{H}$$ | Encode current node into latent space                       |
| **Dynamics** $$g$$       | $$(\hat{o}, a) \rightarrow (\hat{o}_l, \hat{o}_r)$$             | Predict child node representations _without solving any LP_ |
| **Prediction** $$f$$     | $$\hat{o} \rightarrow (\hat{p}, \bar{v}, b)$$                   | Policy prior, subtree value, branchability score            |

The **branchability** head $$b \in \{0,1\}$$ is a novel contribution: it predicts whether a generated node will be pruned or will lead to further branching, allowing the model to simulate realistic subtree rollouts and discard dead branches from the imagined tree $$\hat{\mathcal{T}}$$ early.

> **Key difference with MuZero**: MuZero operates on full state observations. PlanB&B operates exclusively on the current B&B node $$(o, \bar{x}_t)$$ — the minimal local information sufficient for optimal branching under DFS (from BBMDP). The dynamics network $$g$$ must therefore learn to approximate LP resolution in latent space, _without_ being explicitly trained to solve LPs.

---

## The MDP Foundation

PlanB&B builds on the BBMDP formulation {% cite strang2025bbmdp %}: the full B&B tree $$s_t = (\mathcal{V}_t, \mathcal{E}_t, \bar{x}_t)$$ is the MDP state, reward $$R(s,a) = -1$$ per branching step, discount $$\gamma = 1$$. Under Depth-First Search, the value function decomposes as:

$$V^\pi(s) = \sum_{o \in \mathcal{O}} \bar{V}^\pi(o, \bar{x}_o)$$

This subtree decomposition makes local planning globally meaningful: minimising each subtree size locally is equivalent to minimising total tree size. It also keeps planning tractable — subtree trajectories have logarithmic length relative to full episodes.

---

## Gumbel Search in B&B

At each branching decision, PlanB&B runs **Gumbel Search** (Danihelka et al., 2022) — an MCTS variant designed for large action spaces and limited simulation budgets. At the root, it combines **Sequential Halving** with the **Gumbel-Top-k trick** to efficiently identify the best action without evaluating every fractional variable.

Each simulation step costs only 1 call to $$g$$ and 2 calls to $$f$$. After $$N$$ simulations, an improved policy target $$\pi_t$$ is produced:

$$\pi'(\hat{\mathcal{T}}, a) = \text{softmax}\!\left(P(\hat{\mathcal{T}}, a) + \sigma\!\left(\tilde{Q}(\hat{\mathcal{T}}, a)\right)\right)$$

balancing the policy prior $$P$$ (from $$f$$) against search-accumulated Q-values. During training, the policy network is distilled from these improved targets — the standard MuZero self-improvement loop.

---

## Training

PlanB&B is trained over **K-step subtree trajectories** $$(s_t, a_t, \ldots, s_{t+K})$$. The model is unrolled for $$K$$ steps and trained with four complementary losses:

$$\mathcal{L}_t(\theta) = \frac{1}{K+1} \sum_{k=0}^{K} \Bigl[\lambda_p \mathcal{L}_p(\pi_{t+k}, \hat{p}^k_t) + \lambda_v \mathcal{L}_v(z_{t+k}, \hat{v}^k_t)\Bigr] + \frac{1}{|\hat{\mathcal{T}}^K|} \sum_{\hat{o} \in \hat{\mathcal{T}}^K} \Bigl[\lambda_b \mathcal{L}_b(b_o, \hat{b}_{\hat{o}}) + \lambda_t \mathcal{L}_t(\check{o}, \hat{o})\Bigr]$$

| Loss              | Purpose                                                                                                   |
| ----------------- | --------------------------------------------------------------------------------------------------------- |
| $$\mathcal{L}_p$$ | Match Gumbel Search policy targets (cross-entropy)                                                        |
| $$\mathcal{L}_v$$ | Predict n-step returns via HL-Gauss classification                                                        |
| $$\mathcal{L}_b$$ | Classify branchable vs. pruned nodes                                                                      |
| $$\mathcal{L}_t$$ | **Tree consistency** (SimSiam-inspired): align imagined node representations with their real counterparts |

The **tree consistency loss** $$\mathcal{L}_t$$ is a new contribution: it enforces structural coherence between imagined subtrees and real B&B trajectories, constraining $$g$$ to capture only the B&B-relevant aspects of node transitions.

The **HL-Gauss value loss** encodes subtree sizes as categorical histogram distributions over a log-scale support $$\psi(z) = \log_2(-z)$$, handling the multi-order-of-magnitude range of B&B tree sizes.

The **training pipeline** follows the parallelised EfficientZero architecture with four worker types (MILP actors, CPU rollout workers, GPU batch workers, learner) running in parallel on a 128-core CPU / 4×A100 GPU server.

---

## Experiments

### Setup

Four standard Ecole MILP benchmarks · SCIP 8.0.3 backend · DFS node selection · no cuts beyond root · 1h time limit. Results: geometric mean over 100 test and 100 transfer instances, averaged over 5 seeds. **Note:** PlanB&B results use the _policy network alone at evaluation time_ (no MCTS budget) for fair comparison.

### Main Results

Aggregate normalized scores (lower is better, PlanB&B = 100):

| Method      | Node (Test) | Time (Test) | Node (Transfer) | Time (Transfer) |
| ----------- | :---------: | :---------: | :-------------: | :-------------: |
| Random      |    1068     |     454     |      8050       |      3870       |
| PG-tMDP     |     272     |     254     |       373       |       290       |
| DQN-tMDP    |     207     |     188     |       787       |       679       |
| DQN-Retro   |     208     |     241     |      1166       |      1254       |
| **PlanB&B** |   **100**   |   **100**   |     **100**     |     **100**     |
| IL-DFS      |     130     |     131     |       151       |       136       |
| IL★         |     96      |     116     |       70        |       83        |
| SCIP        |     58      |     363     |       121       |       132       |

Key findings:

- **2× reduction** in tree size and solving time vs. prior RL state-of-the-art on test instances; **3× on transfer instances**, where the generalization advantage of MBRL is most visible.
- **First RL agent to outperform IL-DFS** on both test and transfer — the first time any RL branching agent surpasses an IL approach trained on Strong Branching.
- On test instances, PlanB&B achieves ≈10% lower solving time than IL★ _despite slightly larger trees_, due to DFS warm-start speedups (10–30% faster LP re-solves along diving paths, see Q4 below).
- SCIP produces fewer nodes than any ML method due to aggressive presolving, but is **3.6× slower** in total solving time.

### Policy Improvement via Planning (Q2)

With additional MCTS budget at inference time, PlanB&B further improves:

<div class="row justify-content-sm-center">
  <div class="col-sm-10 mt-3 mt-md-0">
    {% include figure.liquid loading="eager"
       path="assets/img/planbb/planning.png"
       title="Node and time vs. MCTS simulation budget on MIS"
       class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Maximum Independent Set instances. On test instances (left), PlanB&B beats IL★ in node count at N > 12,
  reaching ≈20% reduction at N = 50. On transfer instances (right), N = 9 already matches IL★ solving time;
  at N = 50, tree size is reduced by ≈50%. Note: value network OOD effects limit planning gains on other benchmarks (CA, SC, MK).
</div>

### Does PlanB&B Simply Imitate Strong Branching? (Q3)

A natural question: does look-ahead planning push PlanB&B _closer_ to SB, effectively recovering IL? The answer is **no**.

| Policy         | SB C-Entropy ↓ | SB Score ↑ | SB Freq. ↑ |
| -------------- | :------------: | :--------: | :--------: |
| SB (oracle)    |      0.00      |    1.00    |    1.00    |
| IL★            |      0.84      |    0.69    |    0.45    |
| PlanB&B (N=0)  |      1.97      |    0.63    |    0.39    |
| PlanB&B (N=50) |      2.08      |    0.65    |    0.40    |

PlanB&B is consistently _further_ from SB than IL★ on all three alignment metrics, and adding MCTS iterations does not increase alignment. PlanB&B surpasses IL not by better imitating the expert, but by **discovering genuinely novel branching strategies**. This directly demonstrates RL's potential to go beyond expert-bounded IL.

### The DFS Trade-off (Q4)

DFS produces larger trees than advanced node selection policies (Best-First, Best-Estimate), which conventionally implies slower solving. However, DFS also enables **warm-start LP acceleration** — when nodes are explored along diving trajectories, the solver can re-use the previous LP basis, reducing transition time by 10–30% across benchmarks. The net impact is a trade-off:

- On **test instances**, warm-start gains often compensate for larger trees → PlanB&B beats IL★ in time.
- On **transfer instances**, closing the primal gap becomes harder at larger scales; DFS trees grow disproportionately, widening the performance gap with non-DFS IL★.

The relative impact of DFS is captured by the _discovery time_ $$t_d$$: the B&B step at which the global optimum is first found. Once $$\bar{x} = x^*$$, all node selection policies become equivalent. For MK transfer instances, $$t_r = t_d/T = 1.0$$ — the optimal solution is found only at the very end — making DFS maximally costly.

---

## Theoretical Connection: Strong Branching as 1-Step MCTS

The paper establishes a clean theoretical connection: **Strong Branching is exactly a depth-1, full-width MCTS** that maximises immediate dual bound improvement, using LP solvers as a perfect one-step dynamics model. PlanB&B generalises this by learning a multi-step approximate dynamics model, enabling deeper look-ahead at a fraction of the computational cost. This unification clarifies _why_ SB is such a strong expert and _how_ model-based RL can eventually go beyond it.

---

## Limitations & Future Directions

- **Value network OOD generalisation**: the value head $$\bar{v}$$ struggles on transfer instances on 3/4 benchmarks (out-of-distribution subtree sizes), limiting planning benefits outside MIS. A progressive training curriculum over instance sizes is the natural fix.
- **DFS requirement**: theoretically necessary for subtree decomposition; escaping it requires global B&B tree representations (e.g. Zhang et al., 2025).
- **Training cost**: several days per benchmark on 4×A100 GPUs.

Directions opened by PlanB&B:

- **Progressive instance curricula** to improve value network OOD generalisation.
- **Global tree encodings** to move beyond DFS.
- **MBRL for cut selection, node selection, primal search** — the BBMDP+Gumbel Search framework generalises directly.

---

## Resources

**arXiv:** [arXiv:2511.09219](https://arxiv.org/abs/2511.09219) ·

**OpenReview:** [AAAI 2026](https://openreview.net/forum?id=8481hfKVsr)

**Code & pretrained models:**

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.liquid repository="abfariah/planbb" %}
</div>

**Predecessor (BBMDP @ NeurIPS 2025):** [project page](../bbmdp/)

---

## Citation

```bibtex
@inproceedings{strang2026planbb,
  title     = {Planning in {Branch-and-Bound}: {Model-Based} {Reinforcement Learning} for {Exact Combinatorial Optimization}},
  author    = {Strang, Paul and Al\`es, Zacharie and Bissuel, C\^ome and Juan, Olivier and Kedad-Sidhoum, Safia and Rachelson, Emmanuel},
  booktitle = {Proceedings of the AAAI Conference on Artificial Intelligence},
  year      = {2026}
}
```
