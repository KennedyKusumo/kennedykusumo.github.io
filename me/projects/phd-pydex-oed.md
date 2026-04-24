# PhD Research — Algorithms and Tools for Feasibility Analysis and Optimal Experiment Design

**Institution:** Imperial College London, Department of Chemical Engineering
**Duration:** January 2018 – January 2022
**Supervisors:** Prof. Nilay Shah · Prof. Benoît Chachuat (Imperial) · Dr. Federico Galvanin (UCL) · Prof. Salvador García-Muñoz (Eli Lilly, external)
**Industrial collaborator:** Eli Lilly and Company

---

## Research Summary

Kennedy's PhD focused on **model-based design of experiments (MBDOE)** — the science of designing informative experiments to accelerate and reduce the cost of developing mathematical models of chemical and pharmaceutical processes.

Two core deliverables:
1. **Novel MBDOE algorithms** — addressing fundamental limitations in the state-of-the-art
2. **Software tools** — open-source Python packages implementing existing and novel techniques

Primary output: **Pydex** (single-objective MBDOE-PE) and **DEUS** (feasibility analysis via nested sampling), both open-source Python packages used in pharmaceutical process R&D.

---

## Background & Motivation

Mathematical models are central to process systems engineering (PSE). Their development requires experimentation, and experimentation is expensive — especially in pharmaceutical R&D where materials are scarce, safety constraints are tight, and regulatory requirements are strict.

**Two types of uncertainty** exist in any model:
- **Structural uncertainty**: we cannot prove our proposed model structure is correct
- **Parametric uncertainty**: parameters are estimated from noisy experimental data → uncertain parameter estimates

**MBDOE** uses mathematical models to ask: *"Given what we know, what experiment would tell us the most?"* Two sub-fields:
- **MBDOE-MD** (model discrimination): choose among competing model structures
- **MBDOE-PE** (parameter estimation): reduce uncertainty in parameter estimates — Kennedy's primary focus

### Why this matters in pharma
The **Design Space (DS)** concept from ICH Q8 (Quality-by-Design framework): the multidimensional combination of input variables and process parameters that provide assurance of product quality. DS characterization requires well-parameterized mechanistic models. Poor experiment design → high parametric uncertainty → uncertain DS → regulatory risk.

---

## Key Literature Survey Findings (ESA Report, Dec 2018)

### Standard MBDOE-PE Framework

The **Fisher Information Matrix (FIM)**, M(θ, φ), is central:

```
M(θ, φ) = Σ (1/σ²_oo') · S_o^T · S_o' + M₀
```

where S_o is the sensitivity matrix (partial derivatives of model outputs w.r.t. parameters).

Standard design criteria (all maximised):
- **D-optimal**: max det(M) → minimises volume of parameter confidence ellipsoid
- **A-optimal**: max tr(M) → minimises sum of parameter variances
- **E-optimal**: max λ_min(M) → minimises longest axis of confidence ellipsoid

### Critical Limitation Discovered (Novel Finding)

For **non-linear models**, the FIM is θ-dependent — designs are locally optimal (around current parameter estimates) but not globally robust.

More importantly, Kennedy discovered a fundamental limitation: **D-optimal MBDOE-PE for non-linear models does NOT guarantee exploration of the input space.** This is because the "explanatory variables" in the linearised sensitivity space are sensitivities, not input variables — the optimal design in sensitivity space may only cover a tiny fraction of the actual input space, leaving the model unvalidated over most of its intended operating range.

This is distinct from linear models, where D-optimal designs have the **spanning property** — they always cover the full input space by construction.

**Demonstrated with:** consecutive first-order reactions A → B → C in a batch reactor, comparing a linear polynomial model (which spans) vs. the non-linear mechanistic model (which does not).

This gap — designing experiments that simultaneously optimise **parameterisation** AND **exploration/validation** — became WP 1-1.

### Other Identified Challenges
1. Suboptimality for non-linear models (θ-dependence of FIM)
2. Criterion selection — which criterion to use when the goal is process optimisation vs. DS characterisation vs. control?
3. Inadequate model structures — what to do when the best mechanistic model is provably wrong in some regions?

---

## Research Themes and Work Packages

### Theme 1: Experimentation and Validation of Models

**WP 1-1: Simultaneous Optimal Parameterisation and Exploration**

Research question: *Can we design experiments that simultaneously maximise information for parameter estimation AND ensure sufficient exploration of the input space for model validation?*

Approach: **multi-objective optimisation** where parameterisation quality and input space exploration are separate, non-commensurable objectives. Result is a Pareto frontier of efficient experiment designs — letting practitioners choose their preferred trade-off.

### Theme 2: Goal-oriented Model Development

**WP 2-1: MBDOE-PE for Design Space Characterisation**

Existing goal-oriented frameworks (go-MBDOE-PE) focus on process optimisation as the end-goal. DS characterisation is different — it cannot be posed as a process optimisation problem.

Research question: *How do we incorporate DS characterisation as the explicit modelling goal into the experimental design?*

Proposed measures of DS uncertainty: uncertainty in DS volume, DS bounds, or DS centroid.

**WP 2-2: Multi-goal MBDOE-PE**

In practice, models are built for multiple purposes simultaneously (control, optimisation, DS characterisation). Research question: *What is the right framework for MBDOE-PE when there are multiple competing end-goals?*

Proposed approaches: Pareto-optimal multi-objective framework; or weighted combination of goal-specific weighting matrices.

### Theme 3: Interface Between Mechanistic and Black-box Models

**WP 3-1: Hybrid Modelling Development Framework**

Philosophy: accept that even the best mechanistic model may be inadequate (simplified physics, unknown chemistry). Rather than discarding the mechanistic model, keep developing it and characterise its domain of validity.

Proposed approach: **parallel hybrid modelling** — use a Gaussian Process (non-parametric black-box) to model the residuals of the mechanistic model. This is a tractable, interpretable hybrid.

Two model types:
- **Type I**: fixed mechanistic model, updated GP — domain of validity is fixed
- **Type II**: both models updated iteratively — more complex experiment design motivation

Framework for experiment design: **Bayesian Optimisation acquisition functions** (exploration term = GP prediction variance) to drive development of the non-parametric component.

---

## Progress at 10 Months (December 2018)

### Software: Python MBDOE-PE Implementation (P0 Tool)

Built using **Pyomo** and **Pyomo.DAE**, supporting:
- Simulation
- Parameter estimation (MLE / least squares)
- Bootstrap parameter estimation (for non-identifiable systems)
- MBDOE-PE (D-/A-/E-optimal design)

**Demo case study:** Thermal de-protection reaction in a CSTR:
- Reaction: A → P + BP (in THF solvent)
- Kinetics: first-order, Arrhenius rate law
- Key issue: FIM non-invertible at constant temperature (non-identifiable) → bootstrap method used for parameter confidence regions
- Result: A-optimal MBDOE-PE design significantly improved parameter precision vs. arbitrary experiment, even under high parameter correlation

### Novel Finding: Non-linear Validation Gap

See "Critical Limitation Discovered" above. This was Kennedy's first original research contribution — establishing a gap in the existing literature and motivating WP 1-1.

---

## Key Software Outputs

| Tool | Description | Language |
|------|-------------|----------|
| **Pydex** | Single-objective MBDOE-PE | Python (Pyomo) |
| **DEUS** | Feasibility analysis via nested sampling | Python |

Both released open-source. Pydex was the primary deliverable developed throughout the PhD (P0 → P1 evolution).

---

## Publications (from PhD, 7 total)

(Placeholder — to be filled when publication list is shared)

---

## Connections to Other Work

- **MSc Thesis (Syngenta):** Used AIMMS for MILP optimisation — different domain, but same PSE foundations
- **Eli Lilly Internship (2020):** Applied MBDOE-PE knowledge to the Mounjaro synthesis reactor — direct application of PhD methods in industrial setting
- **Imperial Research Associate (2022–2023):** Extended to multi-objective OED (PharmaSEL-Prosperity Partnership with Eli Lilly)
- **Sheffield Research Associate (2022–2023):** Applied process modelling + OED to mRNA vaccine manufacturing

---

## Source Files

- `D:\OneDrive\Imperial OneDrive Copy\OneDrive - Imperial College London\PhD\ESA\December\ESA Report.docx` — PhD Confirmation Report (Dec 2018), ~10 months in
- `D:\OneDrive\Imperial OneDrive Copy\OneDrive - Imperial College London\PhD\ESA\October\ESA Report.docx` — Earlier ESA report (Oct 2018)
- PhD thesis: pending extraction
