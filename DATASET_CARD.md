---
language:
- en
license: cc-by-4.0
size_categories:
- n<1K
task_categories:
- text-generation
tags:
- optimization
- operations-research
- supply-chain
- retail
- milp
- code-generation
- benchmark
pretty_name: RetailOpt-190
configs:
- config_name: default
  data_files:
  - split: test
    path: retailopt_190.parquet
dataset_info:
  features:
    - name: scenario_id
      dtype: string
    - name: prompt_schema
      dtype: string
    - name: prompt_full
      dtype: string
    - name: data
      dtype: string
    - name: reference_status
      dtype: string
    - name: reference_objective
      dtype: float64
  splits:
    - name: test
      num_examples: 190
---

# RetailOpt-190: A Retail Supply Chain Benchmark for Text-to-Optimization

[![arXiv](https://img.shields.io/badge/arXiv-2602.15983-b31b1b.svg)](https://arxiv.org/abs/2602.15983)
[![GitHub](https://img.shields.io/badge/GitHub-RetailOpt--190-blue?logo=github)](https://github.com/junbolian/RetailOpt-190)
[![ReLoop](https://img.shields.io/badge/GitHub-ReLoop-blue?logo=github)](https://github.com/junbolian/ReLoop)
[![License: Data](https://img.shields.io/badge/Data-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

**RetailOpt-190** is a solver-validated benchmark for evaluating semantic reliability in text-to-optimization. It tests whether LLM-based agents can reconstruct the intended optimization structure—not just produce runnable code.

## Dataset Description

- **Repository:** [https://github.com/junbolian/RetailOpt-190](https://github.com/junbolian/RetailOpt-190)
- **Paper:** [ReLoop: Structured Modeling and Behavioral Verification for Reliable LLM-Based Optimization](https://arxiv.org/abs/2602.15983)
- **Point of Contact:** Junbo Jacob Lian

### Dataset Summary

RetailOpt-190 contains 190 retail supply chain optimization instances designed to test compositional consistency in LLM-generated optimization code. Each instance includes a natural-language problem description, structured JSON data, and ground truth solutions from a validated MILP solver.

The benchmark spans 8 scenario families and 38 archetypes covering core retail planning mechanisms:

| Family | Name | Archetypes | Key Mechanisms |
|--------|------|------------|----------------|
| F1 | Core Operations | 4 | Multi-period inventory, seasonal demand, perishability |
| F2 | Assortment & Substitution | 6 | Product substitution, promotions, ultra-short shelf life |
| F3 | Resource Constraints | 4 | Storage bottleneck, supply bottleneck, volumetric limits |
| F4 | Demand Dynamics | 6 | Demand surge, supply risk, peak failure |
| F5 | Feasibility Stress | 4 | Impossible demand, storage overflow, strict service traps |
| F6 | Discrete Logistics | 4 | Lead time, MOQ, pack size, fixed order cost |
| F7 | Network & Multi-Echelon | 6 | Transshipment, hub-spoke, multi-sourcing |
| F8 | Omni-channel | 4 | Reverse logistics, labor constraints, sustainability |

### Languages

English

## Prompt Formats

RetailOpt-190 provides **two prompt formats** in the dataset:

| Format | Field | Data Location | Role | Use Case |
|--------|-------|---------------|------|----------|
| **Data-embedded** | `prompt_full` | In prompt | **Default evaluation format** | Direct comparison with other benchmarks (NL4Opt, MAMO, IndustryOR) |
| **Schema-based** | `prompt_schema` | External (runtime) | ReLoop verification format | Large datasets, agentic workflows |

**Data-embedded (`prompt_full`) is the default evaluation format.** ReLoop and all baseline experiments use this format to maintain a consistent input structure across models and datasets (NL4Opt, MAMO, IndustryOR all embed data in prompts). Schema-based (`prompt_schema`) separates data from the prompt and loads it at runtime, which better reflects real-world industrial workflows where data volumes make in-prompt embedding impractical.

Both formats provide the **same semantic information**—only the data delivery method differs.

## Dataset Structure

### Data Fields

| Field | Type | Description |
|-------|------|-------------|
| `scenario_id` | string | Unique scenario identifier (e.g., `retail_f1_base_v0`) |
| `prompt_schema` | string | Schema-based prompt (data loaded at runtime via `data` variable) |
| `prompt_full` | string | Data-embedded prompt (full JSON data in prompt) |
| `data` | string | JSON-formatted instance data (parse with `json.loads()`) |
| `reference_status` | string | Ground truth solver status (`OPTIMAL`, `INFEASIBLE`, etc.) |
| `reference_objective` | float | Ground truth objective value (null if infeasible) |

### Data Splits

| Split | Examples |
|-------|----------|
| test | 190 |

## Usage

### Loading the Dataset

```python
from datasets import load_dataset
import json

dataset = load_dataset("Jacoblian/RetailOpt-190", split="test")
sample = dataset[0]

print(sample['scenario_id'])        # e.g., "retail_f1_base_v0"
print(sample['prompt_schema'][:200])  # Schema-based prompt
print(sample['prompt_full'][:200])    # Data-embedded prompt
```

### Option A: Data-embedded Evaluation (Default)

Use `prompt_full` for standard evaluation (compatible with other benchmarks):

```python
from datasets import load_dataset

dataset = load_dataset("Jacoblian/RetailOpt-190", split="test")

for sample in dataset:
    prompt = sample['prompt_full']  # Data is already in prompt

    generated_code = your_llm(prompt)
    exec(generated_code)  # Code parses JSON from prompt itself

    print(f"Reference: {sample['reference_status']}, {sample['reference_objective']}")
```

### Option B: Schema-based Evaluation

Use `prompt_schema` when you need external data loading (ReLoop pipeline, agentic workflows):

```python
from datasets import load_dataset
import json

dataset = load_dataset("Jacoblian/RetailOpt-190", split="test")

for sample in dataset:
    prompt = sample['prompt_schema']
    data = json.loads(sample['data'])

    generated_code = your_llm(prompt)
    exec(generated_code, {'data': data})  # Data pre-loaded

    print(f"Reference: {sample['reference_status']}, {sample['reference_objective']}")
```

### Evaluation Metrics

- **Execution Rate**: Percentage of instances that run without error
- **Accuracy**: Percentage matching ground truth (status + objective within tolerance)
- **Silent Failure Rate**: Executable code with incorrect answer

### Accuracy Tolerances

| Scenarios | Problem Type | Tolerance |
|-----------|--------------|-----------|
| F1-F5, F6 (lead_time, moq_binary), F7-F8 | LP / easy MIP | 0.01% |
| F6 (pack_size_integer, fixed_order_cost) | Hard MIP, hits 60s time limit | 1% |

Only 2 of the 4 F6 archetypes require the relaxed tolerance. `pack_size_integer` and `fixed_order_cost` hit the 60-second time limit and return near-optimal solutions; the other F6 archetypes solve to optimality within seconds.

## Inventory Dynamics

All archetypes share a common shelf-life-aware inventory accounting. Inventory is indexed by `(p, l, t, r)` where `r` is the **remaining periods of life** (FIFO: `r=1` is oldest, `r=shelf_life[p]` is freshest):

```
(1) Fresh inflow:    I[p,l,t,SL] = Q[p,l,t-LT[p]] + transshipment_net[p,l,t] + returns[p,l,t]
(2) Aging:           I[p,l,t+1,r] = I[p,l,t,r+1] - sales[p,l,t,r+1]   for r = 1..SL-1
(3) Waste:           W[p,l,t] = I[p,l,t,1] - sales[p,l,t,1]
(4) Sales bound:     sales[p,l,t,r] <= I[p,l,t,r]
(5) Holding cost charged on (I[p,l,t,r] - sales[p,l,t,r]) for r >= 2 only.
```

`Q[p, l, t]` is the **per-location** order quantity (decision variable); aggregate production capacity couples across locations: `sum_l Q[p,l,t] <= production_cap[p][t]`. `transshipment_net = 0` when `trans_edges = []`, and `returns = return_rate[p] * sum_a sales[p,l,t-1,a]`.

**Substitution semantics.** Edge `[p_from, p_to]` in `network.sub_edges` means `p_to`'s inventory can serve `p_from`'s demand. The sub-inventory binding `sum_{p_from} S[p_from, p, l, t] <= sum_r sales[p, l, t, r]` ensures absorbed substitution demand at product `p` is backed by real sales from `p`'s inventory — without this, the optimizer could fictitiously route high-penalty unmet demand through substitution variables alone.

## Dataset Creation

### Source Data

All instances are synthetically generated from 38 archetype specifications. Each archetype is instantiated with 5 numerical variants (v0-v4) via controlled parameter perturbations.

### Annotations

Ground truth solutions are computed using a validated MILP solver (Gurobi) with the following settings:
- TimeLimit: 60 seconds
- MIPGap: 1%
- Threads: 1

## Additional Information

### Citation

```bibtex
@article{lian2026reloop,
  author    = {Junbo Jacob Lian and Yujun Sun and Huiling Chen and Chaoyu Zhang and Hanzhang Qin and Chung-Piaw Teo},
  title     = {ReLoop: Structured Modeling and Behavioral Verification for Reliable LLM-Based Optimization},
  journal   = {arXiv preprint arXiv:2602.15983},
  year      = {2026},
  url       = {https://arxiv.org/abs/2602.15983}
}
```

### License

- **Code**: MIT
- **Data**: CC BY 4.0

### Related Resources

- **ReLoop Framework**: [https://github.com/junbolian/ReLoop](https://github.com/junbolian/ReLoop) - Complete implementation of the ReLoop verification pipeline
