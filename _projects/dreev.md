---
layout: page
title: "DREEV: V2G Fleet Dispatch for Frequency Containment Reserve"
description: >
  MILP formulation and algorithm development for optimal EV fleet dispatch providing
  Frequency Containment Reserve (FCR) services to the French grid. Part of EDF R&D's
  Smart Charging P11L1 program, in collaboration with DREEV (EDF–Nuvve joint venture).
img: assets/img/dreev/teaser.png
importance: 2
category: Optimization
years: "2019 – 2026"
role: "Research Engineer"
team_size: "Multi-org (EDF R&D, DREEV)"
impact: "V2G RTE certification; 50 MW fleet target"
stack: "MILP · Python · Rolling Horizon MPC"
---

**Partner Organizations:** EDF R&D — OSIRIS Department · [DREEV](https://www.dreev.com) · [Nuvve](https://www.nuvve.com) · EDF Agregio · RTE

**Focus Areas:** Mixed Integer Linear Programming · Vehicle-to-Grid (V2G) · Ancillary Services · Rolling Horizon Optimization · Scalable Algorithms

---

## Overview

As part of EDF Lab's OSIRIS department and the broader **Electric Mobility** R&D program, I contributed to the mathematical and algorithmic specifications for DREEV's operational charging platform.

[DREEV](https://www.dreev.com) is a joint venture between EDF and Nuvve that manages bidirectional EV charging on business sites and monetizes the collective battery flexibility through **ancillary services** to the French grid.

The core deliverable was a formal **Mixed Integer Linear Program (MILP)** for dispatching charging and discharging power across a fleet of V2G-capable EVs to meet **Frequency Containment Reserve (FCR)** capacity commitments — the primary fast-response balancing service of the European interconnected grid.

This work directly contributed to DREEV's path toward **RTE certification as a V2G reserve entity**, with a long-term objective of operating a 50 MW virtual V2G power plant across 5,000 chargers.

---

## Problem Context

FCR requires an aggregator to maintain a contracted reserve capacity (upward and downward) continuously available for grid frequency regulation. DREEV aggregates EV charging transactions across multiple business sites and offers the collective flexibility for participation in this market.

The dispatch problem is technically demanding on several fronts:

| Challenge                     | Description                                                                                                                                                                                                                            |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Bidirectionality**          | V2G chargers can both charge and discharge EVs. AC/DC efficiency in both directions introduces **binary mode-selection variables**, making the problem mixed-integer.                                                                  |
| **SOC feasibility under FCR** | Committing FCR capacity means guaranteeing that full activation for up to 15 minutes will not violate any EV's state-of-charge bounds. This requires dedicated forward-looking constraints on top of the baseline charging trajectory. |
| **Temporal coupling**         | SOC evolves dynamically over the horizon; decisions at each 15-minute step must anticipate future needs while maintaining real-time FCR commitments.                                                                                   |
| **Dispatch stability**        | The algorithm re-executes every 15 minutes in a receding-horizon fashion; successive solutions must be stabilized to avoid rapid oscillations in per-charger setpoints.                                                                |
| **Scalability**               | The algorithm must remain tractable as the fleet grows from tens of EVs toward thousands.                                                                                                                                              |

---

## Technical Contributions

### 1. MILP Formulation for FCR Dispatch

I co-developed a complete mathematical formulation of the FCR dispatch problem as a **Mixed Integer Linear Program**, running on a rolling horizon in a **Model Predictive Control** fashion.

The model captures three families of constraints:

- **Power domain constraints** — AC/DC converter efficiency in both directions, auxiliary EVSE consumption, and min/max power bounds, enforced via binary mode variables with mutual exclusion.
- **SOC dynamics** — tracking each EV's state of charge over the full planning horizon, bounded by time-varying user-defined SOC limits.
- **FCR feasibility constraints** — asserting, for each EV at each timestep, that committed capacity can always be physically delivered over the FCR look-ahead duration.

### 2. Objective Function Design

The objective is a weighted sum of six terms reflecting operational priorities:

| Term                       | Role                                                               |
| -------------------------- | ------------------------------------------------------------------ |
| **Customer mobility need** | Fulfil the user's requested energy need at departure time.         |
| **Customer bill**          | Minimise the customer's electricity bill.                          |
| **Energy sourcing cost**   | Minimise the cost of electricity supply for the producer/supplier. |
| **FCR penalty**            | Enforce the fleet to satisfy the contracted FCR capacity.          |

### 3. Scalability Research — Frank-Wolfe Decomposition

A key open challenge is preparing the dispatch algorithm for deployment at scale: from the early pilot of ~100 EVs toward fleets of 1,000, 10,000, and ultimately 100,000 vehicles.

The legacy algorithm relies on **Lagrangian decomposition** to coordinate individual vehicle subproblems, but this approach shows convergence limitations at large scale. Ongoing research investigates replacing it with **Frank-Wolfe-type decomposition methods**, which offer projection-free iterations, improved convergence guarantees, and a better structural fit with the EV coordination problem.

### 4. Operational Specification

The work included specifying complete input/output data interfaces (per-EV SOC parameters, per-CP power limits and efficiencies, system-level FCR demand and penalty functions), edge-case handling (instant-charge requests, unreachable chargers, EVs outside initial SOC bounds), and integration with DREEV's Power Dispatcher platform.

---

## Impact

This work directly supported DREEV's operational development and its **RTE certification process as a V2G reserve entity** — one of the first of its kind in France. Real operational monitoring data confirmed the system's ability to deliver symmetric upward/downward reserve capacity as contracted.

The long-term vision is a **50 MW virtual V2G power plant** operating across 5,000 chargers in France, UK, Italy, Belgium, and Germany, with an estimated impact of **250,000 tonnes of CO2 avoided** through carbon-aware dispatch.

Industrial partners include ABB, Nissan, Stellantis, Iveco, Hager, and RTE.

Results were communicated through several channels:

- **[En français](https://www.rte-france.com/actualites/vehicules-electriques-equilibrage-systeme-electrique)** — Communiqué de presse officiel RTE sur la recharge EV et l'équilibrage du réseau
- **[In English](https://www-rte--france-com.translate.goog/actualites/vehicules-electriques-equilibrage-systeme-electrique?_x_tr_sl=fr&_x_tr_tl=en&_x_tr_hl=fr&_x_tr_pto=wapp)** — English translation via Google Translate
- **[DREEV on Le Monde de Jamy](https://www.linkedin.com/posts/dreev_le-monde-de-jamy-v2g-activity-7016826363202396160-9w4v/?originalSubdomain=fr)** — V2G featured on the French science show, shared by DREEV
- **[IRT SystemX talk]({{ '/assets/pdf/IRT_systemX_EV_flexibility_for_electric_system.pdf' | relative_url }})** — Presentation on EV flexibility for the electric system

---

## Skills Demonstrated

**Optimization & Modeling**

- Mixed Integer Linear Programming formulation and LP reformulation techniques (piecewise-linear functions, absolute values, big-M)
- Rolling horizon / Model Predictive Control for real-time energy dispatch
- Algorithm scalability research for large-scale fleet optimization

**Domain Knowledge**

- Vehicle-to-grid physics: AC/DC converter modeling, SOC dynamics, efficiency constraints
- Ancillary services markets: FCR and aFFR mechanics, aggregator architecture, EDF Agregio interface

**Engineering & Collaboration**

- Algorithm specification for industrial deployment in an operational charging platform
- Cross-functional collaboration across EDF R&D departments (OSIRIS, TREE, EFESE, MIRE) and industrial partners
