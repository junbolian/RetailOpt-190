# RetailOpt-190: A Retail Supply Chain Benchmark for Text-to-Optimization

**RetailOpt-190** is a solver-validated benchmark for evaluating **semantic reliability** in text-to-optimization. It tests whether LLM-based agents can reconstruct the *intended optimization structure*—not just produce runnable code.

The benchmark provides **190 JSON instances** (38 archetypes x 5 variations) covering common retail supply chain mechanisms, together with a reference MILP solver and standardized evaluation protocol.

---

## About This Repository

This repository contains the **standalone benchmark dataset** from the paper:

> **ReLoop: Detecting Silent Failures in LLM-Generated Optimization Code via Behavioral Verification**
>
> Junbo Jacob Lian, Yujun Sun, Huiling Chen, Chaoyu Zhang, Chung-Piaw Teo

RetailOpt-190 is released as an independent dataset to facilitate research in LLM-based optimization. This repository provides:

- **190 benchmark instances** with natural-language problem descriptions
- **380 prompt files** in two formats (zero-shot and agent-style)
- **Ground truth solutions** from a validated MILP solver
- **Standardized JSON schema** for controlled experiments

Researchers can use this dataset to benchmark their own LLMs or optimization agents. For the complete ReLoop evaluation pipeline (including model inference, semantic probes, and result analysis), please visit the main repository:

**[https://github.com/junbolian/ReLoop](https://github.com/junbolian/ReLoop)**

---

## Why RetailOpt-190?

Existing benchmarks (NL4Opt, MAMO, IndustryOR) focus on *translation fidelity*—whether models correctly parse optimization language. RetailOpt-190 targets a distinct axis: **compositional consistency**.

| Benchmark | Scenarios | Multi-period | Compositional |
|-----------|-----------|--------------|---------------|
| NL4Opt | ~100 | Few | No |
| MAMO | ~800 | Some | No |
| IndustryOR | 100 | Some | No |
| **RetailOpt-190** | **190** | **All** | **Yes** |

**Key insight**: Retail optimization involves complex constraint interactions (perishability, substitution, capacity coupling, etc.) that are prone to subtle modeling errors. RetailOpt-190 is designed to expose these failure modes through compositional scenario design.

---

## Scenario Families

RetailOpt-190 spans 8 families covering core retail planning mechanisms:

| ID | Family | Archetypes | Key Mechanisms |
|----|--------|------------|----------------|
| F1 | Core Operations | 4 | Multi-period inventory, seasonal demand, perishability, lost sales |
| F2 | Assortment & Substitution | 6 | Product substitution, promotions, ultra-short shelf life, price bands |
| F3 | Resource Constraints | 4 | Storage bottleneck, supply bottleneck, volumetric limits |
| F4 | Demand Dynamics | 6 | Demand surge, supply risk, peak failure, demand variance |
| F5 | Feasibility Stress | 4 | Impossible demand, storage overflow, strict service traps |
| F6 | Discrete Logistics | 4 | Lead time, MOQ, pack size, fixed order cost |
| F7 | Network & Multi-Echelon | 6 | Transshipment, hub-spoke, multi-sourcing, budget limits |
| F8 | Omni-channel | 4 | Reverse logistics, labor constraints, ship-from-store, sustainability |

Each archetype is instantiated with 5 numerical variants (v0-v4), yielding **190 instances**.

### Complete Archetype List

| Family | Archetypes |
|--------|------------|
| F1 | `base`, `high_waste`, `jit_logic`, `52_weeks` |
| F2 | `no_substitution`, `circular_sub`, `cannibalization`, `ultra_fresh`, `price_band_tight`, `promo_budget` |
| F3 | `storage_bottleneck`, `supply_bottleneck`, `unbalanced_network`, `volumetric_constraint` |
| F4 | `demand_surge`, `early_stockout`, `peak_failure`, `quality_hold`, `robust_variance`, `supply_risk` |
| F5 | `impossible_demand`, `storage_overflow`, `strict_service_trap`, `ultimate_stress` |
| F6 | `lead_time`, `moq_binary`, `pack_size_integer`, `fixed_order_cost` |
| F7 | `transshipment`, `hub_and_spoke`, `multi_sourcing`, `multiechelon_chain`, `ring_routing`, `budget_limit` |
| F8 | `labor_constraint`, `reverse_logistics`, `ship_from_store`, `sustainability` |

---

## Repository Structure

```
RetailOpt-190/
├── retailopt_190.parquet               # All-in-one dataset (Hugging Face format)
├── retailopt_190.jsonl                 # All-in-one dataset (JSON Lines format)
├── scenarios/
│   ├── data/                           # 190 JSON instances
│   │   ├── retail_f1_base_v0.json
│   │   ├── retail_f1_base_v1.json
│   │   └── ...
│   ├── prompts/                        # 380 prompt files (2 per scenario)
│   │   ├── {scenario_id}.scenario.txt  # Complete prompt (for zero-shot baseline)
│   │   └── {scenario_id}.base.txt      # Scenario only (for multi-step agents)
│   └── spec/
│       ├── archetypes.yaml             # 38 archetype definitions with business narratives
│       ├── retail_spec.md              # Full MILP specification
│       └── retail_prompts.md           # Prompt template documentation
├── solvers/
│   └── universal_retail_solver.py      # Reference MILP solver (ground truth)
├── eval/
│   ├── run_benchmark.py                # Batch evaluation script
│   └── benchmark_results.csv           # Ground truth solver results (190 rows)
├── tools/
│   └── generate_prompts.py             # Prompt generation from archetypes
├── requirements.txt
├── LICENSE
└── README.md
```

---

## Prompt System

RetailOpt-190 provides two prompt formats for different evaluation modes:

### 1. Zero-shot Baseline (`.scenario.txt`)

Complete single-prompt evaluation containing:
- `[SCENARIO]` - Family, archetype, and scenario ID
- `[BUSINESS DESCRIPTION]` - Full narrative with structure cues
- `[DATA SCHEMA]` - JSON structure (types only, not actual values)
- `[DATA ACCESS]` - Safe access patterns for nested data
- `[OUTPUT FORMAT]` - Output requirements (GurobiPy, print status/objective)
- `[TASK]` - "Write a GurobiPy script..."

### 2. Multi-step Agent (`.base.txt`)

Minimal prompt for agentic workflows containing only:
- `[SCENARIO]` - Family, archetype, and scenario ID
- `[BUSINESS DESCRIPTION]` - Business narrative and structure cues

Both formats provide the **same semantic information**; the difference is in guidance style (one-shot vs multi-step).

---

## Installation

**Requirements**: Python 3.9+, Gurobi with valid license

```bash
pip install -r requirements.txt
```

### Dependencies

**Core solver and data:**
- gurobipy >= 10.0.0
- numpy >= 1.26.4
- pandas >= 1.5.3
- pyyaml >= 6.0.0

**LLM agent stack (optional):**
- langchain, langchain-openai, openai
- tenacity, tiktoken

---

## Quick Start

### 1. Run reference solver on a single instance

```bash
python solvers/universal_retail_solver.py --file scenarios/data/retail_f1_base_v0.json
```

Output:
```
| retail_f1_base_v0                        | OPTIMAL         | 378,951.50           |
```

### 2. Run batch evaluation on all instances

```bash
python eval/run_benchmark.py
```

This executes the solver on all 190 JSON instances and saves results to `eval/benchmark_results.csv`.

### 3. Load dataset from Parquet (recommended)

```python
import pandas as pd
import json

# Load all-in-one dataset
df = pd.read_parquet('retailopt_190.parquet')

# Each row contains: scenario_id, prompt, data, reference_status, reference_objective
for _, row in df.iterrows():
    prompt = row['prompt']
    data = json.loads(row['data'])  # Parse JSON string to dict

    # Send prompt to your LLM
    generated_code = your_llm(prompt)

    # Execute with data pre-loaded
    exec(generated_code, {'data': data})

    # Compare with reference
    print(f"Reference: {row['reference_status']}, {row['reference_objective']}")
```

### 4. Load from individual files

```python
import json

# Load a prompt
with open('scenarios/prompts/retail_f1_base_v0.scenario.txt', 'r') as f:
    prompt = f.read()

# Load corresponding data
with open('scenarios/data/retail_f1_base_v0.json', 'r') as f:
    data = json.load(f)

# Send prompt to your LLM, get generated code
generated_code = your_llm(prompt)

# Execute generated code with data pre-loaded
exec(generated_code, {'data': data})

# Compare output with ground truth in eval/benchmark_results.csv
```

Ground truth results are stored in `eval/benchmark_results.csv` with columns:
- `scenario`: Scenario name (e.g., `retail_f1_base_v0`)
- `status`: Solver status (`OPTIMAL`, `INFEASIBLE`, etc.)
- `objective`: Objective value as string
- `objective_numeric`: Objective value as float
- `time_sec`: Solve time in seconds

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
| F1-F5, F7-F8 | LP / easy MIP | 0.01% |
| F6 | Hard MIP (MOQ, pack-size) | 10% |

The relaxed tolerance for F6 accounts for binary/integer variables where the 60-second time limit may yield near-optimal solutions.

### Evaluation Metrics

- **Execution Rate** = (exec_ok / total) - Percentage of instances that run without error
- **Accuracy** = (correct / total) - Percentage matching ground truth within tolerance
- **Silent Failure Rate** = (exec_ok - correct) / exec_ok - Executable code with wrong answer

---

## JSON Schema

Each instance follows a universal schema:

```json
{
  "name": "retail_f1_base_v0",
  "description": "Standard seasonal retail scenario.",
  "periods": 20,
  "products": ["SKU_Basic", "SKU_Premium", "SKU_ShortLife"],
  "locations": ["DC1", "DC2", "DC3", "DC4", "DC5"],

  "shelf_life": {"SKU_Basic": 10, "SKU_Premium": 8, "SKU_ShortLife": 4},
  "lead_time": {"SKU_Basic": 0, "SKU_Premium": 0, "SKU_ShortLife": 0},

  "demand_curve": {"SKU_Basic": [303, 311, ...], ...},
  "demand_share": {"DC1": 0.25, "DC2": 0.2, ...},

  "production_cap": {"SKU_Basic": [800, 800, ...], ...},
  "cold_capacity": {"DC1": 4000, "DC2": 3500, ...},
  "cold_usage": {"SKU_Basic": 1.0, ...},

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

### Critical Semantics

**Substitution edges**: `[p_from, p_to]` means `p_from`'s demand can be served by `p_to`'s inventory (upward substitution).
- Example: `["SKU_Basic", "SKU_Premium"]` means Premium can serve Basic's demand when Basic stock is insufficient.
- `S[Basic, Premium, l, t]` = units of Basic's demand fulfilled by Premium at location l in period t.

**Network data access**: Use nested safe access pattern:
```python
sub_edges = data.get('network', {}).get('sub_edges', [])
trans_edges = data.get('network', {}).get('trans_edges', [])
# NOT: data['sub_edges']  # This causes KeyError!
```

**Demand calculation**:
```python
demand[p, l, t] = data['demand_curve'][p][t-1] * data['demand_share'][l]
```

---

## Reference MILP Solver

The Universal Retail Solver (`solvers/universal_retail_solver.py`) implements a modular MILP with:

### Decision Variables
- `Q[p,l,t]` - Order quantity
- `I[p,l,t,a]` - Inventory by remaining-life age bucket
- `y[p,l,t]` - Sales quantity
- `W[p,l,t]` - Waste (expired inventory)
- `L[p,l,t]` - Lost sales (unmet demand)
- `S[pf,pt,l,t]` - Substitution flows
- `X[p,src,dst,t]` - Transshipment between locations
- `z[p,l,t]` - Binary trigger for MOQ/fixed cost
- `n[p,l,t]` - Pack size multiplier (integer)

### Capabilities (driven by JSON fields)
- Perishable inventory with remaining-life tracking
- Lead time with delivery delays
- MOQ (minimum order quantity) triggers
- Pack size constraints (integer multiples)
- Production/procurement capacity
- Storage capacity by location
- Substitution flows between products
- Transshipment between locations
- Reverse logistics (product returns)
- Labor capacity constraints
- Budget constraints per period
- Fixed order costs

### Solver Settings
- TimeLimit: 60 seconds
- MIPGap: 1%
- Threads: 1
- Seed: 0

Full specification: `scenarios/spec/retail_spec.md`

---

## License

- **Code**: MIT
- **Data (RetailOpt-190)**: CC BY 4.0

---

## Citation

If you use RetailOpt-190 in your research, please cite our paper:

```bibtex
@article{lian2026reloop,
  author    = {Junbo Jacob Lian and Yujun Sun and Huiling Chen and Chaoyu Zhang and Chung-Piaw Teo},
  title     = {ReLoop: Detecting Silent Failures in LLM-Generated Optimization Code via Behavioral Verification},
  journal   = {arXiv preprint},
  year      = {2026}
}
```

---

## Related Resources

- **Hugging Face Dataset**: [https://huggingface.co/datasets/Jacoblian/RetailOpt-190](https://huggingface.co/datasets/Jacoblian/RetailOpt-190) - Download dataset directly
- **ReLoop Framework**: [https://github.com/junbolian/ReLoop](https://github.com/junbolian/ReLoop) - Complete implementation of the ReLoop verification pipeline
- **Paper**: Link to be added upon publication

---

## Contact

For questions or issues, please open a GitHub issue or contact the authors.
