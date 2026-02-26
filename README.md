# Omnichannel Inventory & Fulfilment Optimization

## Business Context

Modern retailers no longer operate through a single channel. Today's omnichannel retailer serves customers both in physical stores and online — often from the same pool of inventory spread across multiple locations. These locations may be warehouses, fulfilment centres, stores, or a combination of all three.

This creates an environment where inventory and distribution decisions are deeply interconnected, and where getting them wrong has a direct and measurable impact on profitability, operating costs, and customer satisfaction.

---

## The Problem

Every omnichannel retailer faces two decisions that must be made together:

**1. How much inventory to stock at each location — and when?**
Stock too much and capital is tied up in holding costs. Stock too little and sales are lost, or orders are fulfilled from distant locations at high transportation cost.

**2. When an online order arrives, which location should fulfil it?**
Any location in the network could potentially serve the order. Choosing the wrong one — whether because it is too far away, or because fulfilling from it will deplete stock needed elsewhere — is costly both financially and operationally.

These two decisions are not independent. The right fulfilment strategy depends on how inventory is distributed across the network. The right inventory distribution depends on the fulfilment strategy you plan to use. Optimizing one without the other leaves significant value on the table.

---

## Why This Is Hard

Several factors make this problem genuinely difficult:

**Demand is uncertain.** The retailer must commit to inventory levels before knowing exactly how much demand will arrive, how it will split between online and in-store customers, and at which locations. Getting this wrong in either direction is costly.

**Inventory is shared across channels.** The same stock that serves a walk-in customer also serves an online customer. Decisions made for one channel directly constrain the other.

**Fulfilment costs vary by distance.** Shipping an online order from a nearby location is cheap. Shipping it from across the network is expensive. The distribution of inventory across locations therefore has a direct impact on fulfilment costs.

**Holding and fulfilment costs trade off against each other.** Stocking more inventory at each location reduces the need to ship from distant locations, but increases holding costs. Finding the right balance requires optimizing both simultaneously.

**Scale.** A retailer operating tens or hundreds of locations, across thousands of products, cannot make these decisions manually or through intuition alone.

---

## The Approach

This project builds a **two-stage stochastic optimization model** that makes both the inventory positioning and fulfilment routing decisions jointly, under demand uncertainty. The modelling framework is grounded in and extends established methodologies from supply chain operations research.<sup>1,2</sup>

The word *stochastic* simply means the model explicitly accounts for the fact that demand is unknown in advance. Rather than assuming a single forecast is correct, the model considers a range of plausible demand scenarios and finds the inventory and fulfilment policy that performs best across all of them.

The two stages reflect the natural timing of decisions in the real world:

- **Before the planning horizon begins** — decide how much inventory to place at each location, and when and how much to replenish as the period unfolds
- **As demand arrives** — decide in real time which location fulfils each online order, reserving enough stock at each location to serve future in-store customers

Computational complexity is managed through a scenario clustering technique that reduces the number of demand scenarios the model needs to evaluate without sacrificing solution quality.

---

## What The Model Decides

In plain business terms, the model produces two types of output:

**Inventory Decisions**
- How much stock to place at each location at the start of the planning horizon
- When to trigger replenishment orders during the period
- How much to order at each replenishment
- When to stop replenishing as the horizon approaches its end

**Fulfilment Decisions**
- Period by period, which location should fulfil each online order
- How much inventory each location should hold in reserve for its own in-store customers before committing stock to online fulfilment

---

## What It Optimizes

The model minimizes **total cost to serve** across the entire network and the full planning horizon. This includes:

- **Holding costs** — the cost of carrying inventory at each location over time
- **Inbound costs** — the cost of receiving replenishment orders into the network
- **Outbound costs** — the cost of processing sales orders
- **Transportation costs** — the cost of fulfilling online orders, which increases with distance between the fulfilling location and the customer
- **Returns processing costs** — the cost of handling returned items
- **Lost sales costs** — the revenue and margin foregone when demand cannot be met

Maximizing profit contribution and improving stock availability are direct outcomes of minimizing these costs.

---

## The Constraints

The model respects the following real-world boundaries:

- Inventory used to fulfil demand at any location cannot exceed the stock available at that location
- Online orders fulfilled from a location cannot exceed the online demand for that order
- Inventory levels are non-negative at all times and at all locations
- Replenishment quantities must meet minimum order requirements
- The review period — how frequently replenishment decisions are evaluated — is fixed based on operational constraints

---

## What You Get Out

For each location in the network, the model produces:

- A recommended initial inventory level for the start of the planning horizon
- A replenishment schedule — when to order, how much, and a recommended cutoff date
- A dynamic fulfilment policy — decision rules for routing online orders across the network each period as demand is realized

These outputs are designed to be actionable at the operational level while being directly linked to the financial and service level objectives that matter to the business.

---

## References

<sup>1</sup> Abouelrous, A., Gabor, A.F., Zhang, Y. (2022). Optimizing the inventory and fulfillment of an omnichannel retailer: a stochastic approach with scenario clustering. *Computers & Industrial Engineering*, 173, 108723.

<sup>2</sup> Presbitero, A. et al. (2025). A practical approach to replenishment optimization with extended (R, s, Q) policy and probabilistic models. *Scientific Reports*, 15, 44225.
