# Omnichannel Inventory & Fulfilment Optimization
## Business Understanding Document
### Baseline Iteration

---

## 1. Project Overview

Modern retailers operate across multiple channels simultaneously — serving customers in physical stores and online, often from the same inventory pool distributed across multiple locations. These locations may be warehouses, fulfilment centres, stores, or a combination of all three.

This project builds a two-stage stochastic optimization model that jointly optimizes inventory positioning and fulfilment routing across an omnichannel retail network, under demand uncertainty. The model is grounded in and extends established methodologies from supply chain operations research, specifically Abouelrous et al. (2022) and Presbitero et al. (2025), and is informed by the practical benchmarking context of the VN2 Inventory Planning Challenge (Vandeput, 2025).

---

## 2. Business Problem

Every omnichannel retailer faces two decisions that must be made together:

**Decision 1 — Inventory Positioning:** How much inventory to stock at each location, when to replenish, how much to order, and when to stop replenishing as the planning horizon approaches its end. Stock too much and capital is tied up in holding costs. Stock too little and sales are lost, or online orders are fulfilled from distant locations at high transportation cost.

**Decision 2 — Fulfilment Routing:** When an online order arrives, which location should fulfil it? Any location in the network could potentially serve the order. Choosing the wrong one — whether because it is too far away, or because fulfilling from it will deplete stock needed elsewhere — is costly both financially and operationally.

These two decisions are not independent. The right fulfilment strategy depends on how inventory is distributed across the network. The right inventory distribution depends on the fulfilment strategy you plan to use. Optimizing one without the other leaves significant value on the table.

---

## 3. Why This Is Hard

**Demand is uncertain.** The retailer must commit to inventory levels before knowing exactly how much demand will arrive, how it will split between online and in-store customers, and at which locations.

**Inventory is shared across channels.** The same stock that serves a walk-in customer also serves an online customer. Decisions made for one channel directly constrain the other.

**Fulfilment costs vary by distance.** Shipping an online order from a nearby location is cheap. Shipping it from across the network is expensive. The distribution of inventory across locations therefore has a direct impact on fulfilment costs.

**Holding and fulfilment costs trade off against each other.** Stocking more inventory at each location reduces the need to ship from distant locations but increases holding costs. Finding the right balance requires optimizing both simultaneously.

**Policies that look good on average can fail badly under stress.** Most inventory models are validated under controlled or favourable conditions. The real test of a policy is how it holds up when demand spikes unexpectedly, the online and in-store demand mix shifts, or lead times extend.

**Scale.** A retailer operating tens or hundreds of locations, across thousands of products, cannot make these decisions manually or through intuition alone.

---

## 4. Strategic Goal

Reduce total cost to serve, minimize lost sales, and improve stock availability across the network by making better joint inventory positioning and fulfilment routing decisions under demand uncertainty.

---

## 5. Technical Goal

Achieve this by building a two-stage stochastic optimization pipeline that integrates probabilistic demand forecasting, scenario clustering, joint inventory and replenishment optimization, dynamic fulfilment routing, and stress testing into a single end-to-end system.

---

## 6. Stakeholders

| Stakeholder | Interest |
|---|---|
| **Inventory Planners** | Actionable replenishment schedules they can execute operationally |
| **Fulfilment Operations** | Clear routing rules for online orders period by period |
| **Finance / CFO** | Cost reduction, margin improvement, and capital efficiency |
| **Supply Chain Leadership** | Network-wide visibility into cost-service trade-offs |
| **IT / Engineering** | Deployable and maintainable system |
| **Merchants / Partners** | Stock availability and demand fill rate performance |

---

## 7. What the Model Decides

### Inventory Decisions
- How much stock to place at each location at the start of the planning horizon
- When to trigger replenishment orders during the period
- How much to order at each replenishment point
- When to stop replenishing as the horizon approaches its end

### Fulfilment Decisions
- Period by period, which location should fulfil each online order
- How much inventory each location should hold in reserve for its own in-store customers before committing stock to online fulfilment

---

## 8. What the Model Optimizes

The model minimizes total cost to serve across the entire network and the full planning horizon. Cost components include:

| Cost Component | Description |
|---|---|
| **Holding costs** | Cost of carrying inventory at each location over time |
| **Inbound costs** | Cost of receiving replenishment orders into the network |
| **Outbound costs** | Cost of processing sales orders |
| **Transportation costs** | Cost of fulfilling online orders, increasing with distance |
| **Returns processing costs** | Cost of handling returned items |
| **Lost sales costs** | Revenue and margin foregone when demand cannot be met |

Maximizing profit contribution and improving stock availability are direct outcomes of minimizing these costs.

---

## 9. Constraints

The model respects the following real-world boundaries:

- Inventory used to fulfil demand at any location cannot exceed the stock available at that location
- Online orders fulfilled from a location cannot exceed the online demand for that order
- Inventory levels are non-negative at all times and at all locations
- Replenishment quantities must meet minimum order requirements
- The review period — how frequently replenishment decisions are evaluated — is fixed based on operational constraints

---

## 10. Business KPIs

These are the metrics the business tracks regardless of what model produces them. They represent what actually matters to the business.

| Business KPI | Definition | Direction |
|---|---|---|
| **Gross Merchandise Value (GMV)** | Total value of merchandise sold | Maximize |
| **GMV after Fulfilment Costs** | GMV minus operational costs — the true margin | Maximize |
| **Stock Availability Rate** | Proportion of time a product is in stock and available for sale, weighted by revenue potential | Maximize |
| **Demand Fill Rate** | Proportion of customer demand fulfilled from available stock, weighted by revenue potential | Maximize |
| **Lost Sales** | Revenue and margin foregone when demand cannot be met | Minimize |
| **Total Cost to Serve** | Sum of all cost components across the network and planning horizon | Minimize |
| **Inventory Turnover** | How efficiently capital tied up in stock is converted to sales | Maximize |

---

## 11. Technical Metrics

These are the metrics that tell us whether the model is working well, distinct from whether the business is performing well.

| Technical Metric | Definition | Direction |
|---|---|---|
| **Cost Reduction vs Baseline** | Percentage cost improvement over benchmark policy | Maximize |
| **Scenario Coverage** | Proportion of demand scenarios adequately represented after clustering | Maximize |
| **Policy Robustness Score** | Performance degradation under stress test conditions | Minimize |
| **WAPE** | Weighted Absolute Percentage Error of demand forecast at SKU-location level | Minimize |
| **Service Level** | Probability that demand is met without stockout at each location | Maximize |
| **Computational Time** | Time to solve the optimization per planning cycle | Minimize |
| **Optimality Gap** | Difference between the solution found and the theoretical optimum | Minimize |

---

## 12. What Distinguishes This Project

Existing approaches in the literature address pieces of this problem in isolation:

- **Abouelrous et al. (2022)** addresses multi-location inventory positioning and dynamic fulfilment routing in an omnichannel network but assumes a simplified demand model, a one-time pre-season inventory buy with no ongoing replenishment, and validates exclusively on synthetically generated data
- **Presbitero et al. (2025)** addresses replenishment optimization with probabilistic demand forecasting at scale using real data from Zalando, but focuses on a single-location policy without multi-location fulfilment routing
- **VN2 Inventory Planning Challenge (Vandeput, 2025)** provides a practical benchmarking context with real grocery retail data, but scores purely on holding and shortage costs without transportation, returns, or revenue-side metrics, and involves no network-level routing decisions

This project integrates both streams into a single unified model and extends them in two important directions that neither addresses:

**Ongoing replenishment.** Unlike Abouelrous et al. (2022), which assumes a one-time pre-season inventory buy, this model supports ongoing replenishment decisions throughout the planning horizon, drawing on the extended (R, s, Q) policy framework from Presbitero et al. (2025).

**Scenario-based stress testing.** Once the optimal policy is found, it is deliberately subjected to adverse demand conditions — unexpected demand spikes, shifts in the online and in-store demand mix, lead time extensions, and seasonal volatility. This quantifies the cost-service trade-offs and establishes how robust the policy is before it is deployed. A policy that performs well on average but collapses under stress is not a policy a business can rely on.

---

## 13. Expected Outputs

For each location in the network, the model produces:

- A **recommended initial inventory level** for the start of the planning horizon
- A **replenishment schedule** — when to order, how much, and a recommended cutoff date
- A **dynamic fulfilment policy** — decision rules for routing online orders across the network each period as demand is realized
- A **stress test report** — quantifying how the policy performs under a range of adverse demand scenarios, with explicit cost-service trade-off curves

---

## 14. References

1. Abouelrous, A., Gabor, A.F., Zhang, Y. (2022). Optimizing the inventory and fulfillment of an omnichannel retailer: a stochastic approach with scenario clustering. *Computers & Industrial Engineering*, 173, 108723.

2. Presbitero, A., Syrén, A., Dippel, H. et al. (2025). A practical approach to replenishment optimization with extended (R, s, Q) policy and probabilistic models. *Scientific Reports*, 15, 44225.

3. Vandeput, N. (2025). VN2 Inventory Planning Challenge. DataSource.ai.

---

*Document version: Baseline | Status: Draft | Repository: GitHub*
