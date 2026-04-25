# PhD Thesis — Algorithms and Tools for Feasibility Analysis and Optimal Experiment Design in Pharmaceutical Manufacturing

**Author:** Kennedy P. Kusumo
**Institution:** Imperial College London — Sargent Centre for Process Systems Engineering, Department of Chemical Engineering
**Submitted:** September 2021
**Degree:** Doctor of Philosophy in Chemical Engineering and the Diploma of Imperial College
**Duration:** January 2018 – January 2022

**Supervisors:**
- Prof. Benoît Chachuat (Imperial College London) — primary supervisor, meticulous guidance
- Prof. Nilay Shah (Imperial College London) — academic freedom, PSE community leadership
- Prof. Salvador García-Muñoz (Eli Lilly and Company) — industrial direction, practicality vs. generality balance

**Key collaborators:** Dr. Lucian Gomoescu, Dr. Kamal Kuriyan, Dr. Shankar Vaidyaraman (Eli Lilly)
**Industrial collaborator:** Eli Lilly and Company
**Sponsoring body:** PharmaSEL-Prosperity Partnership (PI: Prof. Claire Adjiman)
**Funding:** EPSRC

---

## Thesis Abstract

The growing need for systematic and risk-based approaches to tackling challenges within the process industry prompted research into model-based methodologies. Of particular interest is the **Quality by Design (QbD)** initiative in the pharmaceutical industry.

This thesis reports contributions to two areas: **feasibility analysis** and **optimal experiment design**.

**Part 1 — Feasibility Analysis (Chapters 2–4):** Adaptation of the nested sampling algorithm for probabilistic design space characterization.
- Ch 2: Initial adaptation of nested sampling for probabilistic design space characterization — competitive with optimization-based approaches
- Ch 3: Two strategic + one implementation improvements, enabling larger problems
- Ch 4: Design centering methodology — computes the largest volume box inside the probabilistic design space (practical format for communicating to process operators)

**Part 2 — Optimal Experiment Design (Chapters 5–7):** Novel techniques for calibrating nonlinear models under uncertainty.
- Ch 5: Continuous-effort methodology for optimal experimental campaigns
- Ch 6: Bi-objective Average/CVaR framework for risk-mitigated experiment design under model uncertainty
- Ch 7: Robust optimal experimental campaigns under operational constraints — integrates nested sampling (Part 1) with continuous-effort design (Ch 5)

**Software deliverables:**
- **DEUS** — Python package for feasibility analysis (nested sampling)
- **Pydex** — Python package for optimal experiment design

---

## Motivation and Context

### Quality by Design (QbD) in Pharma
The ICH Q8 initiative promotes a scientific, risk-based approach to pharmaceutical process and product development. Central to this is the **Design Space (DS)**: *"the multidimensional combination and interaction of input variables and process parameters that have been demonstrated to provide assurance of quality."*

DS characterization requires well-parameterized mechanistic process models. The challenge: building these models demands informative experiments, which are expensive in pharma (scarce materials, safety constraints, regulatory oversight).

### Process Systems Engineering (PSE)
Kennedy's work sits within PSE — a subfield of chemical engineering using mathematics and computers to tackle complex process problems holistically. PSE broadened from traditional refining/chemicals to pharmaceuticals, biological processes, energy systems, and agriculture.

The research directly addresses:
- **How to characterise the design space** robustly under model parameter uncertainty
- **How to design experiments** that most efficiently calibrate models, especially early in development when parameter estimates are poor

---

## Part 1: Feasibility Analysis

### Background: The Design Space Problem

For a steady-state process with uncertain model parameters θ ~ p(θ), the probabilistic design space at reliability α is:

```
DS(α) = { u ∈ U | P(g(x,u,θ) ≤ 0) ≥ α }
```

where u are process inputs, g are quality constraints, and P is the probability over θ. High parameter uncertainty → smaller DS at a given reliability → less operating room → regulatory risk.

Standard approach (optimization-based): expensive, scales poorly with dimensionality.

### Chapter 2: Nested Sampling for Probabilistic Design Space Characterization

**Key idea:** Adapt the nested sampling algorithm (originally for Bayesian evidence computation) to directly sample from the probabilistic design space at multiple reliability levels simultaneously.

**Algorithm:** Instead of sampling from a likelihood-weighted posterior, samples are sorted by their feasibility probability. The algorithm iteratively replaces the least feasible live point with a new sample, naturally building up a map of the probability landscape.

**Two sampling approaches compared:**
- Standard Monte Carlo with Sobol quasi-random sampling (Algorithm 1)
- Nested sampling (Algorithm 2)

**Result:** Nested sampling is competitive with optimization-based approaches in computational time while providing sampling-based flexibility (no gradient requirements, works with black-box simulators).

**Industrial case studies:**
1. **Michael Addition Reaction** — 2D DS, Nθ = 100 and 1,000 uncertainty scenarios
2. **Suzuki Coupling Reaction** — 4D DS (temperature T, oxygen fraction yO₂, batch duration τ, catalyst equivalent RPd|SM2). NL = 10,000 live points, Nθ = 1,000 scenarios. Results shown as trellis plot of 2D projections.

**Key figure (Fig 2.5):** Trellis chart of the 4D Suzuki DS — samples coloured by reliability range (red: α ≥ 0.85, yellow: 0.05 ≤ α < 0.85, blue: α < 0.05).

### Chapter 3: Improved Nested Sampling for Design Space

**Two strategic improvements + one implementation improvement** to the Chapter 2 algorithm:
- Reduced computational burden
- Enables larger problems

**Demonstrated on:** Suzuki coupling reaction. Algorithm 3 achieves DS at α* = 0.85 reliability (Fig 3.1).

Computational statistics show significant speedup vs Chapter 2.

### Chapter 4: Design Centering for Probabilistic Design Space

**Problem:** The probabilistic DS is a complex, irregular shape. Process operators need a simple, communicable operating region.

**Solution:** Compute the **largest volume hyperrectangle (box)** inscribed within the probabilistic DS at a given reliability level.

**Methodology:**
1. Run NS-DS (Algorithm from Ch 2/3) to generate scattered point samples with feasibility probability estimates
2. Train a **Multilayer Perceptron (MLP)** surrogate on the probability map
3. Solve the **design centering optimization** (maximize box volume subject to MLP feasibility constraint)

**Case study: Sequential Reaction**
- NL = 1,000 live points, NR = 500 replacement proposals, Nθ = 1,000 random scenarios
- Computed largest hyperrectangle at α = 0.55 (Fig 4.1)
- Modified solution enforces minimum temperature window of 10 K
- MLP parity plot shows excellent fit (Fig 4.2)
- Computed box reported in Table 4.2

---

## Part 2: Optimal Experiment Design

### Background: Model-Based Design of Experiments (MBDoE)

The Fisher Information Matrix (FIM) M(θ, φ) approximates the inverse of the parameter covariance matrix Σ_θ:

```
M(θ,φ) = Σ_oo' (1/σ²_oo') · S_o^T · S_o' + M₀
```

Standard design criteria:
| Criterion | Optimise | Geometric meaning |
|-----------|----------|-------------------|
| D-optimal | max det(M) | Minimise volume of confidence ellipsoid |
| A-optimal | max tr(M) | Minimise sum of parameter variances |
| E-optimal | max λ_min(M) | Minimise longest axis of ellipsoid |

**Key limitation for nonlinear models:** FIM depends on θ → designs are locally optimal only. Early-stage model development (when parameter estimates are uncertain) is where this hurts most.

### Chapter 5: Continuous-Effort Approach to MBDoE

**Motivation:** Sequential experiment design (run one experiment, update model, repeat) dominates recent literature, but overlooks the concept of **experimental effort** — how many runs to allocate to each experimental condition.

**Continuous-effort framework:**
- Experimental design φ specified as continuous support points + continuous efforts ω_i (ω_i ≥ 0, Σ ω_i = 1)
- The FIM becomes: M(θ, Φ) = N_total · Σ_i ω_i · M(θ, φ_i)
- Optimize both support points and efforts simultaneously
- Rounding procedure converts continuous efforts to integer run counts

**Case study:** D-optimal continuous design with candidate experiment grid (Fig 5.1 — effort allocated as proportional circle sizes at support points).

### Chapter 6: Risk-Mitigated Experiment Design (Average/CVaR)

**Problem:** With uncertain θ early in model development, the locally D-optimal design (using current best θ estimate) may be highly suboptimal or even uninformative if the true θ differs significantly.

**Solution:** Bi-objective optimisation over parameter uncertainty scenarios:
- **Average criterion**: E[φ(M(θ,Φ))] — good average performance
- **CVaR criterion (Conditional Value at Risk)**: expected criterion value in the worst β% of scenarios (robust tail performance)

**CVaR definition (Fig 6.1 and 6.2):**
```
CVaR_β = E[φ | φ ≤ VaR_β]
```
where VaR_β is the β-quantile. CVaR penalises the worst-case scenarios more than VaR.

**ε-constraint formulation:** Solve a sequence of single-objective problems (fix CVaR target, maximise average) to trace the Pareto frontier.

**Computational framework:**
- Discretise parameter uncertainty into Nπ Monte Carlo scenarios
- Continuous-effort design for each scenario
- CASADI + SUNDIALS IDAS solver for sensitivity analysis (automatic differentiation)
- Forward Sensitivity Analysis for ODE systems
- Implemented in Pydex

**Case studies:**

1. **First-Order Response Model** (exponential: y = θ₁ exp(-θ₂t))
   - Analytical sensitivities
   - 10 Pareto-efficient designs computed
   - Worst 25% scenarios: θ₁ ∈ [-10, -7.5) shown in red (Fig 6.4)
   - CDFs comparing average vs CVaR designs (Fig 6.5)
   - Table 6.3: 3 efficient designs with rounded efforts for varying Nt

2. **Fed-batch Reactor**
   - 243 experiment candidates
   - 5 Pareto-efficient campaigns (Table 6.4, Fig 6.6 and 6.7)

3. **Suzuki Coupling Reaction**
   - 10 out of 28 model parameters identifiable (quantitative identifiability study)
   - Unique Pareto-efficient campaign (Table 6.5)
   - 11 equally-spaced samples per batch

**Run-time details (Table 6.2):** AMD Ryzen 2600x single core for optimisation; Intel Xeon E5-2697 V2 @ 2.7 GHz for sensitivity analysis.

### Chapter 7: Robust Design Under Operational Constraints

**Problem:** Experimental space is often restricted by operational constraints — not all combinations of inputs are achievable simultaneously (e.g., a stirred-tank reactor can't immediately achieve any temperature with any throughput; dynamics and safety constraints limit what's feasible during a real experiment).

**Key contribution:** Tractable computational framework combining:
1. **Nested sampling (from Part 1)** to characterise the restricted experimental space (at a given reliability)
2. **Continuous-effort design (from Ch 5)** to find optimal campaigns within that space

**Problem statement:**
- Restricted experimental space R(α) = {φ | P(operational constraints satisfied) ≥ α}
- Robust OED: optimise experimental campaign within R(α) subject to model uncertainty

**Two-step methodology (Fig 7.1):**
1. Characterise R(α) using nested sampling → set of feasible candidate experiments
2. Solve robust OED (average D-optimal or CVaR D-optimal) using the feasible candidates

**Illustrative example:** Response-surface model (RSM) — restricted to 1.85 ≤ y ≤ 3 at 85% reliability. 3,885 candidates sampled. D-optimal continuous design shown (Fig 7.2).

**Industrial case study: Jacketed Stirred-Tank Reactor (CSTR) for Throughput Increase**
- Goal: calibrate model to increase reactor throughput
- Operational constraints: minimum concentration c_B^min, maximum temperature T^max
- Default start-up followed by 2× throughput increase after 100 minutes
- Uncertain parameters: (UA, θ₀) ∈ [-4.101, -3.896] × [350, 400]
- Restricted experimental space at 95% reliability (Fig 7.5 — 3D scatter, marker size = duration τ_d)

Three experimental designs compared (Figs 7.3–7.7, Tables 7.3–7.4):
1. **Unrestricted locally D-optimal** — ignores feasibility constraints, may be infeasible
2. **Restricted locally D-optimal** — within R(0.95), uses nominal θ
3. **Restricted average D-optimal** — within R(0.95), robust over uncertainty scenarios

---

## Software: DEUS and Pydex

### DEUS (Design of Experiments Under Uncertainty via Sampling)
- **Purpose:** Feasibility analysis, design space characterization
- **Core algorithm:** Nested sampling adapted for probabilistic DS
- **Key outputs:** Point samples with feasibility probability estimates; probability maps; design centering
- **Language:** Python

### Pydex (Python Design of Experiments)
- **Purpose:** Optimal experiment design for parameter estimation (MBDoE-PE)
- **Capabilities:**
  - D/A/E-optimal design (standard FIM-based)
  - Continuous-effort formulation
  - Bi-objective Average/CVaR robust design
  - Integration with CASADI for sensitivity analysis
  - Supports DAE/ODE systems via Pyomo.DAE + SUNDIALS IDAS
  - Bootstrap parameter estimation
  - Parameter identifiability analysis
- **Language:** Python (Pyomo, CASADI, SUNDIALS)

---

## Chapter Structure Summary

| Chapter | Title | Part | Key contribution |
|---------|-------|------|-----------------|
| 1 | Introduction + Literature Review | — | Overview, motivation, research aims |
| 2 | Bayesian Approach to Probabilistic DS: Nested Sampling | I | NS-DS algorithm; Michael addition + Suzuki case studies |
| 3 | Nested Sampling Tailored to Bayesian DS | I | 3 improvements to Ch 2; computational speedup |
| 4 | Design Centering for Probabilistic DS | I | MLP surrogate + largest inscribed box |
| 5 | Continuous Optimal Experimental Designs | II | Continuous-effort MBDoE framework |
| 6 | Continuous-Effort Design with Risk Mitigation | II | Bi-objective Average/CVaR Pareto framework |
| 7 | Robust Campaigns Under Operational Constraints | II | NS (Part 1) + continuous-effort (Ch 5) integrated |
| 8 | Concluding Remarks | III | Future work: identifiability, goal-oriented DS OED |

---

## Future Work (Chapter 8)

Two directions identified:
1. **Minimal subset of measurable responses for model identifiability** — systematic approach to selecting which outputs to measure
2. **Goal-oriented experiment design tailored to design space characterization:**
   - Direct method
   - Tailored V-optimal criterion

---

## Key Industrial Case Studies

| Reaction / System | Chapters | DS Dimensions | Key insight |
|-------------------|----------|---------------|-------------|
| Michael Addition Reaction | 2 | 2D | DS computed with 100 and 1,000 θ scenarios |
| Suzuki Coupling Reaction | 2, 3, 6 | 4D (T, yO₂, τ, RPd/SM2) | Large trellis chart DS; unique Pareto OED campaign with 10/28 identifiable params |
| Sequential Reaction | 4 | 2D | Largest inscribed box via design centering |
| Fed-batch Reactor | 6 | — | 5 Pareto-efficient campaigns; 243 candidates |
| Jacketed CSTR | 7 | 3D (τ_d, startup config) | Throughput increase; 95% reliability constraint |
| First-order Response Model | 6 | — | Analytical reference case; 10 Pareto designs |

---

## Thesis Quotes (epigraph)

> "Everything must be taken into account. If the fact will not fit the theory — let the theory go."
> — Agatha Christie, *The Mysterious Affair at Styles*

> "It is the simple suggestion that the only valid reason for rejecting a statistical hypothesis is that some alternative explains the observed events with a greater degree of probability."
> — Herbert I. Weisberg, *Willful Ignorance: The Mismeasure of Uncertainty*

---

## Source File

`D:\OneDrive\Imperial OneDrive Copy\OneDrive - Imperial College London\PhD\Kusumo-K-2022-PhD-Thesis.pdf`
244 pages, ~38.8 MB
