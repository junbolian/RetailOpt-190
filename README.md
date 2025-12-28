# RetailOpt-190

**RetailOpt-190** is a solver-validated benchmark for **text-to-optimization** in retail supply chains.  
It provides **190 JSON instances** (38 archetypes × 5 variations) spanning common retail planning mechanisms—multi-period inventory, perishability, shared capacity, substitution, discrete procurement logic, budgets/waste caps, and network flows—together with:

- a **Universal Retail Solver (URS)**: a transparent Gurobi-based reference MILP implementation,
- a **benchmark generator** that produces instances from archetype specifications,
- **prompt templates** and per-instance **SYSTEM/USER prompts** for evaluating LLM agents under a fixed execution contract,
- an evaluation harness for batch runs and result aggregation.

The benchmark is designed to measure **semantic reliability**: whether a generated optimization program reconstructs the intended model structure (not just runnable code).

---

## Key Features

- **Retail-focused structural coverage.** Scenarios systematically stress retail motifs that frequently cause “feasible-but-wrong” formulations.
- **Universal JSON schema.** All instances share a consistent schema to enable controlled comparisons across agents.
- **Solver validation.** Every instance is validated with the reference solver to record feasibility and reference objectives.
- **Reproducible generation.** Instances are generated from explicit archetype definitions and variation rules.
- **LLM-ready prompts.** Prompt templates and per-instance prompt files support standardized text-to-optimization evaluation.

---

## Repository Layout

```

reloop/
├── solvers/
│   └── universal_retail_solver.py       # Universal Retail Solver (reference MILP)
├── tools/
│   ├── retail_benchmark_generator.py    # Generator for 38 archetypes × 5 variations
│   └── generate_prompts.py              # Build per-instance SYSTEM/USER prompt files
├── scenarios/
│   └── retailopt_190/
│       ├── spec/
│       │   ├── retail_spec.md           # Structural + semantic specification
│       │   ├── retail_prompts.md        # Prompt templates for LLM agents
│       │   └── archetypes.yaml          # 38 archetypes (families, cues, flags)
│       ├── data/                        # JSON instances (RetailOpt-190)
│       └── prompts/                     # Per-instance prompt files
└── eval/
├── run_benchmark.py                 # Batch evaluation script
└── benchmark_results.csv            # Reference solver results (URS)

````

---

## Installation

### 1) Create an environment
You need Python 3.9+ and a working Gurobi installation (with a valid license).

```bash
pip install -r requirements.txt
````

If you do not have a `requirements.txt` yet, typical dependencies include:

* `gurobipy`
* `pyyaml`
* `numpy`, `pandas`

> Note: `gurobipy` requires Gurobi installed and licensed on your machine.

---

## Quick Start

### A) Generate the benchmark JSON instances (RetailOpt-190)

This step creates the 190 instances under:
`reloop/scenarios/retailopt_190/data/`

```bash
python reloop/tools/retail_benchmark_generator.py \
  --out_dir reloop/scenarios/retailopt_190/data
```

The generator reads archetypes and configuration from:
`reloop/scenarios/retailopt_190/spec/archetypes.yaml`.

---

### B) Generate per-instance prompts

This step writes:

* a global `system_prompt.txt`
* per-instance `*.user.txt`
* (optional) combined `*.txt` (SYSTEM + USER)

```bash
python reloop/tools/generate_prompts.py \
  --scenario retailopt_190 \
  --write_combined
```

Output directory:
`reloop/scenarios/retailopt_190/prompts/`

---

### C) Run the Universal Retail Solver (URS) on an instance

URS is the reference MILP consistent with the benchmark semantics.
It reads one JSON instance and solves it using Gurobi.

```bash
python reloop/solvers/universal_retail_solver.py \
  --instance reloop/scenarios/retailopt_190/data/<scenario_id>.json
```

It prints the solver status and (when feasible) the objective value.

---

### D) Batch evaluation

`eval/run_benchmark.py` executes a chosen “agent” (e.g., an LLM program generator) against all instances
under the same execution protocol and produces a CSV of outcomes.

```bash
python reloop/eval/run_benchmark.py \
  --scenario retailopt_190 \
  --data_dir reloop/scenarios/retailopt_190/data \
  --prompts_dir reloop/scenarios/retailopt_190/prompts \
  --out_csv reloop/eval/benchmark_results.csv
```

> The exact agent interface depends on your evaluation setup.
> See the “Evaluation Protocol” section below.

---

## Evaluation Protocol (Text-to-Optimization)

RetailOpt-190 evaluates agents under a fixed prompt-and-execute contract:

1. The evaluation harness provides:

   * a **SYSTEM prompt** defining the execution contract and naming conventions,
   * a per-instance **USER prompt** describing the scenario and structure cues.
2. The instance JSON is **not pasted** into the prompt.
   Instead, the harness loads the JSON into a Python variable called `data` at runtime.
3. The agent must return **one Python script** that:

   * imports `gurobipy`,
   * reads sets/parameters from `data` (read-only),
   * builds the implied MILP, solves it, and prints solver status/objective.

This design isolates *schema-conditioned modeling* and prevents copying numeric values from prompt text.

For prompt templates and the full contract, see:

* `reloop/scenarios/retailopt_190/spec/retail_prompts.md`
* `reloop/scenarios/retailopt_190/spec/retail_spec.md`

---

## Data Specification

RetailOpt-190 instances follow a universal schema capturing:

* time periods, products, locations,
* demand signals (e.g., demand curve + location shares),
* capacity limits (storage, production, labor),
* perishability (shelf life), lead times, returns,
* discrete procurement flags (MOQ, pack size, fixed ordering),
* optional budgets and waste caps,
* network edges (substitution arcs, transshipment arcs).

See:

* `reloop/scenarios/retailopt_190/spec/retail_spec.md`

---

## License

* **Code:** MIT
* **Data:** CC BY 4.0

---

## Citation

If you use RetailOpt-190 in your research, please cite:

**RetailOpt-190: A Solver-Validated Retail Supply-Chain Benchmark for Text-to-Optimization**
Junbo Jacob Lian, Northwestern University; Wenzhou Buyi Pharmacy Chain Co., Ltd.
Email: [jacoblian@u.northwestern.edu](mailto:jacoblian@u.northwestern.edu)

```bibtex
@article{lian2025retailopt190,
  title   = {RetailOpt-190: A Solver-Validated Retail Supply-Chain Benchmark for Text-to-Optimization},
  author  = {Lian, Junbo Jacob},
  journal = {arXiv preprint arXiv:XXXX.XXXXX},
  year    = {2026}
}
```

---

