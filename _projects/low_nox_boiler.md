---
layout: page
title: "Low NO<sub>x</sub> Configurations in an Industrial Boiler via Genetic Algorithm & CFD"
description: >
  Coupling a genetic algorithm with CFD simulations (Code_Saturne) to automatically discover
  optimal operating configurations for a 600 MW tangentially-fired pulverized-coal boiler,
  minimizing NOx emissions while controlling corrosion risk.
img: assets/img/fuel/teaser.png   # replace with an actual image path
importance: 2
category: Optimization               # adapt to your own categories
related_publications: DalSecco2015, DalSecco2010          # must match your BibTeX key in _bibliography/papers.bib
---

## Overview

Industrial coal-fired power plants must balance two conflicting objectives: reducing
nitrogen-oxide (NO<sub>x</sub>) emissions to comply with increasingly strict environmental
legislation, and preserving the mechanical integrity of the boiler's water-wall tubes, which
are exposed to aggressive corrosion under reducing atmospheres created by low-NO<sub>x</sub>
operating strategies.

This project demonstrates how **Genetic Algorithms (GA)** combined with **Computational Fluid
Dynamics (CFD)** can automatically explore the enormous configuration space of a 600 MW
corner-fired boiler and identify settings that achieve the best trade-off between pollutant
reduction and operational safety.

---

## Boiler Description

EDF operates three 600 MW tangentially-fired pulverized-coal units in France (Cordemais 4 & 5,
Le Havre 4). Key characteristics:

- **Total height:** ~80 m; **cross-section:** 16.6 × 16.6 m
- **Four firing corners** (A1–A4), three burner elevation groups (Group 1, 2, 3)
- **Six mills** (A–F) feeding coal to the burners; typically four mills active at full load
- **Adjustable parameters:** air-damper positions (on/off), coal mass-flow per mill, vertical
  tilt of burner nozzles (−30° to +30°)

<div class="row justify-content-sm-center">
  <div class="col-sm-10 mt-3 mt-md-0">
    {% include figure.liquid
       path="assets/img/fuel/boiler_schematic.png"
       title="Schematic of the 600 MW boiler"
       class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Schematic of the 600 MW corner-fired boiler with firing groups and air/coal nozzle arrangement.
</div>

---

## CFD Model (Code_Saturne)

All boiler configurations are evaluated with **Code_Saturne**, EDF's in-house finite-volume
solver, on a structured mesh of **470,000 cells**.

| Sub-model | Approach |
|-----------|----------|
| Turbulence | Standard *k*–ε eddy-viscosity |
| Devolatilization | Two-competing-reactions Kobayashi scheme |
| Char burnout | Shrinking-core with external O₂ diffusion |
| Gas-phase combustion | PDF mixture-fraction model (light & heavy volatiles + CO) |
| Radiation | P1 model with local absorption coefficients |
| NO<sub>x</sub> formation | De Soete (fuel NO) + Zeldovich (thermal NO) |

---

## Genetic Algorithm

The GA is implemented with the **ParadisEO** library (INRIA). Each *individual* (chromosome)
encodes a complete boiler operating point:

- **Binary genes** — on/off state of each air/secondary-air nozzle
- **Integer genes** — vertical tilt angles for corners A1/A3 and A2/A4
- **Continuous genes** — coal mass-flow rate per mill (0 – 100 % capacity)

### Cost Function

Rather than optimising a single metric, a composite **economic cost** (€/year) penalises:

1. **O₂ deviation** — incomplete combustion penalty if O₂ departs >0.5 vol.% from target
2. **CO in flue gas** — heavy penalty above 100 ppm@6%O₂
3. **NO<sub>x</sub> emissions** — SCR ammonia & catalyst savings vs. penalty above 200 mg/Nm³@6%O₂
4. **Water-wall corrosion** — metal-loss model (EPRI/PowerGen UK equations) as a function of
   local CO concentration and metal temperature; coating cost when wastage > 50 µm/year
5. **Heat-flux heterogeneity** — forced-outage penalty when flux variance exceeds a threshold
6. **Unburned carbon** — ash-recycling penalty when carbon-in-ash > 7 %

### Numerical Setup

| Parameter | Value |
|-----------|-------|
| Initial population | 52 individuals (best-practice configurations) |
| Population size (constant) | ~50 |
| Generations | 50 |
| CFD time per individual | ~14 h on 8-core node |
| Cluster | 52 × 8-core nodes → **~15 min effective time per individual** |
| Total CFD runs | thousands |

---

## Key Results

### Low-NO<sub>x</sub> Configurations

Most high-performing configurations share a common pattern:

- **Four mills (A, B, C, D)** feeding the **lower and middle groups only** — concentrates
  fuel-rich zones in the lower furnace, promoting fuel-N reduction before additional OFA air
  is introduced.
- **Horizontal burner tilt** (0°) — upward tilt consistently increases NO<sub>x</sub>; when
  the sum of both tilt angles exceeds +45°, NO<sub>x</sub> hardly drops below 900 mg/Nm³@6%O₂.
- **Selective opening of FOE, FOF, FOO-up nozzles** — staged air injection reduces local O₂
  near active burners; the GA found non-intuitive asymmetric patterns (e.g. opening nozzles
  only at two diagonally opposite corners).

<div class="row">
  <div class="col-sm-6 mt-3 mt-md-0">
    {% include figure.liquid
       path="assets/img/fuel/low_corrosion.png"
       title="Low corrosion risk – case 610"
       class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm-6 mt-3 mt-md-0">
    {% include figure.liquid
       path="assets/img/fuel/high_corrosion.png"
       title="High corrosion risk – case 962"
       class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  CO concentration along the boiler walls for two GA individuals: case 610 (moderate staging,
  low corrosion) vs. case 962 (deep staging, high corrosion risk despite maximum NO<sub>x</sub>
  abatement of 54 %).
</div>


<div class="row">
  <div class="col-sm-6 mt-3 mt-md-0">
    {% include figure.liquid
       path="assets/img/fuel/low_nox.png"
       title="Low NO<sub>x</sub> – case 624"
       class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm-3 mt-3 mt-md-0">
    {% include figure.liquid
       path="assets/img/fuel/high_nox.png"
       title="High NO<sub>x</sub> – reference case"
       class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  NO<sub>x</sub> concentration in the boiler for two GA individuals: case 624 (low NO<sub>x</sub>) vs. reference case (high NO<sub>x</sub> emission).
</div>


### NO<sub>x</sub> vs. Corrosion Trade-off

- Configurations below **500 mg/Nm³@6%O₂** almost always require industrial coating to protect
  water-wall tubes.
- In the **500–600 mg/Nm³** range, many individuals achieve a low corrosion cost, making
  them the best compromise for long-term operation.

### Burner Tilt & Heat Flux

- Downward or horizontal tilting concentrates heat on the **ash hopper**, increasing average
  flux and its spatial variance.
- Upward tilting reduces both quantities — consistent with on-site observations where operators
  tilt burners up to relieve temperature peaks on water-wall tubes.

### Unburned Carbon

- A classic **NO<sub>x</sub>–burnout trade-off** is observed: deeper air staging raises
  unburned carbon.
- With the coal blend used (high volatile, moderate N content), carbon-in-ash remains below 2 %
  even in deep staging, always permitting fly-ash recycling.

---

## Validation Against On-Site Measurements

Results were validated during two test campaigns:

| Campaign | Boiler | Date | Key finding |
|----------|--------|------|-------------|
| Cordemais 4 | ABCD mills | Jan 2012 | 60 % NO<sub>x</sub> abatement (burners ~horizontal); GA-predicted trend confirmed |
| Le Havre 4 | ABCDF mills | Apr 2011 | 36 % NO<sub>x</sub> abatement (GA predicted 35 %); 47 % ammonia saving |

Both campaigns confirmed low corrosion risk for moderate staging, and high CO near walls for
deep staging — fully consistent with GA/CFD predictions.

---

## Conclusions & Perspectives

- **Genetic algorithms + CFD** can automatically discover non-intuitive boiler settings that a
  human operator would not easily find through trial and error.
- The **composite cost function** successfully steers the search away from configurations that
  are good for NO<sub>x</sub> but catastrophic for corrosion.
- Future work: (1) couple a thermo-hydraulic tube model to predict metal temperature peaks more
  accurately; (2) extend to partial loads and variable coal blends; (3) replace the scalar cost
  function with a proper **multi-objective Pareto** formulation.

---

## Citations

'''bibtex
@Article{DalSecco2015,
  abbr         = {Article},
  author    = {Dal Secco, Sandro and Juan, Olivier and Louis-Louisy, Myriam and Lucas, Jean-Yves and Plion, Pierre and Porcheron, Lynda},
  journal   = {Fuel},
  title     = {Using a genetic algorithm and CFD to identify low NOx configurations in an industrial boiler},
  year      = {2015},
  pages     = {672--683},
  volume    = {158},
  abstract  = {This paper focuses on a computational intelligence approach used for minimizing NOx emissions in a 600 MW tangentially-fired pulverized-coal boiler. Genetic Algorithms (GA) were used to correlate operating parameters to significant output data predicted by CFD simulations of the boiler. The operating parameters include the opening or closing of air dampers, changing the coal distribution through mill selection and feed rate and vertical tilting of the burners. A target function was introduced to estimate for each boiler settings defined by given operating parameters, the costs associated with corrosion on the water-wall tubes, heterogeneous heat flux distribution along the walls, unburned carbon in fly ash and NOx emissions. The GA was able to automatically generate innovative boiler configurations among thousands of CFD calculations performed. The target function allowed the search space to be explored to establish configurations offering a good compromise between NOx reduction and the cost associated with corrosion in particular. Moreover, the predicted NOx emissions from the GA model are consistent with the NOx levels measured during test campaigns.},
  doi       = {10.1016/j.fuel.2015.06.021},
  publisher = {Elsevier},
}
@InProceedings{DalSecco2010,
  abbr         = {Proceedings},
  author    = {Dal Secco, Sandro and Juan, Olivier and Lucas, Jean-Yves and Plion, Pierre},
  booktitle = {International conference on metaheuristics and nature inspired computing},
  title     = {Evolutionary algorithm for pollutant emission minimization in coal power plants},
  year      = {2010},
  abstract  = {In order to limit the impact of thermal electrical power plants on environment (nitrogen and
sulphur oxides, flying ashes; effect of green house gas is, so far, out of the scope), producers
can use secondary exhaust gas treatment (catalytic removal, washing) or primary measures (fuel
composition, air staging). EDF as the largest electricity producer in Europe, manages some thermal
plants fed with pulverised coal, heavy fuel oil or natural gas. Nitrogen oxides (NOx) reduction can
be obtained using recently designed burners (inducing expansive investment) or using suitable fuel
and air staging mimicking their behaviour. This study is devoted to pulverised coal boiler settings.},
}
'''