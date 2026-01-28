## Problem Statement

Inventory replenishment decisions in modern supply chains—particularly in large-scale e-commerce environments—must balance two competing objectives: minimizing inventory-related costs (holding, ordering, and shortage costs) while maintaining high product availability. This task is complicated by significant uncertainty in customer demand, seasonality, lead times, and returns.

Existing replenishment optimization approaches often rely on deterministic demand assumptions or point forecasts, even when probabilistic demand forecasts are available. As a result, there is a disconnect between forecasting models, which can capture uncertainty through demand distributions, and optimization models, which frequently ignore that uncertainty during decision making. This mismatch can lead to suboptimal replenishment policies, excess inventory, or stock-outs in practice.

Additionally, classical inventory control policies such as (R, s, Q) are well understood theoretically but are difficult to apply effectively in real-world settings due to nonlinear cost structures, nonconvex decision spaces, and the computational challenges of incorporating probabilistic demand at scale.

This project addresses the problem of integrating probabilistic demand forecasting directly into replenishment optimization by extending the classical (R, s, Q) policy framework and solving the resulting nonlinear, nonconvex optimization problem in a practical and computationally tractable way. The goal is to enable replenishment decisions that more accurately reflect demand uncertainty and achieve a better trade-off between inventory cost efficiency and service level performance.

## Problem Structure Summary

**Model**
- Nonlinear, nonconvex inventory optimization problem

**Objective / Trade-off**
- Cost minimization vs. service level
- Under stochastic (uncertain) demand

**Policy Framework**
- Extended (R, s, Q) replenishment policy

**Demand Modeling**
- Probabilistic demand inputs
- Generated using LightGBM with Conformal Prediction

**Optimization Algorithm**
- SHGO (Simplicial Homology Global Optimization)

**Algorithm Characteristics**
- Numerical optimization method
- Handles black-box, nonconvex, constrained problems
- Plays an analogous role to the simplex method in linear programming

## Attribution

The problem formulation and methodological foundations implemented in this project are based on the research presented in:

> Presbitero, A., et al. (2025). *A practical approach to replenishment optimization with extended (R, s, Q) policy and probabilistic models.*

This repository applies and operationalizes the concepts and results from the above work. Any extensions, implementation details, or adaptations beyond the original publication are the responsibility of the authors of this repository.
