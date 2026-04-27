---
layout: page
title: Apogène
description: EDF's short-term unit commitment software — from ground-up redesign to production deployment at scale.
img: assets/img/apogene/eod.png
importance: 1
category: Optimization
related_publications: true
years: "2012 – present"
role: "Project Manager & Tech Lead"
team_size: "10–25 people"
impact: "Tens of millions €/year in cost savings"
stack: "MILP · C++ · Python · CPLEX · Gurobi · HPC"
---

**Apogène** is EDF's short-term unit commitment software, used to schedule a fleet of power plants over a short horizon (2-3 days ahead) to meet electricity demand at minimum cost, subject to complex operational constraints (ramp rates, minimum up/down times, fuel limits, reserve requirements). At EDF's scale, even marginal efficiency gains translate to tens of millions of euros annually.

Today, Apogène is deployed and used at 3 different time horizons: Weekly (4 weeks), daily (2-3 days), and intra-day (24 hours). Apogène optimizes all of EDF's production assets: 57 nuclear units, 4 Combined Cycle Gas plants, 13 Combustion Turbines, and 450 hydraulic generation units (lock-based and run-of-river). It also takes into account the different electricity markets. Recently, the group's stationary batteries have also been optimized by Apogène.

---

## My role

In 2012, I took on the project management of a complete ground-up redesign. I led the team through the full software lifecycle:

- Mathematical modeling of the unit commitment problem as a **MILP**
- Algorithm design: **branch-and-bound**, cutting planes, decomposition methods, **ADMM**, **Resource Constrained Shortest Path** (RCSP), **Min Cost Flow**
- Software architecture and **C++** implementation
- Multi-solver integration: **CPLEX**, **GUROBI**, **SCIP**, **FICO Xpress**, **COIN-OR**, **HiGHS**
- Production infrastructure: **HPC / Slurm / Singularity**
- CI/CD pipeline (**GitLab**), Docker, testing strategy, and release management
- Technical leadership of a team of **10–25 engineers and researchers**

---

## Timeline

| Year      | Milestone                                                                                                       |
| --------- | --------------------------------------------------------------------------------------------------------------- |
| 2008–2012 | Early contributions to Apogène V1; parallel work on LNG scheduling, maintenance planning, emission optimization |
| 2012      | Took over as project manager; started ground-up redesign of mathematical model and solver architecture          |
| 2018      | **V2 deployed in production** — significant reduction in operating costs vs. V1                                 |
| 2020      | **V3 deployed** — improved decomposition, RCSPP integration, Complete C++ migration.                            |
| 2021      | **V4 deployed** — improved performances, asymmetrical ancillary services integration                            |
| 2022      | **IndusRO'2022 prize** awarded by ROADEF for industrial impact                                                  |
| 2024      | **Intraday Deployment** - Adaptation for the intraday unit-commitment optimization.                             |
| 2025      | **V5 deployed** - improved decomposition algorithm, performance improvement for timestep refinement             |
| 2025      | **Hebdo Deployment** - Adaptation for the weekly unit-commitment optimization.                                  |
| 2025-     | Batteries integration, ongoing integration of ML-based branching heuristics (RL, GNN)                           |

---

## Impact

- Savings of **several tens of millions of euros per major release**, and a few tens of millions per year through intermediate improvements
- **4 production versions** deployed over 10+ years
- covers **3 different time horizons** and rationalizes optimization toolchain into a unified platform
- Team of **10–25 engineers and researchers** led across the full lifecycle
- Awarded the **IndusRO'2022 prize** by the French Operational Research Society (ROADEF)

<div class="row">
  <div class="col-md-6">
    <div class="ratio ratio-16x9">
      <iframe width="435" height="245" src="https://www.youtube.com/embed/WJRD5C2Kjug"
              title="IndusRO'2022 — Short video" frameborder="0"
              allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
    </div>
    <p class="caption text-center mt-1">Short video (2 min)</p>
  </div>
  <div class="col-md-6">
    <div class="ratio ratio-16x9">
      <iframe width="435" height="245" src="https://www.youtube.com/embed/tBM6r-B8H8o"
              title="IndusRO'2022 — Full ROADEF presentation" frameborder="0"
              allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
              referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
    </div>
    <p class="caption text-center mt-1">Full ROADEF presentation</p>
  </div>
</div>

---

## Technical stack

**Optimization & algorithms**
: MILP · Branch-and-bound · Cutting planes · Decomposition methods · ADMM · Resource Constrained Shortest Path Problem (RCSPP) · Min Cost Flow · Heuristics · Genetic Algorithm, Interior point method, Bundle method

**Solvers**
: CPLEX · Gurobi · SCIP · FICO Xpress · COIN-OR · HiGHS

**Languages**
: C++ · Python · Conan · TBB

**Engineering & infrastructure**
: GitLab CI/CD · Docker · HPC / Slurm · Singularity

**ML (ongoing integration)**
: PyTorch · PyG · RLLib · Reinforcement learning · Graph Neural Networks

---

## Related research

The ongoing integration of ML into Apogène's solver loop is the subject of active research. Key results:

- **PlanB&B** (AAAI 2026) {% cite Strang2026 %} — model-based RL agent for branching in branch-and-bound, outperforming prior state-of-the-art RL methods on four standard MILP benchmarks.
- **BBMDP** (NeurIPS 2025) {% cite Strang2025 %} — principled MDP formulation for variable selection in B&B, enabling the use of standard RL algorithms for learning optimal branching policies.
- **FMSTS** (CPAIOR 2020) {% cite Etheve2020 %} — first use of RL to fully optimize a branching strategy from scratch.
