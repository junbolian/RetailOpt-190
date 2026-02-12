# RetailOpt-190: Prompt Format Documentation

---

## 1. Prompt Architecture

RetailOpt-190 uses **three prompt formats** for different evaluation scenarios:

### Three Prompt Files Per Scenario

| File | Content | Data Location | Use Case |
|------|---------|---------------|----------|
| `{scenario_id}.scenario.txt` | Scenario + data schema | External (runtime) | Schema-based evaluation |
| `{scenario_id}.full.txt` | Scenario + full JSON data | In prompt | Data-embedded evaluation |
| `{scenario_id}.base.txt` | Scenario description only | External (runtime) | Agentic workflows |

### Prompt Format Roles

| Format | Role | Use Case |
|--------|------|----------|
| **Data-embedded** (`.full.txt`) | **Default evaluation format** | Standard evaluation, comparison with other benchmarks |
| **Schema-based** (`.scenario.txt`) | Industrial-style format | Large datasets, production scenarios, agentic workflows |
| **Agent base** (`.base.txt`) | Minimal prompt | Multi-step agents that inject their own guardrails |

**Data-embedded (`prompt_full`) is the default evaluation format.** ReLoop and all baseline experiments use this format to maintain a consistent input structure across models and datasets (NL4Opt, MAMO, IndustryOR all embed data in prompts). Schema-based (`prompt_schema`) separates data from the prompt and loads it at runtime, which better reflects real-world industrial workflows where data volumes make in-prompt embedding impractical.

---

## 2. Schema-based Prompt Content

The schema-based prompt (`{scenario_id}.scenario.txt`) contains:

### What We Give

| Section | Content |
|---------|---------|
| Business Narrative | Scenario description, structure cues |
| Data Schema | JSON structure with types (NOT actual data) |
| Data Access | How to read from `data` variable |
| Output Format | GurobiPy, print format |

### What We DON'T Give

| Omitted | Rationale |
|---------|-----------|
| Decision variables | LLM decides how to model |
| Objective function formula | LLM derives from costs |
| Constraint formulas | LLM derives from narrative |
| Boundary conditions | LLM handles edge cases |

**Philosophy:** Give LLM the business context and data structure. Let it decide how to model.

### Data Schema Section

```
{
  "name": str,                          # scenario identifier
  "periods": int,                       # number of time periods
  "products": [str, ...],               # list of product IDs
  "locations": [str, ...],              # list of location IDs

  "shelf_life": {p: int},               # shelf life in periods per product
  "lead_time": {p: int},                # order lead time per product

  "demand_curve": {p: [float, ...]},    # demand per product per period
  "demand_share": {l: float},           # fraction of total demand at each location

  "production_cap": {p: [float, ...]},  # max production per product per period
  "cold_capacity": {l: float},          # storage capacity per location
  "cold_usage": {p: float},             # storage units per unit of product

  "labor_cap": {l: [float, ...]},       # labor hours per location per period
  "labor_usage": {p: float},            # labor hours per unit sold
  "return_rate": {p: float},            # fraction of sales returned next period

  "costs": {
    "purchasing": {p: float},           # cost per unit ordered
    "inventory": {p: float},            # holding cost per unit per period
    "waste": {p: float},                # cost per unit expired
    "lost_sales": {p: float},           # penalty per unit of unmet demand
    "fixed_order": float,               # fixed cost per order placed
    "transshipment": float              # cost per unit transshipped
  },

  "constraints": {
    "moq": float,                       # minimum order quantity
    "pack_size": int,                   # order must be multiple of this
    "budget_per_period": float|null,    # max purchasing cost per period
    "waste_limit_pct": float|null       # max waste as fraction of total demand
  },

  "network": {
    "sub_edges": [[p_from, p_to], ...], # substitution edges
    "trans_edges": [[l_from, l_to], ...]# transshipment edges
  }
}
```

### Key Parameter Semantics

| Parameter | Constraint Semantics |
|-----------|---------------------|
| `shelf_life` | Inventory tracked by **remaining life**. Fresh arrivals enter at `r = shelf_life[p]`. Units at `r=1` not sold become waste. |
| `cold_capacity/cold_usage` | Volume-weighted storage: `sum(cold_usage[p] * inventory[p,l,t]) <= cold_capacity[l]` |
| `lead_time` | Orders placed in period t arrive in period t + lead_time[p]. |
| `labor_cap/labor_usage` | Labor constraint: `sum(labor_usage[p] * units_handled[p,l,t]) <= labor_cap[l,t]` |
| `return_rate` | Returns: `return_rate[p] * sales[p,l,t]` units re-enter inventory in period t+1 |
| `waste_limit_pct` | Global waste cap: `sum(waste) <= waste_limit_pct * sum(demand)` |
| `sub_edges` | Edge [A, B] means A's demand can be served by B's inventory (upward substitution). |
| `trans_edges` | Edge [L1, L2] means inventory can move from L1 to L2 at transshipment cost. |

### Data Access Section

```
- The variable `data` is pre-loaded. Do NOT use file I/O.
- Network data is nested: use data.get('network', {}).get('sub_edges', [])
- IMPORTANT: sub_edges and trans_edges are lists of lists [[a,b], ...].
  Convert to tuples for Gurobi indexing: [tuple(e) for e in sub_edges]
- Lists are 0-indexed (period t uses index [t-1])
```

### Output Format Section

```
- Output ONLY Python code
- Use GurobiPy
- Print status and objective
```

---

## 3. Data-embedded Prompt Content

The data-embedded prompt (`{scenario_id}.full.txt`) contains:

| Section | Content |
|---------|---------|
| Business Narrative | Same as schema-based |
| Full JSON Data | Complete instance data embedded in prompt |
| Output Format | GurobiPy with `import json` |

**Key Difference:** No external data loading needed. The generated code parses JSON directly from the prompt.

---

## 4. Agent Base Prompt Content

The agent base prompt (`{scenario_id}.base.txt`) contains only:

| Section | Content |
|---------|---------|
| Scenario Header | Family, archetype, scenario ID |
| Business Narrative | Description with structure cues |

**Use Case:** Multi-step agentic pipelines that inject their own instructions and guardrails.

---

## 5. Evaluation Metrics

| Metric | Definition | Formula |
|--------|------------|---------|
| **Execution Rate** | Code runs without runtime errors | `exec_ok / total` |
| **Accuracy** | Status matches AND objective within tolerance | `correct / total` |

### Accuracy Criterion

An instance is **correct** if:
1. Solver status matches ground truth (both feasible, or both infeasible)
2. For feasible instances: |y_pred - y_ref| / |y_ref| < ε

| Family | Tolerance (ε) |
|--------|---------------|
| F1–F5, F7–F8 | 0.01% |
| F6 (Hard MIP) | 5% |
