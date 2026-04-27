---
layout: about
title: about
permalink: /
subtitle: Research Engineer Expert — <a href="https://www.edf.fr">EDF R&D</a> · OSIRIS · Palaiseau, France

profile:
  align: right
  image: picture.jpg
  image_circular: false
  more_info: >
    <p>EDF Lab OSIRIS</p>
    <p>7 Bd Gaspard Monge</p>
    <p>91120 Palaiseau, France</p>
    <p><a href="mailto:olivier.juan@edf.fr">olivier.juan@edf.fr</a></p>
    <p><a href="mailto:olivier.juan@gmail.com">olivier.juan@gmail.com</a></p>

news: true
selected_papers: true
social: true
announcements:
  enabled: true # includes a list of news items
  scrollable: true # adds a vertical scroll bar if there are more than 3 news items
  limit: 5 # leave blank to include all the news in the `_news` folder
---

<div id="open-to-work-banner" markdown="1">
> **Open to new opportunities** — I am actively looking for a research position at the cutting edge of machine learning and combinatorial optimization (academic lab, industrial research center, or advanced R&D team).
</div>

**Research Engineer Expert** at [EDF R&D](https://www.edf.fr) (OSIRIS — the optimization & operations research division, Palaiseau). For nearly twenty years, I have **shipped** combinatorial optimization software — in **C++** and **Python** — that runs Europe's largest electric utilities. I **led** teams of up to **20 engineers and researchers** and **delivered four production releases** of EDF's flagship short-term unit-commitment solver, **saving tens of millions of euros per release**. I **built** the optimization platform behind the **first French EV aggregator accredited for FCR by RTE** (2022). My recent research on **ML for branch-and-bound** has been published at **NeurIPS 2025** and **AAAI 2026**.

**Languages & tools:** C++ · Python · PyTorch · PyG · RLLib · Ray · CPLEX · GUROBI · SCIP · FICO Xpress · COIN-OR · HiGHS · GitLab CI/CD · Docker · HPC/Slurm · Singularity

_Most production code lives in private repositories (EDF's proprietary infrastructure); public contributions are limited to academic collaborations._

---

#### Flagship project — Apogène

My most significant contribution has been leading the development of **[Apogène](/projects/apogene/)**, EDF's short-term unit commitment software. In 2012, I led the ground-up redesign, modeling the problem as a MILP and developing custom branch-and-bound algorithms, cutting planes, decomposition methods, Resource Constrained Shortest Path algorithms, and Min Cost Flow formulations. I led a team of **10–20 engineers and researchers** through four production releases (V2 in 2018, V3 in 2020, V4 in 2021, V5 in 2025), saving **several tens of millions of euros per major release** and **a few tens of millions per year** through intermediate improvements. In June 2022, the project was awarded the **IndusRO'2022 prize** by the French Operational Research Society (ROADEF).

---

#### Flagship project — DREEV optimization platform

From 2019, I designed and built the optimization platform powering **[DREEV](/projects/dreev/)**, a joint venture between EDF and [Nuvve](https://www.nuvve.com) — the equivalent of Apogène, but for electric vehicles. The platform schedules and dispatches EV fleets as flexible assets participating in electricity markets: **NEBEF** (demand response), **Spot**, **Intraday**, and **FCR** (Frequency Containment Reserve). In 2022, DREEV became the **first French EV aggregator accredited for FCR by RTE**, a direct result of the optimization infrastructure built for this platform.

---

#### Current research

- ML for combinatorial optimization

  Since 2019, I investigate how reinforcement learning and graph neural networks can enhance classical branch-and-bound solvers for MILP. Recent results published at [NeurIPS 2025](https://arxiv.org/abs/2510.19348) (BBMDP) and [AAAI 2026](https://arxiv.org/abs/2511.09219) (PlanB&B) outperform prior state-of-the-art RL branching agents on standard benchmarks.

- Optimization algorithms at Scale

  I investigate replacing the **Lagrangian decomposition** at the core of legacy algorithms with **Frank-Wolfe-type decomposition methods**, aiming at improved convergence guarantees and better scalability. The main target is to handle large EV fleets (tens of thousands) before going to other methods like **Mean fields**.

---

#### Earlier work at EDF (2008–2012)

Task scheduling for maintenance planning, LNG terminal planning & scheduling, and combustion/emission optimization for coal power plants — broad exposure to mathematical programming before shifting focus to unit commitment and the early development of Apogène V1.

---

#### Academic background

Before joining EDF, I completed postdoctoral fellowships at **École Centrale de Paris** (2007–2008, with [Nikos Paragios](http://cvn.ecp.fr/personnel/nikos), combinatorial algorithms for medical imaging) and at the **University of Western Ontario** (2006–2007, with [Yuri Boykov](https://cs.uwaterloo.ca/~yboykov/), graph cuts for computer vision). I hold a **PhD in Computer Science and Mathematics** from [ENPC](https://www.enpc.fr) (2006, distinction) and a Master's degree from the [MVA program](https://www.master-mva.com/) at ENS Paris-Saclay (2002).
