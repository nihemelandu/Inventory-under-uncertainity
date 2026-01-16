# Methods: Replicating the Probabilistic Replenishment Optimization System

## Overview

The goal of the replication is to optimize inventory replenishment under demand uncertainty using the approach described by Presbitero et al. (2025). The system is modular, separating demand modeling, lead time modeling, inventory simulation, and optimization, allowing independent validation of each component. The overall workflow follows a data-science perspective: given historical demand data, we construct probabilistic demand models, propagate uncertainty through inventory simulations, and optimize policy parameters using a global optimizer.

---

## Phase 1: Probabilistic Demand Forecasting (Weeks 1–3)

### Objective
Generate calibrated, non-parametric probabilistic forecasts for SKU-level demand to quantify uncertainty.

### Data
- Historical demand for each SKU at weekly granularity (e.g., 62 weeks of historical sales)
- Missing values addressed via imputation; stockouts treated as censored observations
- Channels and SKUs aggregated as needed

### Method
- Train **LightGBM quantile regression models** for each SKU to predict multiple quantiles of future demand
- Apply **Conformal Prediction** to calibrate prediction intervals and ensure proper coverage
- **Output**: distribution-free probabilistic demand forecasts; these can be sampled to generate realistic demand scenarios for simulation

### Validation
- Check calibration: verify predicted intervals contain the observed demand at the target coverage probability
- Inspect skewness, variance, and tail behavior of predicted distributions

---

## Phase 2: Lead Time Modeling (Week 4)

### Objective
Capture uncertainty in replenishment and return lead times.

### Method
- Fit parametric distributions (e.g., **gamma distributions**) to historical replenishment lead times and return processes
- Validate by comparing simulated lead times against historical distributions

### Output
- Sampleable lead time distributions for each SKU
- Inputs to inventory simulation to model stock arrival variability

---

## Phase 3: Discrete Event Inventory Simulation (Weeks 5–6)

### Objective
Model inventory dynamics under stochastic demand and lead times to evaluate policy performance.

### Method
Implement inventory balance equations:

```
I(t+1) = I(t) + Q_received - D(t)
```

- Include handling of stockouts, returns, and lead times
- Start with **deterministic demand** to validate simulator logic
- Introduce **stochastic demand and lead times** using sampled distributions
- Output metrics: on-hand inventory, stockouts, order quantities, service levels

### Validation
- Test simulator with manual cases of known demand and lead times
- Verify that simulation outputs match expected inventory trajectories

---

## Phase 4: Policy Parameterization and Optimization (Weeks 7–8)

### Policy Parameterization
- **Policy family**: extended (R, s, Q) with additional parameters θ = (t₀, Q₀, s, Q, t_limit)
- **Decision variables**:
  - **t₀**: initial order time
  - **Q₀**: initial order quantity
  - **s**: reorder point
  - **Q**: order quantity
  - **t_limit**: order limit

### Optimization Loop
1. **Monte Carlo simulation** evaluates each candidate policy by running 500 stochastic scenarios per SKU
2. **Objective**: minimize the 75th percentile of total cost, Q₇₅[C(θ)], to ensure robust, risk-aware decisions
3. **Optimizer**: SHGO (Simplicial Homology Global Optimization) searches the parameter space

**Loop structure:**
- Propose policy θ
- Run Monte Carlo simulation
- Evaluate Q₇₅ cost
- Update θ with SHGO until convergence

### Output
- Optimal policy parameters and next replenishment recommendations

---

## Integration Diagram

```
Historical demand data
        ↓
[1] Probabilistic demand modeling (LightGBM + Conformal Prediction)
        ↓
[2] Lead time modeling (returns + replenishment)
        ↓
        ┌─────────────────────────────────┐
        │  OPTIMIZATION LOOP              │
        │  ┌──────────────────────┐       │
        │  │ [3] Propose policy θ │       │
        │  └──────────┬───────────┘       │
        │             ↓                    │
        │  ┌──────────────────────┐       │
        │  │ [4] Monte Carlo      │       │
        │  │     Simulation       │       │
        │  │  (demand + lead      │       │
        │  │   times)             │       │
        │  └──────────┬───────────┘       │
        │             ↓                    │
        │  ┌──────────────────────┐       │
        │  │ [5] Evaluate         │       │
        │  │     Q₇₅[C(θ)]        │       │
        │  └──────────┬───────────┘       │
        │             ↓                    │
        │  ┌──────────────────────┐       │
        │  │ [6] Update θ (SHGO)  │       │
        │  └──────────┬───────────┘       │
        │             ↓                    │
        │      Converged? ──No──┐         │
        │             │         │         │
        └─────────────┼─────────┘         │
                     Yes                  │
                      ↓                   │
           Optimal policy parameters      │
                 (t₀, Q₀, s, Q, t_limit) │
```

---

## Phase 5: End-to-End Testing and Validation (Weeks 9–10)

### Steps
1. Backtest the integrated system on historical demand
2. Compare against baseline policies (e.g., naive reorder, EOQ, deterministic (R,s,Q))
3. Evaluate metrics:
   - Service level
   - Stockout frequency
   - Total cost
4. Identify robustness under demand shifts or lead time variability

### Observations for Replication
- **Training and deployment are separated**: forecasts are generated for 12-week horizons, but the system does not adapt online
- **Potential extension**: online learning or feedback loop to handle concept drift

---

## Key Takeaways from Decomposition

✅ **Separate concerns**: Forecasting, lead times, simulation, and optimization are modular

✅ **Non-parametric uncertainty modeling**: LightGBM + Conformal Prediction allows realistic demand sampling

✅ **Risk-aware policy optimization**: 75th percentile cost minimizes tail risk rather than average cost

✅ **Replication-ready**: Each module can be implemented, validated, and replaced independently

---

## Implementation Roadmap Summary

| Phase | Weeks | Component | Deliverable |
|-------|-------|-----------|-------------|
| 1 | 1-3 | Demand Forecasting | Calibrated LightGBM quantile model |
| 2 | 4 | Lead Time Models | Gamma distributions for lead times |
| 3 | 5-6 | DES Simulator | Validated inventory simulator |
| 4 | 7-8 | Optimization | SHGO-based policy optimizer |
| 5 | 9-10 | Integration & Testing | Backtested system with metrics |

---

## Critical Success Factors

1. **Data quality**: Ensure clean historical demand and lead time data
2. **Independent validation**: Test each module separately before integration
3. **Calibration checks**: Verify probabilistic forecasts are well-calibrated
4. **Baseline comparisons**: Always compare against simple policies (EOQ, base-stock)
5. **Documentation**: Keep detailed logs of design decisions and parameter choices

---

*This section provides everything a replicator needs: data requirements, modeling choices, algorithmic structure, validation strategy, and phased implementation.*
