# RetailOpt-190: A Retail Supply Chain Benchmark for Text-to-Optimization

**RetailOpt-190** is a solver-validated benchmark for evaluating **semantic reliability** in text-to-optimization. It tests whether LLM-based agents can reconstruct the *intended optimization structure*—not just produce runnable code.

The benchmark provides **190 JSON instances** (38 archetypes × 5 variations) covering common retail supply chain mechanisms, together with a reference MILP solver and standardized evaluation protocol.

---

## Why RetailOpt-190?

Existing benchmarks (NL4Opt, MAMO, IndustryOR) focus on *translation fidelity*—whether models correctly parse optimization language. RetailOpt-190 targets a distinct axis: **compositional consistency**.

| Benchmark | Scenarios | Multi-period | Compositional | Semantic Probes |
|-----------|-----------|--------------|---------------|-----------------|
| NL4Opt | ~100 | Few | ✗ | ✗ |
| MAMO | ~800 | Some | ✗ | ✗ |
| IndustryOR | 100 | Some | ✗ | ✗ |
| **RetailOpt-190** | **190** | **All** | **✓** | **✓** |

**Key insight**: Retail optimization is prone to *silent failures*—models that solve successfully but silently omit critical constraint couplings. RetailOpt-190 specifically stresses these failure modes.

---

## Key Features

| Feature | Description |
|---------|-------------|
| **Compositional Focus** | Difficulty arises from constraint *interactions*, not linguistic complexity |
| **Solver Validation** | Every instance validated with reference MILP; ground truth recorded |
| **Semantic Probes** | 14 lightweight tests that detect specific implementation errors |
| **Universal Schema** | All instances share consistent JSON format for controlled comparison |
| **Reproducible** | Deterministic generation from archetype specifications |

---

## Scenario Families

RetailOpt-190 spans 8 families covering core retail planning mechanisms:

| ID | Family | #Arch | Perishable | Capacity | Substitution | Discrete | Network | Stress |
|----|--------|-------|------------|----------|--------------|----------|---------|--------|
| F1 | Core Operations | 4 | ✓ | ✓ | ✓ | -- | -- | -- |
| F2 | Assortment | 6 | ✓ | ✓ | ✓ | ✓ | -- | -- |
| F3 | Resources | 4 | -- | ✓ | -- | -- | -- | -- |
| F4 | Dynamics | 6 | ✓ | ✓ | -- | -- | -- | ✓ |
| F5 | Feasibility | 4 | -- | ✓ | -- | -- | -- | ✓ |
| F6 | Logistics | 4 | -- | -- | -- | ✓ | -- | -- |
| F7 | Network | 6 | -- | ✓ | -- | ✓ | ✓ | -- |
| F8 | Omni-channel | 4 | ✓ | ✓ | -- | -- | -- | -- |
| | **Total** | **38** | | | | | | |

Each archetype is instantiated with 5 numerical variants via controlled perturbations (±15%), yielding **190 instances**.

---

## Repository Structure

```
RetailOpt-190/
├── data/                           # 190 JSON instances
│   ├── retail_f1_base_v0.json
│   ├── retail_f1_base_v1.json
│   └── ...
├── solver/
│   └── universal_retail_solver.py  # Reference MILP (ground truth)
├── prompts/
│   ├── {scenario_id}.scenario.txt  # Complete prompt (for zero-shot baseline)
│   └── {scenario_id}.base.txt      # Scenario only (for multi-step agents)
├── probes/
│   └── semantic_probes.py          # 14 semantic verification tests
├── spec/
│   ├── archetypes.yaml             # 38 archetype definitions
│   ├── retail_spec.md              # Full MILP specification
│   └── retail_prompts.md           # Prompt template documentation
├── eval/
│   ├── run_benchmark.py            # Batch evaluation script
│   └── ground_truth.csv            # Reference solver results
└── README.md
```

---

## Installation

**Requirements**: Python 3.9+, Gurobi with valid license

```bash
pip install gurobipy pyyaml numpy pandas
```

---

## Quick Start

### 1. Run reference solver on a single instance

```bash
python solver/universal_retail_solver.py \
  --instance data/retail_f1_base_v0.json
```

Output:
```
status: 2
objective: 12345.67
```

### 2. Batch evaluation

```bash
python eval/run_benchmark.py \
  --data_dir data/ \
  --prompts_dir prompts/ \
  --output results.csv
```

---

## Evaluation Protocol

Agents are evaluated under a fixed prompt-and-execute contract:

1. **Input**: Natural-language description specifying active mechanisms
2. **Data loading**: Instance JSON loaded as `data` variable at runtime (not in prompt)
3. **Output**: Python script that builds MILP, solves, and prints status/objective
4. **Accuracy**: Status must match ground truth; for feasible instances, objective within tolerance

### Accuracy Tolerances

| Family | Problem Type | Tolerance |
|--------|--------------|-----------|
| F1–F5, F7–F8 | LP / easy MIP | 10⁻⁴ |
| F6 | Hard MIP (MOQ, pack-size) | 10% |

The relaxed tolerance for F6 accounts for binary/integer variables where the 60-second time limit may yield near-optimal solutions.

---

## JSON Schema

Each instance follows a universal schema:

```json
{
  "scenario_id": "retail_f1_base_v0",
  "periods": 20,
  "products": ["SKU_Basic", "SKU_Premium", "SKU_ShortLife"],
  "locations": ["DC1", "DC2", "DC3"],
  
  "shelf_life": {"SKU_Basic": 5, "SKU_Premium": 4, "SKU_ShortLife": 2},
  "lead_time": {"SKU_Basic": 0, "SKU_Premium": 0, "SKU_ShortLife": 0},
  
  "demand_curve": {"SKU_Basic": [100, 110, ...], ...},
  "demand_share": {"DC1": 0.4, "DC2": 0.35, "DC3": 0.25},
  
  "production_cap": {"SKU_Basic": [500, 500, ...], ...},
  "cold_capacity": {"DC1": 2000, ...},
  "cold_usage": {"SKU_Basic": 1.0, ...},
  
  "costs": {
    "purchasing": {"SKU_Basic": 10, ...},
    "inventory": {"SKU_Basic": 1, ...},
    "waste": {"SKU_Basic": 5, ...},
    "lost_sales": {"SKU_Basic": 20, ...}
  },
  
  "network": {
    "sub_edges": [["SKU_Basic", "SKU_Premium"]],
    "trans_edges": []
  },
  
  "constraints": {
    "moq": 0,
    "pack_size": 1,
    "budget_per_period": null,
    "waste_limit_pct": null
  }
}
```

### Critical Semantics

**Substitution edges**: `[p_from, p_to]` means `p_from`'s demand can be served by `p_to`'s inventory (upward substitution).

**Network data access**: Use nested access pattern:
```python
sub_edges = data.get('network', {}).get('sub_edges', [])
# NOT: data['sub_edges']  # This causes KeyError!
```

---

## Semantic Probes

14 probes detect specific implementation errors by testing *behavior*, not code:

| # | Probe | Mechanism | Detection |
|---|-------|-----------|-----------|
| 1 | `substitution_basic` | S variables | Objective range |
| 2 | `demand_route_constraint` | S_out ≤ demand | UNBOUNDED |
| 3 | `no_substitution` | Empty edges | Spurious benefit |
| 4 | `production_capacity` | Prod cap | Objective bound |
| 5 | `storage_capacity` | Storage cap | INFEASIBLE |
| 6 | `aging_dynamics` | Shelf-life | Waste cost |
| 7 | `lost_sales_slack` | L variable | INFEASIBLE |
| 8 | `nonnegativity` | I ≥ 0 | Negative inventory |
| 9 | `initialization` | t=1 init | Objective = 0 |
| 10 | `lead_time` | Lead time | Delivery timing |
| 11 | `moq` | Min order qty | MOQ enforcement |
| 12 | `transshipment` | Network flows | Trans constraint |
| 13 | `labor_capacity` | Labor limits | Capacity check |
| 14 | `holding_cost` | End-of-period | Objective too low |

**Most critical**: `initialization` (missing I[p,l,1,k]=0 → objective ≈ 0) and `holding_cost` (using I instead of I-y → objective ~60% too low).

---

## Reference MILP

The Universal Retail Solver implements a modular MILP with:

- **Perishable inventory dynamics**: Aging by remaining-life buckets
- **Substitution flow conservation**: Directed edges with demand routing
- **Shared capacity coupling**: Storage and production limits across products
- **Discrete procurement**: MOQ triggers, pack-size integrality
- **Network flows**: Transshipment between locations

Full specification: `spec/retail_spec.md`

---

## License

- **Code**: MIT
- **Data (RetailOpt-190)**: CC BY 4.0

---

## Citation

```bibtex
@inproceedings{retailopt190,
  author    = {Junbo Jacob Lian and Yujun Sam Sun and Diego Klabjan},
  title     = {RetailOpt-190: A Solver-Validated Benchmark for Semantic 
               Reliability in Text-to-Optimization},
  booktitle = {Advances in Neural Information Processing Systems (NeurIPS)},
  year      = {2026}
}
```

---

## Contact

For questions or issues, please open a GitHub issue or contact the authors.
