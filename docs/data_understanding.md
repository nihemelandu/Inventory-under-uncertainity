# Omnichannel Inventory & Fulfilment Optimization
## Data Understanding Document
### Iteration 1

---

## 1. The Problem We Are Solving

As an omnichannel retailer operating multiple store locations, we face two tightly linked decisions that must be optimized jointly:

**Inventory Positioning:** How much inventory to stock at each location before the selling season begins. Stock too much and capital is tied up in holding costs. Stock too little and sales are lost, or online orders are fulfilled from distant locations at high transportation cost.

**Fulfilment Routing:** When an online order arrives, which location should fulfil it? Any location in the network could potentially serve the order. Choosing the wrong one — whether because it is too far away, or because fulfilling from it will deplete stock needed elsewhere — is costly both financially and operationally.

These decisions cannot be optimized independently. Inventory distribution affects fulfilment choices, and fulfilment strategy influences how inventory should be positioned. Optimizing them separately leaves significant value on the table.

The overall objective is to reduce total cost to serve, minimize lost sales, and improve stock availability across the network.

---

## 2. Data Required to Solve the Problem

The data requirements are derived directly from the mathematical structure of the two-stage stochastic optimization model, using Abouelrous et al. (2022) as the anchor reference.

### 2.1 Demand Data

The most critical input to the model. Without demand data, no scenarios can be generated and the optimization cannot run.

| Variable | Description |
|---|---|
| `d_s_it` | In-store sales per SKU per day at each store location i |
| `d_o_jt` | Online orders per SKU per region j per day t |

Both must be available at the SKU-store-day level, with sufficient historical depth to fit demand distributions and generate realistic scenarios.

### 2.2 Cost Parameters

These parameters define the objective function. The model cannot evaluate the cost of any inventory or fulfilment decision without them.

| Parameter | Description |
|---|---|
| `h` | Holding cost per unit per day |
| `p_s` | In-store penalty cost per unit of unmet demand |
| `p_o` | Online penalty cost per unit of unmet demand |
| `s` | Local fulfilment cost per unit |
| `s_ij` | Cross-location fulfilment cost per unit from store i to region j |

### 2.3 Network Data

Defines the physical structure of the retail network and enables the distance-based fulfilment cost calculation.

| Data | Description |
|---|---|
| Store locations | Geographic coordinates of each store location |
| Distance matrix | Pairwise distances between all store locations — used to compute `s_ij` |

### 2.4 Planning Horizon Parameters

| Parameter | Description |
|---|---|
| `T` | Number of days in the planning horizon — the length of the selling season |

---

## 3. What Data Is Available

### 3.1 The M5 Dataset

The primary real data source for this project is the M5 Forecasting Dataset, made available through the M5 Accuracy Competition (Kaggle, 2020).

**What it contains:**

The M5 dataset consists of three files.

---

**`sales_train_validation.csv`**

- Shape: 30,490 rows x 1,918 columns
- Each row is one SKU-store combination
- Columns 1 to 5 are identifiers: `item_id`, `dept_id`, `cat_id`, `store_id`, `state_id`
- Columns 6 to 1,918 are daily unit sales: `d_1` through `d_1913` — covering 1,913 days of history
- Three product categories: FOODS, HOBBIES, HOUSEHOLD
- Ten store locations across three US states: CA_1, CA_2, CA_3, CA_4, TX_1, TX_2, TX_3, WI_1, WI_2, WI_3
- Sales data is in wide format — each day is a column

---

**`calendar.csv`**

- Shape: 1,969 rows x 13 columns
- One row per day from 2011-01-29 to 2016-06-19
- Columns: `date`, `wm_yr_wk`, `weekday`, `wday`, `month`, `year`, `event_name_1`, `event_type_1`, `event_name_2`, `event_type_2`, `snap_CA`, `snap_TX`, `snap_WI`
- The day column in sales (`d_1`, `d_2`, ...) maps to rows in calendar by position
- SNAP flags indicate government food assistance programme days — relevant for Food category demand

---

**`sell_prices.csv`**

- Shape: 6,841,121 rows x 4 columns
- One row per store-item-week combination
- Columns: `store_id`, `item_id`, `wm_yr_wk`, `sell_price`
- Joins to sales via `store_id` and `item_id`, and to calendar via `wm_yr_wk`

---

**Scope selected for this project:**

| Parameter | Value |
|---|---|
| Category | FOODS |
| Department | FOODS_1 |
| SKU | FOODS_1_085 |
| Stores | CA_1, TX_1, WI_1 |
| SKU-store combinations | 3 |
| Planning horizon T | 3 days |
| Training window | 2011-01-29 to 2015-10-31 |
| Planning window | 2015-11-02 to 2015-11-04 |

FOODS_1_085 was selected for its lowest zero-sale rate (5.7%), stable demand (CV = 0.761), and consistent presence across all three stores. One store per state was selected to ensure geographic spread and meaningful transportation costs in the synthetic network.

**Why it is suitable:**

The M5 dataset provides real demand history that captures the full complexity of retail demand — seasonality, promotional effects, intermittent demand, and cross-SKU variation. This is the foundation for fitting per-SKU demand distributions and generating realistic demand scenarios for the stochastic optimization.

### 3.2 What M5 Does Not Provide

The M5 dataset is a single-channel in-store sales dataset. It does not contain:

- Online demand — there is no channel split between in-store and online sales
- Store geographic coordinates — only state-level identifiers are provided
- Any cost parameters — no holding, penalty, or fulfilment cost information

---

## 4. Data Gaps and How They Are Addressed

Data gaps are addressed through principled synthetic augmentation, directly grounded in the experimental design of Abouelrous et al. (2022). Every synthetic component is explicitly mapped to the real data need it substitutes and the published source that calibrates it.

| Data Required | Available | Gap | Synthetic Substitute | Source |
|---|---|---|---|---|
| In-store demand `d_s_it` | M5 daily sales | None | — | M5 Dataset |
| Online demand `d_o_jt` | Not in M5 | Full gap | Split M5 total sales using `pi_on` in {0.3, 0.5, 0.7} | Abouelrous et al. (2022) |
| Store locations | State only | Full gap | Place 3 stores uniformly in a 2D bounding box | Abouelrous et al. (2022) |
| Distance matrix | Not available | Full gap | Compute Euclidean distance from synthetic store coordinates | Abouelrous et al. (2022) |
| Holding cost `h` | Not available | Full gap | `h` in {1, 2} per unit per day | Abouelrous et al. (2022) |
| Penalty costs `p_s` and `p_o` | Not available | Full gap | `p_s` = `p_o` in {50, 100} per unit | Abouelrous et al. (2022) |
| Local fulfilment cost `s` | Not available | Full gap | `s` = 9.182 per unit | Abouelrous et al. (2022) |
| Cross-location cost `s_ij` | Not available | Full gap | See Section 4.2 | Abouelrous et al. (2022) |

### 4.1 The Online Demand Split

The most consequential synthetic assumption is the channel split `pi_on` — the proportion of total demand that arrives as online orders versus in-store customers. This parameter directly affects the model's fulfilment routing decisions and inventory positioning strategy.

Abouelrous et al. (2022) demonstrate that model performance is highly sensitive to `pi_on` — the largest cost improvements over benchmark policies occur when the proportion of in-store customers is relatively large. This sensitivity makes `pi_on` a key parameter to vary in the experimental design.

The split is applied as follows:

```
in-store demand  =  (1 - pi_on)  *  total daily sales
online demand    =       pi_on   *  total daily sales
```

### 4.2 Store Locations and Fulfilment Costs

The M5 dataset identifies stores only by state. To construct the cross-location fulfilment cost structure required by the model, store locations are synthetically placed using a uniform distribution within a 2D bounding box of side length 50, following the experimental design of Abouelrous et al. (2022).

Euclidean distances between locations are computed and used to derive the fulfilment cost matrix:

```
s_ij  =  0.75 * distance(i, j)  +  s_local      (cross-location)
s_ii  =  s_local                                 (same region)
```

This ensures fulfilment costs increase realistically with distance and that local fulfilment is always cheaper than cross-location fulfilment.

### 4.3 Cost Parameters

All cost parameters are calibrated to the experimental ranges used in Abouelrous et al. (2022). This serves two purposes: it grounds the synthetic assumptions in a published and peer-reviewed experimental design, and it enables direct benchmarking of results against that paper's findings.

---

## 5. Is the Available Data Sufficient?

Yes — with augmentation. The M5 dataset provides the most critical input to the model: real demand history that captures the full complexity of retail demand across SKUs, stores, and time. The demand data drives scenario generation, and scenario quality determines the quality of the inventory and fulfilment decisions the model produces.

Everything else that is missing — cost parameters, store locations, and the online demand split — is synthesized in a principled and transparent way, directly grounded in Abouelrous et al. (2022). The synthetic components are not arbitrary assumptions. They are calibrated to published experimental designs, making the augmentation both defensible and reproducible.

This approach is consistent with standard practice in omnichannel optimization research, where real transactional demand data is routinely combined with synthetic network and cost structures when proprietary operational data is unavailable.

---

## 6. Data Augmentation Pipeline

The following pipeline summarizes how the M5 data and synthetic components are combined to produce the inputs required by the two-stage stochastic optimization model.

```
M5 Raw Data
    sales_train_validation.csv  —  daily unit sales per SKU-store
    calendar.csv                —  date, week, event, and SNAP flags
    sell_prices.csv             —  weekly sell prices per SKU-store
    |
    |-- Step 1: Load and validate
    |           Filter to FOODS_1_085, stores CA_1, TX_1, WI_1
    |           Check for missing values
    |           Flag stockout periods
    |
    |-- Step 2: Prepare daily demand
    |           Melt wide format to long format
    |           One row per SKU-store-day
    |           Attach calendar features: date, weekday, snap, events
    |           Label each row as training or planning split
    |
    |-- Step 3: Fit demand distributions
    |           Fit Negative Binomial per store
    |           Fitted from training window daily sales
    |           Captures overdispersion in retail demand
    |           Output: fitted parameters (r, p) per store
    |
    |-- Step 4: Synthesize omnichannel structure
    |           Apply pi_on to split demand into in-store and online
    |           Place 3 stores uniformly in 2D bounding box
    |           Compute Euclidean distance matrix
    |           Compute fulfilment cost matrix s_ij
    |           Set cost parameters h, p_s, p_o, s from literature
    |
    |-- Step 5: Generate demand scenarios
    |           Sample N=500 scenarios from fitted Negative Binomial
    |           Each scenario omega is a complete realization of
    |           (d_s_it, d_o_jt) across all stores and T days
    |           Output: scenario array shape (N, n_stores, T, 2)
    |
    |-- Step 6: Apply OC scenario clustering
    |           Solve FINV LP for each scenario under x_est
    |           Compute OC similarity S_OC between scenarios
    |           Cluster using Good-Turing stopping criterion
    |           Assign probabilities P(omega) to each cluster center
    |           Output: representative scenario set Omega_bar
    |                   K << N scenarios with probabilities
    |                   sum of P(omega) = 1
```

---

## 7. References

1. Abouelrous, A., Gabor, A.F., Zhang, Y. (2022). Optimizing the inventory and fulfillment of an omnichannel retailer: a stochastic approach with scenario clustering. *Computers & Industrial Engineering*, 173, 108723.

2. Makridakis, S., Spiliotis, E., Assimakopoulos, V. (2022). M5 accuracy competition: Results, findings, and conclusions. *International Journal of Forecasting*, 38(4), 1346-1364.

---

*Document version: Iteration 1 | Status: Draft | Repository: GitHub*
