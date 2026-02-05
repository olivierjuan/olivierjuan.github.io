---
layout: page
title: A Markov Decision Process for Variable Selection in Branch & Bound
description: A principled reinforcement learning formulation for learning optimal branching heuristics in MILP solvers.
img: assets/img/bbmdp/teaser.png
importance: 1
category: Machine Learning
related_publications: true
---

# Context: The Branching Challenge

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
    Mixed-Integer Linear Programming (MILP) is central to many critical industrial problems, from nuclear reactor management to complex logistics. Modern solvers rely on the Branch & Bound (B&B) algorithm, the performance of which depends crucially on the variable selection heuristic (branching).<br/>

    Until now, Reinforcement Learning (RL) approaches suffered from a major theoretical limitation: the use of approximate formulations like TreeMDP, which do not respect standard Markov properties.
    </div>
    <div class="col-sm mt-3 mt-md-0"> <div> {% include figure.liquid loading="eager" path="assets/img/bbmdp/results.png" title="BBMDP Structure" class="img-fluid rounded z-depth-1" %} </div>
        <div class="caption"> Normalized scores in log scale of IL, RL and random agents across the Ecole benchmark Prouvost et al. [2020]. </div>
    </div>
</div>

# The Innovation: BBMDP

In our paper {% cite Strang2025 %} presented at NeurIPS 2025, we introduce BBMDP (Branch & Bound MDP), a "vanilla" and rigorous MDP formulation for the variable selection process.

Unlike previous approaches that attempted to minimize the size of independent subtrees, BBMDP models the complete state of the search tree $s_t=(V_t, E_t, x_t)$. This allows us to:

1. Define a theoretically valid value function $Q^π$.

1. Use standard RL algorithms (like DQN with k-step returns) without ad-hoc approximations.

1. Guarantee convergence towards a global optimal policy $π^∗$.

$$π^∗=\displaystyle arg\min_{π} E_{P∼p_0} (∣BB_{(π,ρ)}(P)∣)$$

# Technical Architecture
Our agent, DQN-BBMDP, uses a problem representation in the form of a bipartite graph (Variables/Constraints) processed by a Graph Convolutional Network (GCN). It overcomes the limitations of TreeMDP models by taking into account the real sequential dynamics of the solver.

<div class="row"> <div class="col-sm mt-3 mt-md-0"> {% include figure.liquid loading="eager" path="assets/img/bbmdp/structure.png" title="BBMDP Structure" class="img-fluid rounded z-depth-1" %} </div> </div> <div class="caption"> Trajectory comparison: When applying TD(0), TreeMDP and BBMDP yield equivalent results, see a, b. However, when minimizing k-step temporal difference, the two methods diverge as exemplified in c, d. </div>

# Results and Impact
Experiments conducted on standard benchmarks (Set Covering, Auctions, Maximum Independent Set) demonstrate that our approach establishes a new state-of-the-art for RL agents.

* Performance: DQN-BBMDP reduces the number of nodes by 27% and solving time by 38% compared to the previous state-of-the-art (DQN-Retro).

* Generalization: The agent shows superior capability to generalize on larger dimension instances (Transfer Learning) compared to methods based on Imitation Learning.

| ---:| ---:| ---:| ---:| ---:| ---:| ---:| ---:| ---:| ---:| ---:|
| Benchmarks | **Set Covering** |  | **Comb. Auction** |  | **Max. Ind. Set** |  | **Mult. Knapsack** |  | **Norm. Score** |  |
| Method | Node | Time | Node | Time | Node | Time | Node | Time | Node | Time |
| ===:| ===:| ===:| ===:| ===:| ===:| ===:| ===:| ===:| ===:| ===:|
| Presolve | - | 12.3 | - | 2.67 | - | 5.16 | - | 0.46 | - | - |
| Random | 271632 | 842 | 317235 | 749 | 215879 | 2102 | 93452 | 70.6 | 5555 | 2737 |
| SB | 672.1 | 398 | 389.6 | 255 | 169.9 | 2172 | <span style="color: red;"><b>1709</b></span> | <span style="color: red;"><b>12.5</b></span> | 9 | 1425 |
| SCIP | 3309 | 48.4 | 1376 | 14.77 | 3368 | 90.0 | 30620 | 22.1 | 62 | 90 |
| IL | <span style="color: red;"><b>2610</b></span> | 23.1 | <span style="color: red;"><b>1309</b></span> | <span style="color: red;"><b>9.8</b></span> | <span style="color: red;"><b>1882.0</b></span> | <span style="color: red;"><b>37.6</b></span> | 9747 | 46.5 | <span style="color: red;"><b>39</b></span> | <span style="color: red;"><b>55</b></span> |
| IL-DFS | 3103 | <span style="color: red;"><b>22.5</b></span> | 1802 | 11.1 | 3501 | 55.5 | 43224 | 177 | 75 | 93 |
| ===:| ===:| ===:| ===:| ===:| ===:| ===:| ===:| ===:| ===:| ===:|
| PG-tMDP | 44649 | 221 | 6001 | 30.7 | <span style="color: blue;"><b>3133</b></span> | <span style="color: blue;"><b>39.5</b></span> | 35614 | 123 | 298 | 223 |
| DQN-tMDP | 8632 | 71.3 | 20553 | 116 | 45634 | 477 | <span style="color: blue;"><b>22631</b></span> | <span style="color: blue;"><b>65.1</b></span> | 439 | 445 |
| DQN-Retro | 6100 | 59.4 | 2908 | 18.4 | 119478 | 1863 | 27077 | 79.5 | 494 | 662 |
| DQN-BBMDP | <span style="color: blue;"><b>5651</b></span> | <span style="color: blue;"><b>46.4</b></span> | <span style="color: blue;"><b>2273</b></span> | <span style="color: blue;"><b>11.8</b></span> | 7168 | 81.3 | 37098 | 109 | <span style="color: blue;"><b>100</b></span> | <span style="color: blue;"><b>100</b></span> |
| ---:| ---:| ---:| ---:| ---:| ---:| ---:| ---:| ---:| ---:| ---:|

Note: Results show a significant reduction in the gap with the expert (Strong Branching), while being much faster to execute.

# Resources
The source code for the implementation and pre-trained models are available to the scientific community to foster reproducibility.

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center"> {% include repository/repo.liquid repository="abfariah/bbmdp" %} </div>

# Citation
If you use this work, please cite our paper:

    @inproceedings{strang2025bbmdp, title={A Markov Decision Process for Variable Selection in Branch & Bound}, author={Strang, Paul and Alès, Zacharie and Juan, Olivier and Bissuel, Côme and Kedad-Sidhoum, Safia and Rachelson, Emmanuel}, booktitle={Advances in Neural Information Processing Systems (NeurIPS)}, year={2025} }






































**********************************
Every project has a beautiful feature showcase page.
It's easy to include images in a flexible 3-column grid format.
Make your photos 1/3, 2/3, or full width.

To give your project a background in the portfolio page, just add the img tag to the front matter like so:

    ---
    layout: page
    title: project
    description: a project with a background image
    img: /assets/img/12.jpg
    ---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/1.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/3.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Caption photos easily. On the left, a road goes through a tunnel. Middle, leaves artistically fall in a hipster photoshoot. Right, in another hipster photoshoot, a lumberjack grasps a handful of pine needles.
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    This image can also have a caption. It's like magic.
</div>

You can also put regular text between your rows of images, even citations {% cite einstein1950meaning %}.
Say you wanted to write a bit about your project before you posted the rest of the images.
You describe how you toiled, sweated, _bled_ for your project, and then... you reveal its glory in the next row of images.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    You can also have artistically styled 2/3 + 1/3 images, like these.
</div>

The code is simple.
Just wrap your images with `<div class="col-sm">` and place them inside `<div class="row">` (read more about the <a href="https://getbootstrap.com/docs/4.4/layout/grid/">Bootstrap Grid</a> system).
To make images responsive, add `img-fluid` class to each; for rounded corners and shadows use `rounded` and `z-depth-1` classes.
Here's the code for the last row of images above:

{% raw %}

```html
<div class="row justify-content-sm-center">
  <div class="col-sm-8 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
```

{% endraw %}
