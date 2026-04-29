# RetailOpt-190: Retail Supply Chain Benchmark

---

## 1. Overview

RetailOpt-190 is a benchmark dataset for evaluating **text-to-optimization** methods on **38 retail operations archetypes**, expanded into **190 solver-validated instances** via controlled perturbations.

### Dataset Summary

| Feature | Description |
|---------|-------------|
| **Instances** | 38 archetypes × 5 variants = 190 |
| **Ground Truth** | Universal Retail Solver (Gurobi MILP) |
| **Domain** | Retail supply chain optimization |
| **Format** | JSON data + text prompts |

All instances share a **single JSON schema** and are solved by a **single universal MILP formulation**.

### Comparison with Existing Benchmarks

| Benchmark | Scenarios | Multi-period | Domain | Compositional |
|-----------|-----------|--------------|--------|---------------|
| NL4Opt | 289 | Few | General OR | ✗ |
| MAMO | ~800 | Some | General OR | ✗ |
| IndustryOR | 100 | Some | Industrial | ✗ |
| **RetailOpt-190** | 190 | All | Supply Chain | ✓ |

---

## 2. Prompt Formats

RetailOpt-190 provides **three prompt formats** for different evaluation scenarios:

| File | Content | Data Location | Use Case |
|------|---------|---------------|----------|
| `{id}.scenario.txt` | Scenario + data schema | External (runtime) | Schema-based evaluation |
| `{id}.full.txt` | Scenario + full JSON data | In prompt | Data-embedded evaluation |
| `{id}.base.txt` | Scenario description only | External (runtime) | Agentic workflows |

### Prompt Format Roles

| Format | Field | Role | Use Case |
|--------|-------|------|----------|
| **Data-embedded** | `prompt_full` | **Default evaluation format** | Standard evaluation, comparison with other benchmarks |
| **Schema-based** | `prompt_schema` | Industrial-style format | Large datasets, production scenarios, agentic workflows |

**Data-embedded (`prompt_full`) is the default evaluation format.** ReLoop and all baseline experiments use this format to maintain a consistent input structure across models and datasets (NL4Opt, MAMO, IndustryOR all embed data in prompts). Schema-based (`prompt_schema`) separates data from the prompt and loads it at runtime, which better reflects real-world industrial workflows where data volumes make in-prompt embedding impractical.

### Schema-based Prompt Structure (`.scenario.txt`)

```
[SCENARIO]
├── Family/Archetype/Scenario ID

[BUSINESS DESCRIPTION]
├── Business narrative
└── Structure cues

[DATA SCHEMA]
├── JSON structure (types only, NOT actual data)

[DATA ACCESS]
├── How to read from `data` variable

[OUTPUT FORMAT]
├── GurobiPy, print format

[TASK]
└── Write complete GurobiPy script...
```

### Data-embedded Prompt Structure (`.full.txt`)

```
[SCENARIO]
├── Family/Archetype/Scenario ID

[BUSINESS DESCRIPTION]
├── Business narrative
└── Structure cues

[DATA]
├── Full JSON data embedded in prompt

[OUTPUT FORMAT]
├── GurobiPy, print format

[TASK]
└── Parse JSON and solve...
```

---

## 3. Evaluation Metrics

| Metric | Definition | Formula |
|--------|------------|---------|
| **Execution Rate** | Code runs without runtime errors | `exec_ok / total` |
| **Accuracy** | Status matches AND objective within tolerance | `correct / total` |

### Accuracy Criterion

An instance is **correct** if:
1. Solver status matches ground truth (both feasible, or both infeasible)
2. For feasible instances: |y_pred - y_ref| / |y_ref| < ε

| Scenarios | Problem Type | Tolerance (ε) |
|-----------|--------------|---------------|
| F1–F5, F6 (lead_time, moq_binary), F7–F8 | LP / easy MIP | 0.01% |
| F6 (pack_size_integer, fixed_order_cost) | Hard MIP, hits 60s time limit | 1% |

Only `pack_size_integer` and `fixed_order_cost` hit the 60-second time limit; the other F6 archetypes solve to optimality within seconds.

---

## 4. Scenario Families

| Family | Name | Archetypes | Key Mechanisms |
|--------|------|------------|----------------|
| F1 | Core Operations | 4 | Multi-period inventory, seasonal demand, lost sales |
| F2 | Assortment | 6 | Substitution, promotions, ultra-short shelf life |
| F3 | Resources | 4 | Storage bottleneck, production capacity coupling |
| F4 | Dynamics | 6 | Supply disruptions, demand volatility |
| F5 | Feasibility | 4 | Stress tests requiring lost-sales slack |
| F6 | Logistics | 4 | Lead time, MOQ, pack size, fixed order cost |
| F7 | Network & Multi-Echelon | 6 | Transshipment, hub-spoke, three-echelon |
| F8 | Omni-channel | 4 | Returns, labor constraints, sustainability |

**Total:** 38 archetypes × 5 variants = **190 instances**

---

## 5. JSON Schema

Each instance has this structure:

```json
{
  "name": "retail_f1_base_v0",
  "periods": 20,
  "products": ["SKU_Basic", "SKU_Premium", "SKU_ShortLife"],
  "locations": ["DC1", "DC2", "DC3", "DC4", "DC5"],

  "shelf_life": {"SKU_Basic": 10, "SKU_Premium": 8, "SKU_ShortLife": 4},
  "lead_time": {"SKU_Basic": 0, "SKU_Premium": 0, "SKU_ShortLife": 0},

  "demand_curve": {
    "SKU_Basic": [303, 311, 328, ...],
    "SKU_Premium": [151, 155, 164, ...]
  },
  "demand_share": {"DC1": 0.25, "DC2": 0.2, "DC3": 0.2, "DC4": 0.2, "DC5": 0.15},

  "production_cap": {"SKU_Basic": [800, 800, ...], ...},
  "cold_capacity": {"DC1": 4000, "DC2": 3500, ...},
  "cold_usage": {"SKU_Basic": 1.0, "SKU_Premium": 3.0, ...},

  "labor_cap": {"DC1": [99999.0, ...], ...},
  "labor_usage": {"SKU_Basic": 0.0, ...},
  "return_rate": {"SKU_Basic": 0.0, ...},

  "costs": {
    "purchasing": {"SKU_Basic": 10.0, ...},
    "inventory": {"SKU_Basic": 1.0, ...},
    "waste": {"SKU_Basic": 2.0, ...},
    "lost_sales": {"SKU_Basic": 50.0, ...},
    "fixed_order": 0.0,
    "transshipment": 0.5
  },

  "constraints": {
    "moq": 0,
    "pack_size": 1,
    "budget_per_period": null,
    "waste_limit_pct": null
  },

  "network": {
    "sub_edges": [["SKU_Basic", "SKU_Premium"]],
    "trans_edges": []
  }
}
```

### Data Access Patterns

```python
# Network data is NESTED - use safe access
sub_edges = data.get('network', {}).get('sub_edges', [])
trans_edges = data.get('network', {}).get('trans_edges', [])

# Demand calculation
demand[p, l, t] = data['demand_curve'][p][t-1] * data['demand_share'][l]

# Production capacity (0-indexed list)
prod_cap = data['production_cap'][p][t-1]
```

---

## 6. Inventory Dynamics (Perishability)

All archetypes share a common shelf-life-aware inventory accounting. Inventory is indexed by `(p, l, t, r)` where `r` is the **remaining periods of life**.

### Variable Definitions

| Symbol | Meaning |
|--------|---------|
| `Q[p, l, t]` | Per-location order quantity placed in period `t` for product `p` |
| `I[p, l, t, r]` | Inventory at the **start** of period `t` with `r` periods of life remaining |
| `sales[p, l, t, r]` | Units of product `p` sold from age-bucket `r` at location `l` in period `t` |
| `W[p, l, t]` | Waste (expired units that were not sold) |
| `L[p, l, t]` | Lost sales (unmet demand after substitution) |
| `S[p_from, p_to, l, t]` | Substitution flow: units of `p_from`'s demand fulfilled by `p_to`'s inventory |
| `X[p, l_src, l_dst, t]` | Transshipment flow from `l_src` to `l_dst` |

### Convention

- `r = 1` is the **OLDEST** age bucket (FIFO: must be sold first or wasted).
- `r = shelf_life[p]` is the **FRESHEST** bucket (newly arrived).

### Key Equations

**(1) Fresh inflow** (only the freshest age bucket receives new units):

```
I[p, l, t, SL] = Q[p, l, t - LT[p]] + transshipment_net[p, l, t] + returns[p, l, t]
```

- `Q[p, l, t]` is the **per-location** decision variable. Treat `Q[p, l, s] = 0` for `s < 1`.
- `LT[p] = lead_time[p]` (orders arrive after this many periods).
- `transshipment_net[p, l, t] = sum over (l_src, l) in trans_edges of X[p, l_src, l, t] − sum over (l, l_dst) in trans_edges of X[p, l, l_dst, t]`. Zero when `network.trans_edges` is empty.
- `returns[p, l, t] = return_rate[p] * sum over a of sales[p, l, t-1, a]` for `t > 1`, else `0`.
- This channel is **only** the fresh inflow — do **not** subtract sales here.

**Aggregate production capacity** (couples per-location orders to product capacity):

```
sum over l of Q[p, l, t]  <=  production_cap[p][t]
```

**(2) Aging** (yesterday's age-`r+1` becomes today's age-`r` after sales of the older bucket):

```
I[p, l, t+1, r] = I[p, l, t, r+1] − sales[p, l, t, r+1]      for r = 1 .. SL-1
```

**(3) Waste** (anything in the oldest bucket not sold becomes waste):

```
W[p, l, t] = I[p, l, t, 1] − sales[p, l, t, 1]
```

**(4) Sales availability** (cannot sell more than is on hand in each age bucket):

```
sales[p, l, t, r]  <=  I[p, l, t, r]
```

**(5) Holding cost** (charged on end-of-period inventory in non-expiring buckets, `r >= 2`):

```
holding_cost = sum_{p,l,t,r>=2} inventory_cost[p] * (I[p, l, t, r] − sales[p, l, t, r])
```

### Demand Fulfillment with Substitution

For substitution edge `[p_from, p_to]` (means: `p_to`'s inventory can serve `p_from`'s demand):

```
sum_r sales[p_from, l, t, r] + S[p_from, p_to, l, t] + L[p_from, l, t] = demand[p_from, l, t]
sum_r sales[p_to,   l, t, r] − S[p_from, p_to, l, t] + L[p_to,   l, t] = demand[p_to,   l, t]
```

When `sub_edges = []`, only the diagonal `total_sales[p] + L[p] = demand[p]` applies.

### Capacity & Auxiliary Constraints

| Constraint | Form |
|------------|------|
| Storage | `sum_p cold_usage[p] * total_inventory[p, l, t]  <=  cold_capacity[l]` |
| Labor | `sum_p labor_usage[p] * units_handled[p, l, t]  <=  labor_cap[l, t]` |
| Budget | `sum_{p,l} purchasing[p] * Q[p, l, t] (+ fixed_order * z[p, l, t])  <=  budget_per_period` |
| Waste cap | `sum_{p,l,t} W[p, l, t]  <=  waste_limit_pct * sum demand` |
| MOQ | `Q[p, l, t] = 0  OR  Q[p, l, t] >= moq` (binary `z`) |
| Pack size | `Q[p, l, t] = pack_size * n[p, l, t]`, `n` integer |
| Fixed order | `Q[p, l, t] > 0  =>  fixed_order` charged once via binary `z[p, l, t]` |

### Objective

Minimize total cost over the horizon:

```
min  sum (purchasing + inventory + waste + lost_sales + transshipment + fixed_order)
```

---

## 7. Solver Settings

| Parameter | Value | Purpose |
|-----------|-------|---------|
| TimeLimit | 60s | Prevent stalling on complex MIPs |
| MIPGap | 1% | Tolerance for near-optimal solutions |
| OutputFlag | 0 | Suppress solver logs |
| Threads | 1 | Reproducibility |
| Seed | 0 | Reproducibility |
