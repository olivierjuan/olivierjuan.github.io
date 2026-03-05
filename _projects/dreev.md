---
layout: page
title: "DREEV: V2G Fleet Dispatch for Frequency Containment Reserve"
description: >
  MILP formulation and algorithm development for optimal EV fleet dispatch providing
  Frequency Containment Reserve (FCR) services to the French grid. Part of EDF R&D's
  Smart Charging P11L1 program, in collaboration with DREEV (EDF–Nuvve joint venture).
img:
importance: 1
category: Optimization
---

**Partner Organizations:** EDF R&D — OSIRIS Department · [DREEV](https://www.dreev.com) · [Nuvve](https://www.nuvve.com) · EDF Agregio · RTE

**Focus Areas:** Mixed Integer Linear Programming · Vehicle-to-Grid (V2G) · Ancillary Services · Rolling Horizon Optimization · Scalable Algorithms

---

## Overview

As part of EDF Lab's OSIRIS department and the broader **Smart Charging P11L1** R&D program, I contributed to the mathematical and algorithmic specifications for DREEV's operational charging platform. [DREEV](https://www.dreev.com) is a joint venture between EDF and Nuvve that manages bidirectional EV charging on business sites and monetizes the collective battery flexibility through **ancillary services** to the French grid.

The core deliverable was a formal **Mixed Integer Linear Program (MILP)** for dispatching charging and discharging power across a fleet of V2G-capable EVs to meet **Frequency Containment Reserve (FCR)** capacity commitments — the primary fast-response balancing service of the European interconnected grid — while respecting per-vehicle and per-site physical constraints and user charging preferences. This work directly contributed to DREEV's path toward **RTE certification as a V2G reserve entity**, with a long-term objective of operating a 50 MW virtual V2G power plant (5,000 chargers).

---

## Problem Context

FCR requires an aggregator to maintain a contracted reserve capacity (upward and downward) continuously available for grid frequency regulation. DREEV aggregates EV charging transactions across multiple business sites and offers the collective flexibility to EDF Agregio for participation in this market. The dispatch problem is technically demanding on several fronts:

- **Bidirectionality and converter physics:** V2G chargers can both charge and discharge EVs, but AC/DC conversion efficiency requires enforcing minimum power thresholds in each direction. This introduces **binary mode-selection variables**, making the problem mixed-integer rather than purely continuous.
- **SOC feasibility under FCR activation:** Committing FCR capacity means guaranteeing that *full* activation for up to 15 minutes will not violate any EV's state-of-charge bounds. This requires dedicated forward-looking constraints — derived from the user's minimum energy requirement curve (Lances curve) — on top of the baseline charging trajectory.
- **Temporal coupling:** SOC evolves dynamically over a 96-timestep horizon. Decisions at each 15-minute step must anticipate future charging needs while maintaining real-time FCR commitments.
- **Dispatch stability:** Because the algorithm re-executes every 15 minutes in a receding-horizon fashion, successive solutions must be stabilized to avoid rapid oscillations in per-charger setpoints sent to the EVSE hardware.
- **Scalability:** The algorithm must remain tractable as the fleet grows from tens of EVs toward thousands — a key open research question for the project roadmap.

---

## Technical Contributions

### 1. MILP Formulation for FCR Dispatch

I co-developed a complete mathematical formulation of the FCR dispatch problem as a **Mixed Integer Linear Program**. The algorithm runs on a rolling horizon of one day (96 × 15-minute timesteps) and is re-executed every 15 minutes — only the first-timestep decision is applied before the next solve (receding horizon / MPC-style).

The model captures:

- **Power domain constraints** at each charging point: AC/DC converter efficiency in both directions, auxiliary EVSE consumption, and minimum/maximum charging and discharging power bounds, enforced via binary charging/discharging mode variables with mutual exclusion.
- **SOC dynamics:** Linear recursion tracking each EV's state of charge over the full planning horizon, bounded by time-varying user-defined SOC limits.
- **FCR feasibility constraints:** For each EV at each timestep, separate power trajectories are computed for "full upward FCR activation" and "full downward FCR activation" scenarios. These are constrained to remain within SOC bounds over the FCR look-ahead duration, guaranteeing that committed capacity can always be physically delivered.
- **Fleet aggregation:** Per-vehicle FCR capacities are summed to compute fleet-level upward and downward reserve availability, compared against the contracted demand.

### 2. Objective Function Design

The objective is a weighted sum of four terms reflecting operational priorities:

1. **FCR penalty** — A convex piecewise-linear penalty on the gap between available and contracted FCR capacity (separately for up and down), with high penalization for under-capacity and decreasing marginal reward for over-capacity margin. Incorporated into the LP via standard epigraph reformulation.
2. **Fleet baseline stability** — Penalizes changes in the aggregate charging setpoint from the previously applied dispatch, to avoid step changes in the fleet's net grid consumption.
3. **Per-CP baseline stability** — Penalizes per-charger setpoint changes from the previous dispatch, weighted to exclude chargers that were inactive in the prior run.
4. **FCR allocation stability** — Penalizes changes in individual upward/downward FCR capacity allocations to reduce dispatch chatter. Absolute values are linearized via standard LP reformulation.

### 3. Post-Processing: Margin Dispatch

When the LP solution yields aggregate FCR capacity exceeding the contracted demand, a proportional scaling post-processor redistributes the margin back to the exact contracted level across all participating charging points, ensuring contractual compliance while preserving individual feasibility.

### 4. Scalability Research

A key open challenge in the project was preparing the dispatch algorithm for deployment at scale: from the current pilot of ~100 EVs toward fleets of 1,000, 10,000, and ultimately 100,000 vehicles. This involved investigating decomposition approaches and heuristics to maintain tractable solve times as the fleet size grows.

### 5. Broader Algorithm Roadmap

The work also addressed the algorithmic roadmap for DREEV beyond FCR, including:

- Integration of **aFFR (automatic Frequency Restoration Reserve)** and **SPOT energy market** optimization into the dispatch
- **Carbon intensity-aware charging** (dispatching relative to the grid's real-time CO2 intensity)
- **Local site constraints**: branch and site power limits, coordination with Izivia's local smart-charging proxy (OCPP)
- **EV/charger admission control**: optimizing acceptance/rejection of EVs at under-dimensioned sites
- **Charging demand forecasting**: predicting EV arrival, departure, and energy needs from historical transaction data to feed the optimizer

### 6. Operational Specification

The work included specifying complete input/output data interfaces (per-EV SOC parameters, per-CP power limits and efficiencies, system-level FCR demand and penalty functions), edge-case handling (instant-charge requests, unreachable chargers, EVs outside initial SOC bounds), and integration with DREEV's Power Dispatcher platform.

---

## Impact

This work directly supported DREEV's operational development and its **RTE certification process as a V2G reserve entity** — one of the first of its kind in France. Real operational monitoring data (frequency tracking, aggregated power, FCR capacity curves) confirmed the system's ability to deliver symmetric upward/downward reserve capacity as contracted. The long-term vision is a **50 MW virtual V2G power plant** operating across 5,000 chargers across France, UK, Italy, Belgium, and Germany, with an estimated impact of 250,000 tonnes of CO2 avoided through carbon-aware dispatch. Industrial partners include ABB, Nissan, Stellantis, Iveco, Hager, and RTE.

Results on EV fleet participation in grid balancing services were also communicated through official RTE France channels:

- **[In French](https://www.rte-france.com/actualites/vehicules-electriques-equilibrage-systeme-electrique)** — Official RTE press release on EV charging and grid balancing
- **[In English](https://www-rte--france-com.translate.goog/actualites/vehicules-electriques-equilibrage-systeme-electrique?_x_tr_sl=fr&_x_tr_tl=en&_x_tr_hl=fr&_x_tr_pto=wapp)** — English translation via Google Translate

---

## Skills Demonstrated

- Mixed Integer Linear Programming formulation and LP reformulation techniques (piecewise-linear functions, absolute values, big-M)
- Rolling horizon / Model Predictive Control for real-time energy dispatch
- Vehicle-to-grid physics: AC/DC converter modeling, SOC dynamics, efficiency constraints
- Ancillary services markets: FCR and aFFR mechanics, aggregator architecture, EDF Agregio interface
- Algorithm scalability research for large-scale fleet optimization
- Algorithm specification for industrial deployment in an operational charging platform
- Cross-functional collaboration across EDF R&D departments (OSIRIS, TREE, EFESE, MIRE) and industrial partners

---
