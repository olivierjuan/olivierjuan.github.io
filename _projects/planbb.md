---
layout: page
title: "PlanB&B: Model-Based RL for Branch-and-Bound"
description: MuZero for CO. First MBRL agent for exact combinatorial optimization, achieving state-of-the-art performance on MILP benchmarks
img: /assets/img/planbb/teaser.jpg
importance: 1
category: Machine Learning
---

**Authors:** Paul Strang, Zacharie Alès, Côme Bissuel, Olivier Juan, Safia Kedad-Sidhoum, Emmanuel Rachelson

**Published:** arXiv:2511.09219v3 [cs.LG] 25 Nov 2025

**Affiliations:** EDF R&D, ENSTA Paris, CNAM Paris, ISAE-SUPAERO

---

### Abstract

This work introduces **Plan-and-Branch-and-Bound (PlanB&B)** presented at **AAAI 2026**, a novel model-based reinforcement learning (MBRL) agent designed to improve variable selection strategies in branch-and-bound (B&B) algorithms for solving Mixed-Integer Linear Programming (MILP) problems. Unlike previous approaches that rely on imitation learning or model-free RL, PlanB&B learns an internal model of B&B dynamics and uses Monte Carlo Tree Search (MCTS) to discover improved branching strategies through planning.

Using previous results from {% cite Strang2025 %}, 

### Key Contributions

1. **First MBRL Agent for Exact CO**: PlanB&B is the first model-based reinforcement learning agent specifically designed for exact combinatorial optimization problems, extending MCTS-based techniques beyond board games to MILP solving.

2. **Value-Equivalent Dynamics Model**: Rather than explicitly learning to solve linear programs, PlanB&B learns an abstract, value-equivalent MDP that captures only the aspects of B&B essential for policy improvement via planning.

3. **State-of-the-Art Performance**: Achieves 2× improvements in both tree size and solving time compared to prior RL baselines, and outperforms imitation learning approaches on multiple benchmarks.

4. **Novel Strategies Beyond Experts**: Analysis shows PlanB&B discovers genuinely novel branching strategies rather than simply imitating strong branching, actively diverging from expert patterns while achieving superior performance.

### Methodology

#### Model Architecture

PlanB&B consists of three interconnected neural networks:

- **Representation Network (h)**: Maps B&B node observations to internal latent representations
- **Dynamics Network (g)**: Predicts internal representations of child nodes after branching decisions
- **Prediction Network (f)**: Outputs policy priors, subtree value estimates, and branchability scores

The model learns to simulate subtree trajectories in latent space, enabling planning without expensive LP solves.

<div class="row"> <div class="col-sm mt-3 mt-md-0"> {% include figure.liquid loading="eager" path="assets/img/planbb/teaser.png" title="PlanBB Structure" class="img-fluid rounded z-depth-1" %} </div> </div> <div class="caption"> Planning in B&B over a learned model. The combined use of h, f, and g allows simulating subtree rollouts starting from the current B&B node. Here, we show how the planning is done 3 steps ahead. </div>


#### Planning with Gumbel Search

PlanB&B integrates Gumbel Search (a variant of MCTS) to generate improved policy targets. At each branching decision, the agent:

1. Simulates multiple trajectories through the learned model
2. Balances exploration and exploitation using policy and value estimates
3. Produces refined branching policies that guide action selection

#### Training Approach

The agent is trained on K-step subtree trajectories with multiple loss components:

- **Policy loss**: Matches search-improved policy targets
- **Value loss**: Predicts n-step returns using classification (HL-Gauss)
- **Branchability loss**: Discriminates between branchable and pruned nodes
- **Tree consistency loss**: Enforces structural consistency between real and imagined subtrees

### Experimental Results

#### Benchmarks

Evaluated on four standard MILP benchmarks from the Ecole library:
- Set Covering (SC)
- Combinatorial Auctions (CA)
- Maximum Independent Set (MIS)
- Multiple Knapsack (MK)

#### Performance Comparison

On aggregate test instances:

| Method | Normalized Node Count | Normalized Time |
|--------|----------------------:|-----------------:|
| PlanB&B | **100** | **100** |
| DQN-Retro | 208 | 241 |
| DQN-tMDP | 207 | 188 |
| PG-tMDP | 272 | 254 |
| IL (DFS) | 130 | 131 |
| IL★ | 96 | 116 |
| SCIP | 58 | 363 |

#### Key Findings

1. **2-3× Improvement over RL Baselines**: PlanB&B achieves substantial reductions in both tree size and solving time compared to previous RL approaches.

2. **Surpasses Imitation Learning**: First RL-based approach to outperform IL agents trained on strong branching under depth-first search.

3. **Planning Enables Further Improvement**: With additional search budget at inference time, PlanB&B achieves up to 20-50% tree size reduction on test/transfer instances.

4. **Superior Generalization**: Demonstrates strong transfer performance on higher-dimensional instances, maintaining competitive solving times.

### Technical Innovations

1. **Local Observation Planning**: Unlike traditional MuZero which requires full state observations, PlanB&B plans using only local node information by recursively predicting child node representations.

2. **Branchability Prediction**: Introduces a novel prediction head to distinguish nodes leading to branching from those destined to be pruned, enabling accurate subtree simulation.

3. **HL-Gauss Value Learning**: Employs histogram-based value representation with Gaussian smoothing, improving scalability and performance in the high-dimensional value space.

4. **Tree Consistency Loss**: Novel self-supervised loss enforces hierarchical consistency between consecutive B&B node representations using SimSiam-inspired architecture.

### Implications

This work demonstrates that:

- Branching dynamics in B&B can be approximated with sufficient fidelity to enable policy improvement through planning
- MBRL can overcome the sample efficiency and performance limitations of model-free RL in combinatorial optimization
- Planning-based approaches can discover strategies that surpass expert heuristics like strong branching

The success of PlanB&B opens new avenues for applying model-based reinforcement learning to exact combinatorial optimization and suggests potential for MBRL in broader operational research domains.

### Future Directions

The paper identifies depth-first search as a limitation for higher-dimensional problems, suggesting future work should:
- Develop scalable observation functions for encoding evolving B&B trees
- Move beyond local node representations to enable planning with advanced node selection policies
- Explore applications to other B&B heuristics (node selection, cut selection, primal search)

---

### Resources

- **arXiv**: [arXiv:2511.09219](https://arxiv.org/abs/2511.09219)

### Citation

```bibtex
@article{strang2025planning,
  title={Planning in Branch-and-Bound: Model-Based Reinforcement Learning for Exact Combinatorial Optimization},
  author={Strang, Paul and Al{\`e}s, Zacharie and Bissuel, C{\^o}me and Juan, Olivier and Kedad-Sidhoum, Safia and Rachelson, Emmanuel},
  journal={arXiv preprint arXiv:2511.09219},
  year={2025}
}
```
