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

## 6. Solver Settings

| Parameter | Value | Purpose |
|-----------|-------|---------|
| TimeLimit | 60s | Prevent stalling on complex MIPs |
| MIPGap | 1% | Tolerance for near-optimal solutions |
| OutputFlag | 0 | Suppress solver logs |
| Threads | 1 | Reproducibility |
| Seed | 0 | Reproducibility |
