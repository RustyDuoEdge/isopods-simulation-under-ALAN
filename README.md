# 🦠 Isopods Simulation Under Artificial Light at Night

[中文版](README_ZH.md) | English

A zero-dependency, pure front-end (Vanilla JS & Canvas) digital behavioral biology laboratory designed to simulate and observe the self-organizing behaviors of Isopod (pill bug) populations.

By integrating mathematical modeling with individual behavioral rules, this simulation demonstrates how a crowd of independent agents can spontaneously construct a stable, topological conduit without any centralized orchestration.

[👉 Live Laboratory Demo](https://rustyduoedge.github.io/isopods-simulation-under-ALAN/)

---

## 🔬 Dual-Core Behavioral Dynamics

Within the virtual environment, the heading and velocity of each individual agent (`SmartIsopod`) are simultaneously governed by environmental phototaxis and multi-body neighborhood social torques, formulated into a unified steering framework (`total_omega`):

### 1. 3-Phase Phototaxis (Fermi-Dirac Gradient)
The central lamp projects a photon dispersion field modeled by a **Fermi-Dirac smooth step decay**. By evaluating inputs from its left and right antennae (`I_left`, `I_right`) separated by a 30° angle, an agent triggers three distinct behavioral phases based on intensity thresholds:
* **Core Over-exposure Zone ($I > I_{max}$ - Within Red Perimeter)**: Exceeds physiological tolerance. The agent experiences intense negative phototaxis, generating a massive steering torque (`omega_light = k_light_turn * (I_right - I_left)`) to execute an immediate U-turn away from the core.
* **Optimization Conduit Safe-Zone ($I_{min} \le I \le I_{max}$ - Orange-Red Lane)**: Matches the agent's psychological comfort preference. Steering torque is deactivated (`omega_light = 0.0`), allowing agents to align and cruise steadily along the intensity isolines (tangent vectors).
* **Dim Outer Margin ($I_{threshold} \le I < I_{min}$ - Gray to Orange)**: Triggers low-security exploration due to fading light. Generates a weak positive phototaxis torque (`omega_light = k_light_turn * (I_left - I_right)`) to pull wandering agents back toward the shelter threshold.

---

### 2. Inter-Agent Social Dynamics
Beyond light interaction, each agent continuously evaluates distance and heading vectors relative to all neighbors within its local perception radius (`R_c`). By iterating through the multi-body neighborhood matrices, the engine replicates complex microscopic traffic flows:

#### 1. Vicsek Polarized Alignment (Conformity Effect)
* **Code Implementation**: `omega_align += Math.sin(other.theta - this.theta)`
* **Behavior Pattern**: The classic Vicsek multi-agent alignment model. Agents calculate heading deltas with neighbors within `R_c`. Driven by a sinusoidal torque response, they mimic their peers, correcting their headings to match the local flow.
* **Emergence**: In total darkness (without lamp interference), this mutual conformity dominates the system, causing chaotic agents to spontaneously polarize into a unified flocking flow across the toroidal wrap space.

#### 2. 2D Avoidance Vector Masking (Rigid Collision Shield)
* **Code Implementation**: `omega_repel += (u_x * (-Math.sin(this.theta)) + u_y * Math.cos(this.theta))`
* **Behavior Pattern**: Enforces a rigid shell boundary (`R_r = 0.42`). When two agents breach this physical perimeter, an aggressive repulsive torque (`omega_repel`) slides their headings apart, maintaining a realistic solid-particle granularity.
* **Emergence**: This provides a horizontal anti-collision shield in 2D space, ensuring agents bounce away from one another during chaotic movements to preserve a distinct physical presence.

#### 3. Car-Following Braking (Micro-Traffic Deceleration)
* **Code Implementation**: `let factor = 1.0 - Math.exp(-(dist - this.R_r) / 0.5); factor = Math.max(0.10, factor);`
* **Behavior Pattern**: When an agent detects obstacles ahead (heading cosine `> 0.2`), it activates an automated car-following adaptive deceleration routine. As the forward distance shrinks, the speed modifier drops exponentially, enforcing a heavy `0.10` braking floor.
* **Emergence**: This effectively eliminates unnatural overlapping artifacts during high-density bottlenecks, resulting in visual bumper-to-bumper traffic jams along the conduit.

#### 4. Quasi-3D Trampling Bypass (Vertical Stacking Escape)
* **Code Implementation**: `let isStacking = Math.random() < trampleProbability; factor = Math.max(0.65, factor);`
* **Behavior Pattern**: When 2D horizontal traffic completely chokes under extreme density, a stochastic longitudinal breakdown rule triggers. Under a given `trampleProbability`, an agent bypasses both 2D avoidance and braking penalties, maintaining a `0.65` velocity baseline to climb over its neighbors.
* **Emergence**: The agent's state transitions to `isTrampling = true`, flashing into **Neon Pink** with an accented stroke width. This "elevation shift" resolves spatial friction via 3D multi-layer stacking, mimicking how real-world isopods pile up in cramped corners.

---

## 🎛️ Control Panel Guide

You can manipulate environmental physics or adjust genetic instincts via the range sliders to watch how steady-state equilibriums fracture or reorganize:

### ⚙️ Physical Environment (Left Panel)
* **Simulation Speed (`speedSlider`)**: Adjusts the global time-step scaling multiplier (`1.0x - 4.0x`) to accelerate the emergence of stable states.
* **Isopod Count (`countSlider`)**: Dynamically scales the active agent array limits (`10 - 300` agents) via runtime slicing (`isopods.splice`).
* **Light Radius (`radiusSlider`)**: Modifies the physical geometric disk radius $R_{disk}$ of the central lamp source.

### 🧬 Biological Instincts (Right Panel)
* **Trample Probability (`trampleSlider`)**: Controls the likelihood (`0% - 100%`) of an agent climbing onto another during a jam. At `0%`, severe rigid lockups occur; higher settings generate vibrant neon pink layered stackings.
* **1. Outer Threshold (`thresholdSlider`, Gray Dashed Line)**: Defines the absolute peripheral boundary $I_{threshold}$ for light detection.
* **2. Positive Taxis Boundary (`iminSlider`, Orange Dashed Line)**: Defines the outer perimeter $I_{min}$ of the safe conduit.
* **3. Negative Taxis Boundary (`imaxSlider`, Red Dashed Line)**: Defines the critical tolerance threshold $I_{max}$ of the core over-exposure zone.

> ⚠️ **Geometrical Frustration Note**
>
> Compressing the gap between $I_{min}$ and $I_{max}$ excessively close ($\le 2.0$) forces a spatial contradiction. Agents bounce violently between panic escape and parallel tracking loops, causing a warning banner on the UI:
> `⚠️ 临界阻挫！两区闭合，交界线乱哆嗦中！(Sliding mode chattering on boundary!)`

---

## 💡 Inspiration & Acknowledgments

* **Academic Foundation**: The multi-agent behavioral aggregation mechanics are deeply inspired by the ecological paper: [*How do isopods aggregate? Formulating and simulating a model based on individual behavior* (Wiley Online Library)](https://onlinelibrary.wiley.com/doi/10.1002/ece3.73487).
* **AI Collaboration**: Full development pipelines of the core Canvas engine, internationalization infrastructure (`i18nPack`), and technical documentation layout were refined in collaborative workflows alongside Google Gemini LLM.

---

## 🚀 Quick Start

Zero dependencies (No Node.js, No npm, No Webpack). Clone and use right out of the box:

```bash
# 1. Clone the repository
git clone https://github.com/RustyDuoEdge/isopods-simulation-under-ALAN.git
cd isopods-simulation-under-ALAN

# 2. Open index.html directly in any modern browser
firefox index.html
