# Probabilistic Demand Forecasting & Inventory Decision-Making under Uncertainty

## Overview
This project explores inventory decision-making under demand uncertainty using probabilistic forecasting, Monte Carlo simulation, and scenario analysis.  
Rather than relying on point forecasts, the project explicitly models forecast uncertainty and evaluates how inventory policies perform across multiple plausible demand scenarios.

The objective is decision support, not end-to-end supply chain optimization.

---

## Problem Statement
Retail and hybrid supply chains face volatile and non-stationary demand.  
While point forecasts are useful, they are almost always wrong — yet inventory decisions must still be made in advance.

This project addresses:
- How to represent demand uncertainty using probabilistic forecasts
- How to translate forecast uncertainty into scenario-based simulations
- How to evaluate inventory policies based on risk, service levels, and cost trade-offs

---

## Project Scope

### In Scope
- Demand uncertainty modeling
- Probabilistic demand forecasting (prediction intervals, quantiles)
- Monte Carlo simulation
- Scenario-based inventory evaluation
- Visualization and dashboards for decision support

### Out of Scope (Future Work)
- Supply uncertainty (lead-time variability, supplier reliability)
- Operational uncertainty (capacity, labor constraints)
- Multi-echelon or network-wide optimization
- Real-time or rolling re-optimization

---

## Data
- Historical SKU-level demand data
- Multiple SKUs and channels (e.g., online vs in-store demand signals)
- Aggregated to appropriate planning horizons (daily or weekly)

Note: Synthetic or anonymized data is used if proprietary data is unavailable.

---

## Methodology

### 1. Probabilistic Demand Forecasting
- Built forecasting models that generate:
  - Point forecasts
  - Prediction intervals
  - Quantile forecasts
- Emphasis on capturing uncertainty, not just minimizing error metrics
- Forecasts are treated as inputs to decisions, not final outputs

---

### 2. Scenario Modeling
Defined a small set of canonical demand scenarios:
- Baseline
- Optimistic (high demand)
- Pessimistic (low demand)
- Disruption (demand spike or structural shift)

Scenarios are derived from probabilistic forecast distributions.

---

### 3. Monte Carlo Simulation
- Sampled demand realizations from probabilistic forecasts
- Ran Monte Carlo simulations to evaluate inventory policies under uncertainty
- Simulated outcomes include:
  - Stockouts
  - Service level / fill rate
  - Inventory holding cost

This allows evaluation of policy robustness, not just expected outcomes.

---

### 4. Inventory Policy Evaluation
Evaluated inventory decisions such as:
- Reorder points
- Safety stock levels
- Target inventory thresholds

Focused on:
- Trade-offs between service level and holding cost
- Risk exposure under adverse demand scenarios
- Sensitivity to forecast uncertainty

No claim of global optimality — emphasis is on decision insight.

---

### 5. Visualization & Dashboards
Built interactive dashboards to:
- Compare outcomes across scenarios
- Visualize distributions (not just averages)
- Highlight stockout risk vs inventory investment
- Support data-driven replenishment decisions

---

## Key Metrics
- Service level / fill rate
- Stockout probability
- Inventory holding cost
- Variability of outcomes across scenarios

---

## Tools & Technologies
- Python (NumPy, pandas, SciPy)
- Forecasting libraries (statsmodels, ML models)
- Monte Carlo simulation
- Data visualization (Plotly, Dash, Matplotlib)
- Jupyter notebooks

---

## Key Takeaways
- Probabilistic forecasts are more informative than point forecasts for inventory decisions
- Scenario-based simulation reveals risks hidden by average-case planning
- Inventory decisions should be evaluated on robustness, not forecast accuracy alone
- Forecasting is a means to an end, not the end itself

---

## Future Work
- Incorporate supply-side uncertainty (lead time variability)
- Add operational constraints and capacity limits
- Extend to multi-location inventory allocation
- Explore decision-aware or cost-aware forecasting objectives

---

## Project Structure


### References
1. Smith, J., & Lee, A. (2023). Hybrid ML Forecasting & Inventory Optimization in a Multi-Channel Supply Chain. *International Journal of Engineering and Creative Research (IJECR)*.  
2. Zhang, Y., & Kumar, R. (2021). Two-Stage Stochastic Optimization for Inventory Management under Demand Uncertainty. *European Journal of Operational Research (EJOR)*.
