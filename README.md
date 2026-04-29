# RetailOpt-190: A Retail Supply Chain Benchmark for Text-to-Optimization

[![arXiv](https://img.shields.io/badge/arXiv-2602.15983-b31b1b.svg)](https://arxiv.org/abs/2602.15983)
[![Dataset](https://img.shields.io/badge/%F0%9F%A4%97%20HuggingFace-RetailOpt--190-yellow)](https://huggingface.co/datasets/Jacoblian/RetailOpt-190)
[![Code](https://img.shields.io/badge/GitHub-RetailOpt--190-blue?logo=github)](https://github.com/junbolian/RetailOpt-190)
[![ReLoop](https://img.shields.io/badge/GitHub-ReLoop-blue?logo=github)](https://github.com/junbolian/ReLoop)
[![License: Code](https://img.shields.io/badge/Code-MIT-green.svg)](LICENSE)
[![License: Data](https://img.shields.io/badge/Data-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

**RetailOpt-190** is a solver-validated benchmark for evaluating **semantic reliability** in text-to-optimization. It tests whether LLM-based agents can reconstruct the *intended optimization structure*—not just produce runnable code.

The benchmark provides **190 JSON instances** (38 archetypes x 5 variations) covering common retail supply chain mechanisms, together with a reference MILP solver and standardized evaluation protocol.

---

## About This Repository

This repository contains the **standalone benchmark dataset** from the paper:

> **ReLoop: Structured Modeling and Behavioral Verification for Reliable LLM-Based Optimization**
>
> Junbo Jacob Lian, Yujun Sun, Huiling Chen, Chaoyu Zhang, Hanzhang Qin, Chung-Piaw Teo

RetailOpt-190 is released as an independent dataset to facilitate research in LLM-based optimization. This repository provides:

- **190 benchmark instances** with natural-language problem descriptions
- **380 prompt files** in two formats (zero-shot and agent-style)
- **Ground truth solutions** from a validated MILP solver
- **Standardized JSON schema** for controlled experiments

Researchers can use this dataset to benchmark their own LLMs or optimization agents. For the complete ReLoop evaluation pipeline (including model inference, semantic probes, and result analysis), please visit the main repository:

**[https://github.com/junbolian/ReLoop](https://github.com/junbolian/ReLoop)**

### Related Resources

- **Hugging Face Dataset**: [https://huggingface.co/datasets/Jacoblian/RetailOpt-190](https://huggingface.co/datasets/Jacoblian/RetailOpt-190) - Download dataset directly
- **ReLoop Framework**: [https://github.com/junbolian/ReLoop](https://github.com/junbolian/ReLoop) - Complete implementation of the ReLoop verification pipeline
- **Paper**: [arXiv:2602.15983](https://arxiv.org/abs/2602.15983)

---

## Why RetailOpt-190?

Existing benchmarks (NL4Opt, MAMO, IndustryOR) focus on *translation fidelity*—whether models correctly parse optimization language. RetailOpt-190 targets a distinct axis: **compositional consistency**.

| Benchmark | Scenarios | Multi-period | Compositional |
|-----------|-----------|--------------|---------------|
| NL4Opt | 289 | Few | No |
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
│   │   └── ...
│   ├── prompts/                        # 570 prompt files (3 per scenario)
│   │   ├── {scenario_id}.scenario.txt  # Schema-based (data loaded at runtime)
│   │   ├── {scenario_id}.full.txt      # Data-embedded (full JSON in prompt)
│   │   └── {scenario_id}.base.txt      # Agent base (scenario only)
│   └── spec/
│       ├── archetypes.yaml             # 38 archetype definitions
│       ├── retail_spec.md              # Full MILP specification
│       └── retail_prompts.md           # Prompt documentation
├── solvers/
│   └── universal_retail_solver.py      # Reference MILP solver (ground truth)
├── eval/
│   ├── run_benchmark.py                # Batch evaluation script
│   └── benchmark_results.csv           # Ground truth solver results
├── tools/
│   └── generate_prompts.py             # Prompt generation from archetypes
├── requirements.txt
├── LICENSE
└── README.md
```

---

## Prompt System

RetailOpt-190 provides **three prompt formats** for different evaluation scenarios:

### Prompt Formats

| Format | Data Location | Role | Use Case |
|--------|---------------|------|----------|
| **Data-embedded** (`.full.txt`) | In prompt | **Default evaluation format** | Direct comparison with other benchmarks (NL4Opt, MAMO, IndustryOR) |
| **Schema-based** (`.scenario.txt`) | External (runtime) | ReLoop verification format | Large datasets, production scenarios, agentic workflows |
| **Agent base** (`.base.txt`) | None | Minimal prompt | Multi-step agents that inject their own guardrails |

**Data-embedded (`prompt_full`) is the default evaluation format.** ReLoop and all baseline experiments use this format to maintain a consistent input structure across models and datasets (NL4Opt, MAMO, IndustryOR all embed data in prompts). Schema-based (`prompt_schema`) separates data from the prompt and loads it at runtime, which better reflects real-world industrial workflows where data volumes make in-prompt embedding impractical.

### 1. Schema-based (`.scenario.txt`)

Data is loaded externally at runtime via `data` variable.

```
[SCENARIO] - Family, archetype, scenario ID
[BUSINESS DESCRIPTION] - Full narrative with structure cues
[DATA SCHEMA] - JSON structure (types only, NOT actual values)
[DATA ACCESS] - How to access the pre-loaded `data` variable
[OUTPUT FORMAT] - GurobiPy requirements
[TASK] - Write optimization script
```

**Advantages**: Handles large datasets, tests realistic data access patterns.

### 2. Data-embedded (`.full.txt`)

Complete JSON data embedded directly in the prompt.

```
[SCENARIO] - Family, archetype, scenario ID
[BUSINESS DESCRIPTION] - Full narrative with structure cues
[DATA] - Full JSON data embedded in prompt
[OUTPUT FORMAT] - GurobiPy requirements
[TASK] - Parse JSON and solve
```

**Advantages**: No external data loading, compatible with text-to-solution benchmarks.

### 3. Agent Base (`.base.txt`)

Minimal prompt for multi-step agentic workflows.

```
[SCENARIO] - Family, archetype, scenario ID
[BUSINESS DESCRIPTION] - Business narrative and structure cues
```

**Use case**: ReLoop and similar agents that inject their own guardrails.

### Semantic Equivalence

All three formats provide the **same semantic information**. The difference is only in:
- **Data delivery**: External vs embedded
- **Guidance style**: Full instructions vs minimal

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

# Columns: scenario_id, prompt_schema, prompt_full, data, reference_status, reference_objective

# Option A: Data-embedded evaluation (default, no external data needed)
for _, row in df.iterrows():
    prompt = row['prompt_full']  # Data is already in prompt
    generated_code = your_llm(prompt)
    exec(generated_code)  # Code parses JSON from prompt itself

# Option B: Schema-based evaluation (ReLoop pipeline, data loaded at runtime)
for _, row in df.iterrows():
    prompt = row['prompt_schema']
    data = json.loads(row['data'])
    generated_code = your_llm(prompt)
    exec(generated_code, {'data': data})  # Data pre-loaded
```

### 4. Load from individual files

```python
import json

scenario_id = 'retail_f1_base_v0'

# Option A: Data-embedded (.full.txt) - default evaluation format
with open(f'scenarios/prompts/{scenario_id}.full.txt', 'r') as f:
    prompt = f.read()  # Data is embedded in prompt
generated_code = your_llm(prompt)
exec(generated_code)  # Code parses JSON internally

# Option B: Schema-based (.scenario.txt) - ReLoop pipeline
with open(f'scenarios/prompts/{scenario_id}.scenario.txt', 'r') as f:
    prompt = f.read()
with open(f'scenarios/data/{scenario_id}.json', 'r') as f:
    data = json.load(f)
generated_code = your_llm(prompt)
exec(generated_code, {'data': data})
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

| Scenarios | Problem Type | Tolerance |
|-----------|--------------|-----------|
| F1-F5, F6 (lead_time, moq_binary), F7-F8 | LP / easy MIP | 0.01% |
| F6 (pack_size_integer, fixed_order_cost) | Hard MIP, hits 60s time limit | 1% |

Only 2 of the 4 F6 archetypes (`pack_size_integer` and `fixed_order_cost`) require the relaxed 1% tolerance. These are the only scenarios where the solver hits the 60-second time limit and returns a near-optimal rather than provably optimal solution. The other F6 archetypes (`lead_time`, `moq_binary`) solve to optimality within seconds and use the standard 0.01% tolerance.

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
- **Sub-inventory binding**: `sum_{p_from} S[p_from, p, l, t] <= sum_r sales[p, l, t, r]` — incoming substitution at product `p` must be backed by actual sales drawn from `p`'s inventory. Without this binding, a cost-minimizing solver could fictitiously absorb high-penalty unmet demand at a low-penalty product through substitution variables alone.

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

### Inventory Dynamics (Shelf-Life Tracking)

All archetypes index inventory by `(p, l, t, r)` where `r` is the **remaining periods of life** (FIFO: `r=1` is oldest, `r=shelf_life[p]` is freshest):

```
(1) Fresh inflow:    I[p,l,t,SL] = Q[p,l,t-LT[p]] + transshipment_net[p,l,t] + returns[p,l,t]
(2) Aging:           I[p,l,t+1,r] = I[p,l,t,r+1] - sales[p,l,t,r+1]   for r = 1..SL-1
(3) Waste:           W[p,l,t] = I[p,l,t,1] - sales[p,l,t,1]
(4) Sales bound:     sales[p,l,t,r] <= I[p,l,t,r]
(5) Holding cost charged on (I[p,l,t,r] - sales[p,l,t,r]) for r >= 2 only.
```

Key conventions:

- `Q[p, l, t]` is the **per-location** order quantity (decision variable). Aggregate production capacity binds across locations: `sum_l Q[p,l,t] <= production_cap[p][t]`.
- `transshipment_net[p, l, t] = inflow − outflow` along `trans_edges`. Zero when `trans_edges = []`.
- `returns[p, l, t] = return_rate[p] * sum_a sales[p, l, t-1, a]` for `t > 1`, else `0`.
- Treat `Q[p, l, s] = 0` for `s < 1` (no orders placed before the horizon).

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
  author    = {Junbo Jacob Lian and Yujun Sun and Huiling Chen and Chaoyu Zhang and Hanzhang Qin and Chung-Piaw Teo},
  title     = {ReLoop: Structured Modeling and Behavioral Verification for Reliable LLM-Based Optimization},
  journal   = {arXiv preprint arXiv:2602.15983},
  year      = {2026},
  url       = {https://arxiv.org/abs/2602.15983}
}
```

---

## Contact

For questions or issues, please open a GitHub issue or contact the authors.
