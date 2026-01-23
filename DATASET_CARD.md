---
language:
- en
license: cc-by-4.0
size_categories:
- n<1K
task_categories:
- text-generation
- text2text-generation
tags:
- optimization
- operations-research
- supply-chain
- retail
- milp
- code-generation
- benchmark
pretty_name: RetailOpt-190
dataset_info:
  features:
    - name: scenario_id
      dtype: string
    - name: prompt
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

**RetailOpt-190** is a solver-validated benchmark for evaluating semantic reliability in text-to-optimization. It tests whether LLM-based agents can reconstruct the intended optimization structure—not just produce runnable code.

## Dataset Description

- **Repository:** [https://github.com/junbolian/RetailOpt-190](https://github.com/junbolian/RetailOpt-190)
- **Paper:** ReLoop: Detecting Silent Failures in LLM-Generated Optimization Code via Behavioral Verification
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

## Dataset Structure

### Data Fields

| Field | Type | Description |
|-------|------|-------------|
| `scenario_id` | string | Unique scenario identifier (e.g., `retail_f1_base_v0`) |
| `prompt` | string | Natural-language problem description with structure cues |
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

# Load dataset
dataset = load_dataset("junbolian/RetailOpt-190", split="test")

# Access a sample
sample = dataset[0]
print(sample['scenario_id'])  # e.g., "retail_f1_base_v0"
print(sample['prompt'][:200])  # First 200 chars of prompt

# Parse JSON data
data = json.loads(sample['data'])
print(data['periods'])  # Number of time periods
print(data['products'])  # List of products
```

### Benchmarking Your Model

```python
from datasets import load_dataset
import json

dataset = load_dataset("junbolian/RetailOpt-190", split="test")

for sample in dataset:
    # Get prompt and data
    prompt = sample['prompt']
    data = json.loads(sample['data'])

    # Generate code with your LLM
    generated_code = your_llm(prompt)

    # Execute generated code
    exec(generated_code, {'data': data})

    # Compare with ground truth
    print(f"Reference: {sample['reference_status']}, {sample['reference_objective']}")
```

### Evaluation Metrics

- **Execution Rate**: Percentage of instances that run without error
- **Accuracy**: Percentage matching ground truth (status + objective within tolerance)
- **Silent Failure Rate**: Executable code with incorrect answer

### Accuracy Tolerances

| Family | Problem Type | Tolerance |
|--------|--------------|-----------|
| F1-F5, F7-F8 | LP / easy MIP | 0.01% |
| F6 | Hard MIP (MOQ, pack-size) | 10% |

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
@article{lian2025reloop,
  author    = {Junbo Jacob Lian and Yujun Sun and Huiling Chen and Chaoyu Zhang and Chung-Piaw Teo},
  title     = {ReLoop: Detecting Silent Failures in LLM-Generated Optimization Code via Behavioral Verification},
  journal   = {arXiv preprint},
  year      = {2026}
}
```

### License

- **Code**: MIT
- **Data**: CC BY 4.0

### Related Resources

- **ReLoop Framework**: [https://github.com/junbolian/ReLoop](https://github.com/junbolian/ReLoop) - Complete implementation of the ReLoop verification pipeline
