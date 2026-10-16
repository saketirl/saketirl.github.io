---
title: 'Preserving Plasticity in Continual Robot Learning'
date: 2026-10-16
permalink: /posts/2026/10/skewon-plasticity/
excerpt: 'Swapping AdamW for a Stiefel-constrained optimizer keeps pixel-based robot learners plastic for 100M frames, and a Gram-preserving variant improves continual fine-tuning of a VLA on a real SO-101 arm from 37% to 62% success.'
header:
  teaser: skewon/real_sort.jpg
tags:
  - reinforcement learning
  - continual learning
  - plasticity
  - robotics
  - optimization
hide_date: true
comments: false
related: false
---

<style>
  .sk-fig { margin: 1.75em 0; text-align: center; }
  .sk-fig img { max-width: 100%; height: auto; }
  .sk-fig figcaption { font-size: 0.82em; color: #555; text-align: left; margin-top: 0.6em; line-height: 1.5; }
  .sk-grid { display: grid; gap: 0.6em; align-items: start; }
  .sk-grid-2 { grid-template-columns: repeat(2, minmax(0, 1fr)); }
  .sk-grid-4 { grid-template-columns: repeat(4, minmax(0, 1fr)); }
  .sk-grid-5 { grid-template-columns: repeat(5, minmax(0, 1fr)); }
  .sk-grid figure { margin: 0; }
  .sk-grid figure span { display: block; font-size: 0.75em; color: #555; margin-top: 0.25em; }
  .sk-grid img { display: block; width: 100%; margin: 0; border-radius: 4px; }
  @media (max-width: 600px) {
    .sk-grid-2 { grid-template-columns: 1fr; }
    .sk-grid-4, .sk-grid-5 { grid-template-columns: repeat(2, minmax(0, 1fr)); }
  }
  .sk-legend { display: flex; justify-content: center; flex-wrap: wrap; gap: 1.6em; font-size: 0.85em; margin: 0.5em 0 0.2em; }
  .sk-legend i { display: inline-block; width: 2.2em; height: 0.8em; margin-right: 0.45em; vertical-align: -0.05em; border-radius: 2px; }
  .sk-stats { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 0.8em; margin: 1.5em 0; }
  @media (max-width: 600px) { .sk-stats { grid-template-columns: 1fr; } }
  .sk-stat { border: 1px solid #e3e3e3; border-radius: 6px; padding: 0.8em 1em; }
  .sk-stat b { display: block; font-size: 1.5em; line-height: 1.2; color: #2E3338; }
  .sk-stat small { color: #666; font-size: 0.78em; line-height: 1.4; display: block; margin-top: 0.3em; }
  .sk-table-wrap { overflow-x: auto; margin: 1em 0; }
  .sk-table { font-size: 0.82em; border-collapse: collapse; width: 100%; }
  .sk-table th, .sk-table td { padding: 0.35em 0.6em; text-align: center; border-bottom: 1px solid #e6e6e6; }
  .sk-table th:first-child, .sk-table td:first-child { text-align: left; }
  .sk-table thead th { border-bottom: 2px solid #bbb; }
  .sk-table .sk-total td { border-top: 2px solid #bbb; font-weight: 600; }
  .sk-algo { font-size: 0.85em; background: #f7f7f8; border-left: 3px solid #2E3338; padding: 0.8em 1.1em; margin: 1.2em 0; overflow-x: auto; }
  .sk-algo ol { margin: 0.3em 0 0 0; }
  .sk-meta { margin: 0 0 1.6em; padding-bottom: 1.2em; border-bottom: 1px solid #e6e6e6; }
  .sk-badge { display: inline-block; font-size: 0.72em; font-weight: 600; letter-spacing: 0.06em; text-transform: uppercase; color: #8a5a00; background: #fdf3dc; border: 1px solid #ecd49a; border-radius: 999px; padding: 0.2em 0.8em; }
  .page__content .sk-meta p { margin: 0; }
  .page__content .sk-meta .sk-authors { font-size: 1.05em; font-weight: 600; margin-top: 0.7em; }
  .page__content .sk-meta .sk-affil { font-size: 0.9em; color: #666; }
  .sk-hero { display: grid; grid-template-columns: 280fr 374fr 386fr; column-gap: 1.6em; row-gap: 1.2em; align-items: start; }
  .sk-hero figure { display: block; margin: 0; }
  .sk-hero img { display: block; width: 100%; height: auto; margin: 0; border-radius: 4px; }
  .sk-hero figure span { display: block; font-size: 0.78em; color: #555; margin-top: 0.45em; text-align: center; }
  @media (max-width: 600px) {
    .sk-hero { grid-template-columns: 1fr; }
  }
  .MJXc-display { overflow-x: auto; overflow-y: hidden; max-width: 100%; }
</style>

<div class="sk-meta">
  <span class="sk-badge">Under review</span>
  <p class="sk-authors">Saket Tiwari, Yao Qin</p>
  <p class="sk-affil">University of California, Santa Barbara (UCSB)</p>
</div>

<figure class="sk-fig">
  <div class="sk-hero">
    <figure><img src="/images/skewon/mujoco_chain.gif" alt="Simulated MuJoCo Ant and Humanoid agents learning to walk as their dynamics change"><span>(a) MuJoCo Locomotion</span></figure>
    <figure><img src="/images/skewon/so101_chain.gif" alt="Real SO-101 arm performing Pick, Place, Stack, Cup and Sort tasks in sequence"><span>(b) Real World SO-101 Arm</span></figure>
    <figure><img src="/images/skewon/panda_chain.gif" alt="Simulated Franka Panda arm performing Grasp, Place, Stack and Pyramid tasks in sequence"><span>(c) Franka Panda Simulation</span></figure>
  </div>
  <figcaption>We show continual learning across three environments: (a) pixel-based Ant and Humanoid locomotion in MuJoCo, (b) a physical SO-101 arm fine-tuned through a chain of real-world manipulation tasks, and (c) a simulated Franka Panda manipulation chain in ManiSkill. In each, a single agent learns the tasks one after another.</figcaption>
</figure>

**TL;DR.** A robot that keeps learning new tasks from camera images slowly loses its ability to learn. Standard fixes (layer norm, L2, reward normalization, Parseval regularization) aren't enough in this setting. Keeping the network's weight matrices on the **Stiefel manifold** ($$W^\top W = I$$) *inside the optimizer*, using **Skewon**, keeps agents plastic for up to 100M frames. For pretrained models, where weights aren't orthonormal, our new **Gram-Skewon** optimizer preserves each matrix's Gram matrix $$W^\top W$$ instead, and continually fine-tunes a vision-language-action model on a physical robot arm.

<div class="sk-stats">
  <div class="sk-stat"><b>62% vs 37%</b><small>Final-task success on the real SO-101 arm (Sort), Gram-Skewon vs AdamW</small></div>
  <div class="sk-stat"><b>92% vs 3%</b><small>SO-100 Cup-Align success after a chain of tasks in simulation, Skewon vs AdamW</small></div>
  <div class="sk-stat"><b>100M frames</b><small>Of continual pixel-based locomotion, across 19 dynamics changes</small></div>
</div>

<figure class="sk-fig">
  <div class="sk-grid sk-grid-5">
    <figure><img src="/images/skewon/real_pick.jpg" alt="SO-101 arm picking up a cube"><span>1. Pick</span></figure>
    <figure><img src="/images/skewon/real_place.jpg" alt="SO-101 arm placing a cube on a black disc"><span>2. Place</span></figure>
    <figure><img src="/images/skewon/real_stack.jpg" alt="SO-101 arm stacking a cube on a blue cube"><span>3. Stack</span></figure>
    <figure><img src="/images/skewon/real_cup.jpg" alt="SO-101 arm dropping a cube into a blue cup"><span>4. Cup</span></figure>
    <figure><img src="/images/skewon/real_sort.jpg" alt="SO-101 arm sorting a coloured cube into a red or green cup"><span>5. Sort</span></figure>
  </div>
  <figcaption>We fine-tune SmolVLA sequentially on these five real-world tasks. With Gram-Skewon the final policy succeeds 62% of the time on the last task, versus 37% with AdamW.</figcaption>
</figure>

## Why plasticity matters for robots

Imagine a robotic arm in a warehouse. Over its lifetime it learns a stream of tasks: picking boxes, placing them on conveyors, stacking them for storage. Meanwhile its joints wear, its grippers change, and new objects show up. The *big world hypothesis* says that an agent's capacity is always small relative to the world it lives in. So the robot can't learn everything up front. It has to **stay able to learn**.

Deep RL agents are known to lose this ability. Their networks become harder and harder to train as the data distribution shifts, a phenomenon called **loss of plasticity**. This has been well studied for control from low-dimensional proprioceptive states (joint angles and velocities). It is much less studied for control from **pixels**, which is how real manipulation systems usually perceive the world. That is the setting we focus on.

<figure class="sk-fig" style="max-width: 300px; margin-left: auto; margin-right: auto;">
  <img src="/images/skewon/humanoid_walk.jpg" alt="A humanoid robot image above a stick-figure schematic of its pose">
  <figcaption>The agent sees the high-dimensional image (top), but the underlying state, the pose (bottom), is low-dimensional. Image generated by ChatGPT.</figcaption>
</figure>

## The idea: constrain the matrices, not the loss

A deep network is mostly a composition of matrix multiplications. Many plasticity fixes act on those matrices *through the loss*. L2 regularization keeps them small. Parseval regularization (Chung et al., 2024) penalizes their distance from orthonormality, $$\lVert W W^\top - I \rVert_F$$.

We move the constraint out of the loss and **into the optimizer**. Every hidden weight matrix is kept exactly on the Stiefel manifold

$$
\mathrm{St}(d,k) = \{ X \in \mathbb{R}^{d\times k} : X^\top X = I_k \},
$$

so all of its singular values are pinned at one. A plain gradient step would leave the manifold. Instead, we pick an ascent direction in the tangent space and then *retract* back onto the manifold.

<figure class="sk-fig" style="max-width: 380px; margin-left: auto; margin-right: auto;">
  <img src="/images/skewon/stiefel_retraction.png" alt="Diagram of a gradient step leaving a curved manifold and a retraction mapping it back">
  <figcaption>A vanilla gradient step \(X + \eta D_X\) moves off the manifold; the retraction pulls it back to \(X^+\).</figcaption>
</figure>

We use **Skewon** (Solonko, 2026), which gives a closed-form, Muon-style update for this constrained problem. Given a momentum or gradient matrix $$M_X$$,

$$
N_X = \tfrac{1}{2}\left(M_X X^\top - X M_X^\top\right),
$$

$$
\Xi = \operatorname{polar}(N_X)\, X, \qquad X^{+} = R_X(\eta\, \Xi),
$$

where $$\operatorname{polar}(N) = UV^\top$$ for the SVD $$N = U\Sigma V^\top$$ (computed with Newton–Schulz iterations), and $$R_X$$ is a retraction.

**Why should this help?** We track plasticity through the network's **input–output Jacobian**,

$$
\frac{\partial F}{\partial x} = W_L D_{L-1}(x) W_{L-1} \cdots D_1(x) W_1 .
$$

If the Jacobian norm is large, the network's output responds to changes in its input. The agent can then turn reward feedback into useful learning signal. If it collapses, the network goes flat and stops responding. Fixing every $$W_l$$'s singular values at one stops the linear layers from adding ill-conditioning. (The activation derivatives $$D_l(x)$$ still matter, though.)

## A toy problem that isolates the effect

To compare the optimizers cleanly, with full hyper-parameter sweeps, we built a small pixel-based **linear–quadratic regulator** (LQR) environment in JAX. Eight blobs, coupled by "strings," must be driven onto their red crosses. The 16-dimensional state is seen only through an 8,192-pixel image. Each chain alternates between two kinds of task:

- tasks that need strong feedback control of an unstable plant;
- heavily damped tasks that only need a large constant push.

So the optimal sensitivity of the policy swings down and back up at every task change.

<figure class="sk-fig">
  <div class="sk-grid" style="grid-template-columns: 1.45fr 1fr; align-items: center;">
    <img src="/images/skewon/blob_env.jpg" alt="Eight tiles each containing a light blob and a red target cross">
    <img src="/images/skewon/lqr_scatter.png" alt="Scatter plot of mean task return against input-output Jacobian norm for AdamW and Skewon">
  </div>
  <figcaption><b>Left:</b> the blob environment. <b>Right:</b> mean return against input–output Jacobian norm for 24 task chains, each tuned separately for each optimizer. AdamW's return falls roughly log-linearly with its Jacobian norm. Skewon (all 24 points overlap) keeps a large Jacobian and near-optimal return.</figcaption>
</figure>

Even with the best of 12 hyper-parameter settings per chain, AdamW gets stuck on many chains, and the chains where it does worst are exactly the ones where its Jacobian norm has shrunk the most. Skewon solves every chain.

## Continual locomotion from pixels (MuJoCo)

Next, we train PPO agents from $$84\times84$$ RGB frames on four continual locomotion variants. The dynamics change every 5M frames:

- **Ant-Friction** (15 tasks): floor friction drawn from a bimodal distribution.
- **Ant-Fluid** (10 tasks): the ant is immersed in fluid of varying density.
- **Humanoid-Actuator** (20 tasks): motor gear scales from $$[0.2, 0.3] \cup [2.4, 2.5]$$.
- **Humanoid-Joint-Damping** (20 tasks): damping from $$[2, 8] \cup [40, 80]$$.

All three methods share the same architecture, layer norm, reward normalization and Swish activations, so the AdamW baseline already includes the standard plasticity fixes. They differ only in how six key matrices are treated: unconstrained (AdamW), penalized (Parseval) or constrained (Skewon).

<figure class="sk-fig">
  <div class="sk-grid sk-grid-2">
    <figure><img src="/images/skewon/ant_friction.png" alt="Normalized return over 75M frames on Ant-Friction"><span>Ant-Friction, 15 tasks</span></figure>
    <figure><img src="/images/skewon/ant_fluid.png" alt="Normalized return over 50M frames on Ant-Fluid"><span>Ant-Fluid, 10 tasks</span></figure>
    <figure><img src="/images/skewon/humanoid_actuator.png" alt="Normalized return over 100M frames on Humanoid-Actuator"><span>Humanoid-Actuator, 20 tasks</span></figure>
    <figure><img src="/images/skewon/humanoid_jointdamping.png" alt="Normalized return over 100M frames on Humanoid-Joint-Damping"><span>Humanoid-Joint-Damping, 20 tasks</span></figure>
  </div>
  <div class="sk-legend">
    <span><i style="background:#2E3338"></i>Skewon</span>
    <span><i style="background:#D9B45B"></i>Parseval</span>
    <span><i style="background:#6E9BC9"></i>AdamW</span>
  </div>
  <figcaption>Mean normalized return across ten task sequences per variant. Within each task, returns are rescaled to [0, 1] using the pooled min and max over the three methods.</figcaption>
</figure>

Skewon pulls ahead early and stays ahead as tasks pile up. The gap is starkest on **Ant-Fluid**, where AdamW's normalized return collapses to $$0.010 \pm 0.003$$ by the end of the chain. On both humanoid variants, Skewon is still ahead at 95–100M frames, after 19 dynamics changes, while AdamW and Parseval plateau or decline.

The Jacobian tells the same story. Skewon is the only method whose Jacobian norm doesn't shrink over the chain:

<figure class="sk-fig">
  <div class="sk-grid sk-grid-2">
    <figure><img src="/images/skewon/jac_ant_friction.png" alt="Jacobian Frobenius norm at the start of each task on Ant-Friction"><span>Ant-Friction</span></figure>
    <figure><img src="/images/skewon/jac_ant_fluid.png" alt="Jacobian Frobenius norm at the start of each task on Ant-Fluid"><span>Ant-Fluid</span></figure>
  </div>
  <div class="sk-legend">
    <span><i style="background:#2E3338"></i>Skewon</span>
    <span><i style="background:#D9B45B"></i>Parseval</span>
    <span><i style="background:#6E9BC9"></i>AdamW</span>
  </div>
  <figcaption>Frobenius norm of the policy's input–output Jacobian at the start of every task. By the end of the chain, Skewon keeps 0.91× (Friction) and 1.11× (Fluid) of its initial value. Parseval keeps 0.70× / 0.99×, and AdamW keeps 0.62× / 0.84×. These Jacobian measurements rank the methods in the same order as their returns.</figcaption>
</figure>

## Continual manipulation in simulation (ManiSkill)

We also train two simulated arms on chains of manipulation tasks. Each task starts from the previous task's final checkpoint. The policy sees camera images plus proprioception, with no privileged object positions. In *Sort*, the arm must work out from pixels alone whether it is holding a cube or a ball.

<figure class="sk-fig">
  <div class="sk-grid sk-grid-5">
    <figure><img src="/images/skewon/so100_1_grasp.jpg" alt="SO-100 grasp task"><span>Grasp</span></figure>
    <figure><img src="/images/skewon/so100_2_place.jpg" alt="SO-100 place task"><span>Place</span></figure>
    <figure><img src="/images/skewon/so100_3_stack.jpg" alt="SO-100 stack task"><span>Stack</span></figure>
    <figure><img src="/images/skewon/so100_4_cupalign.jpg" alt="SO-100 cup-align task"><span>Cup-Align</span></figure>
    <figure><img src="/images/skewon/so100_5_sort.jpg" alt="SO-100 sort task"><span>Sort</span></figure>
  </div>
  <div class="sk-grid sk-grid-4" style="margin-top: 0.8em;">
    <figure><img src="/images/skewon/panda_1_grasp.jpg" alt="Panda grasp task"><span>Grasp</span></figure>
    <figure><img src="/images/skewon/panda_2_place.jpg" alt="Panda place task"><span>Place</span></figure>
    <figure><img src="/images/skewon/panda_3_stack.jpg" alt="Panda stack task"><span>Stack</span></figure>
    <figure><img src="/images/skewon/panda_4_pyramid.jpg" alt="Panda pyramid task"><span>Pyramid</span></figure>
  </div>
  <figcaption>Top: the five-task SO-100 chain. Bottom: the four-task Franka Panda chain.</figcaption>
</figure>

<div class="sk-table-wrap">
<table class="sk-table">
  <thead>
    <tr><th rowspan="2">SO-100 task</th><th colspan="2">Chain</th><th colspan="2">Scratch</th></tr>
    <tr><th>AdamW</th><th>Skewon</th><th>AdamW</th><th>Skewon</th></tr>
  </thead>
  <tbody>
    <tr><td>Grasp</td><td><b>.70</b> ± .05</td><td>.64 ± .05</td><td><b>.88</b></td><td>.75</td></tr>
    <tr><td>Place</td><td>.75 ± .08</td><td><b>.92</b> ± .04</td><td>.88</td><td><b>.94</b></td></tr>
    <tr><td>Stack</td><td>.31 ± .11</td><td><b>.47</b> ± .07</td><td>.56</td><td><b>.69</b></td></tr>
    <tr><td>Cup-Align</td><td>.03 ± .02</td><td><b>.92</b> ± .04</td><td>.81</td><td>.81</td></tr>
    <tr><td>Sort</td><td>.23 ± .14</td><td><b>.63</b> ± .15</td><td>.88</td><td><b>1.00</b></td></tr>
  </tbody>
</table>
</div>

<div class="sk-table-wrap">
<table class="sk-table">
  <thead>
    <tr><th rowspan="2">Panda task</th><th colspan="2">Chain</th><th colspan="2">Scratch</th></tr>
    <tr><th>AdamW</th><th>Skewon</th><th>AdamW</th><th>Skewon</th></tr>
  </thead>
  <tbody>
    <tr><td>Grasp</td><td><b>.92</b> ± .04</td><td>.73 ± .04</td><td>1.00</td><td>1.00</td></tr>
    <tr><td>Place</td><td><b>1.00</b> ± .00</td><td>.96 ± .04</td><td>1.00</td><td>1.00</td></tr>
    <tr><td>Stack</td><td>.75 ± .06</td><td><b>.96</b> ± .02</td><td><b>.81</b></td><td>.69</td></tr>
    <tr><td>Pyramid</td><td>.58 ± .08</td><td><b>.94</b> ± .04</td><td>.00</td><td>.00</td></tr>
  </tbody>
</table>
</div>

<p style="font-size: 0.82em; color: #555;"><i>Chain</i>: each task is warm-started from the previous one (mean ± s.e. over seeds). <i>Scratch</i>: each task is trained independently from a random initialisation.</p>

AdamW starts out better on Grasp, but Skewon wins on every task from Stack onward in both chains. Two results stand out:

- **Cup-Align (SO-100):** both optimizers reach 0.81 from scratch. In the chain, AdamW collapses to **0.03** while Skewon reaches **0.92**. The task is learnable, but AdamW has lost the ability to learn it.
- **Pyramid (Panda):** neither optimizer learns it from scratch (0.00). Sequential training makes it learnable, and Skewon gets more out of the chain (**0.94** vs 0.58).

## Gram-Skewon: plasticity for pretrained models

Vision-language-action (VLA) models are pretrained, and their weights are nowhere near orthonormal. Projecting them onto the Stiefel manifold would wreck what they have learned. So we generalize the constraint. Rather than $$W^\top W = I$$, we preserve each matrix's **pretrained Gram matrix**:

$$
\mathcal{M}_{G_0} = \{ W \in \mathbb{R}^{m\times n} : W^\top W = G_0 \}, \qquad G_0 = W_0^\top W_0 .
$$

Stiefel is the special case $$G_0 = I$$. This keeps every singular value of the pretrained matrix fixed and lets the matrix *rotate* freely. Three observations make this cheap and clean:

- **The tangent space doesn't change.** For any full-rank $$W$$, the tangent space is $$\{YW : Y \text{ skew-symmetric}\}$$, the same parametrisation Skewon uses. So the steepest-descent direction still has a closed form,

  $$
  Y^\star = -\operatorname{msign}\!\big(\operatorname{skew}(G W^\top)\big), \qquad \Delta W = \eta\, Y^\star W .
  $$

- **Steps are measured as rotations.** We measure step size by the spectral norm of the generator $$Y$$, which rotates the columns rigidly. This bounds the *relative* change in every output direction by $$\eta$$, so a single learning rate works across layers whose scales differ by more than 10×. It also protects the fragile, near-singular directions of the checkpoint.
- **A one-time polar split.** Write $$W_0 = V_0 P$$ with $$V_0$$ orthonormal and $$P = (W_0^\top W_0)^{1/2}$$. The Gram-Skewon step on $$W$$ is then exactly a plain Skewon step on $$V$$, with the gradient pulled back as $$G P$$. We never update or invert $$P$$, so $$W^\top W = P V^\top V P = G_0$$ holds by construction.

<div class="sk-algo" markdown="1">
**Gram-Skewon (one weight matrix)**

1. Once: take the SVD $$W_0 = U\,\mathrm{diag}(s)\,Q^\top$$, floor the spectrum at $$s_{\max}/\kappa_{\max}$$, and set $$V \leftarrow UQ^\top$$, $$P \leftarrow Q\,\mathrm{diag}(s)\,Q^\top$$.
2. Each step: compute the Euclidean gradient $$G$$ at $$W = VP$$ and apply Nesterov momentum to get $$\tilde G$$.
3. $$N \leftarrow \operatorname{skew}(\tilde G P V^\top)$$, then $$Y \leftarrow -\operatorname{msign}(N)$$ (Newton–Schulz / Polar Express).
4. $$V \leftarrow \operatorname{qr}(V + \eta\, Y V)$$, and $$W \leftarrow VP$$.
</div>

The spectrum floor in step 1 ($$\kappa_{\max} = 30$$) only touches a handful of freshly initialized cross-attention layers in SmolVLA, changing them by 1–2% in Frobenius norm. After that, the condition number stays bounded for the whole run, with no regularizer and no weight decay. Memory is about 1× the parameter count, versus 2× for AdamW's moment estimates.

## Real-world continual fine-tuning on the SO-101

We continually fine-tune **SmolVLA**'s action expert (99.9M trainable parameters; the VLM backbone is frozen) by behavior cloning on teleoperated demonstrations for five physical tasks: Pick, Place, Stack, Cup, and a language-conditioned Sort. In Sort, the same scene admits two correct goals, so only the instruction (*"Put the yellow cube in the green cup"*) tells the policy what to do. There's no replay of earlier data, and the optimizer is the **only** difference between the two arms.

<div class="sk-table-wrap">
<table class="sk-table">
  <thead>
    <tr><th rowspan="2">Task</th><th colspan="3">AdamW</th><th colspan="3">Gram-Skewon</th></tr>
    <tr><th>Trials</th><th>Success</th><th>Rate</th><th>Trials</th><th>Success</th><th>Rate</th></tr>
  </thead>
  <tbody>
    <tr><td>Pick</td><td>50</td><td>23</td><td>46.0%</td><td>50</td><td>25</td><td><b>50.0%</b></td></tr>
    <tr><td>Place</td><td>40</td><td>39</td><td><b>97.5%</b></td><td>40</td><td>36</td><td>90.0%</td></tr>
    <tr><td>Stack</td><td>50</td><td>6</td><td>12.0%</td><td>40</td><td>15</td><td><b>37.5%</b></td></tr>
    <tr><td>Cup</td><td>40</td><td>32</td><td>80.0%</td><td>40</td><td>38</td><td><b>95.0%</b></td></tr>
    <tr><td>Sort</td><td>60</td><td>22</td><td>36.7%</td><td>60</td><td>37</td><td><b>61.7%</b></td></tr>
    <tr class="sk-total"><td>Overall</td><td>240</td><td>122</td><td>50.8%</td><td>230</td><td>151</td><td><b>65.7%</b></td></tr>
  </tbody>
</table>
</div>

Gram-Skewon improves on Stack, Cup and Sort, raising final-task success from **36.7% to 61.7%** and overall success from 50.8% to 65.7%. The cost is about 2× training time per task.

## Takeaways

- **Loss of plasticity is real in pixel-based control**, and the standard fixes (layer norm, L2, reward normalization, Swish, Parseval) don't remove it.
- **Changing only the optimizer** to Skewon, a Stiefel-constrained update, sustains learning for up to 100M frames, with no extra loss terms or regularization coefficients.
- **The input–output Jacobian norm** tracks plasticity: it stays stable under Skewon and shrinks under AdamW and Parseval, in the same order as their returns.
- **Gram-Skewon** carries the idea over to pretrained models by preserving $$W^\top W$$ rather than forcing it to $$I$$. It gives sizeable gains in continual VLA fine-tuning on real hardware.

Open questions remain. A theoretical account of how orthogonality and Gram preservation shape the Jacobian, and through it learning, is still missing. The constrained update also costs 1.4–2× in wall-clock time.
