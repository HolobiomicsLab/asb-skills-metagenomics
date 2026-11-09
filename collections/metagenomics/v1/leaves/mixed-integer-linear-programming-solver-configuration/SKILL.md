---
name: mixed-integer-linear-programming-solver-configuration
description: Use when deconvolving mixed MLST samples into strains via the paper's
  ILP/MILP formulations, or when configuring CPLEX runs whose NP-hard models need
  gap/time-limit stopping criteria and objective variants.
when_to_use_negative:
- The problem is a simple linear program or has no combinatorial integer decisions
  — a MILP/CPLEX setup adds complexity without benefit
- 'You need guaranteed global optimality in fixed time: the NP-hard SDP is stopped
  by relative gap (5%) and time limit (600 s), so exactness is not assured'
- Allele calls are not yet established — this skill assumes Stage-1 ADP outputs (geneX_proportions.csv)
  already exist; run alleleDiversity.py first
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: CPLEX 12.6.3.0
  role: Solves both the 0-1 ADP ILP and the SDP MILP through its Python API; requires
    academic license and CPLEX-Python module installation
- name: globalStrainDiversity.py / alleleDiversity.py (WGS-TB/MLST pipeline)
  role: Wrap CPLEX invocation, expose gap/time-limit/objective-component/output-folder
    arguments, and write per-sample strain-and-proportion CSVs
  repo: https://github.com/WGS-TB/MLST
- name: editdistance (aflc)
  role: Computes Levenshtein distance weighting of novel candidate strains against
    the 753 known strain types
  repo: https://github.com/aflc/editdistance
- name: GAML
  role: Likelihood-based selection among multiple optimal 0-1 ILP solutions (ADP post-processing)
- name: PyEMD
  role: Earth-Mover's distance used to evaluate recovered strain distributions against
    simulated ground truth
  repo: https://github.com/garydoranjr/pyemd
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:b7cf5e73cd63303589217b37da3c85ecb8f94adb40045269a987962c78bfaebf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGS-TB/MLST
  tools:
  - CPLEX 12.6.3.0
  - GAML
  - PyEMD
  - Python 2.7
  - editdistance
  - editdistance (aflc)
  - globalStrainDiversity.py / alleleDiversity.py (WGS-TB/MLST pipeline)
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# mixed-integer-linear-programming-solver-configuration

## Summary

Configures and runs IBM CPLEX 12.6.3.0 (Python 2.7 API) to solve the Allele Diversity Problem (0-1 ILP) and Strain Diversity Problem (MILP) for within-host B. burgdorferi strain deconvolution, with explicit stopping criteria (relative gap, time limit) and objective-function components.

## When to use

Use when deconvolving mixed MLST samples into strains via the paper's ILP/MILP formulations, or when configuring CPLEX runs whose NP-hard models need gap/time-limit stopping criteria and objective variants.

## When NOT to use

- The problem is a simple linear program or has no combinatorial integer decisions — a MILP/CPLEX setup adds complexity without benefit
- You need guaranteed global optimality in fixed time: the NP-hard SDP is stopped by relative gap (5%) and time limit (600 s), so exactness is not assured
- Allele calls are not yet established — this skill assumes Stage-1 ADP outputs (geneX_proportions.csv) already exist; run alleleDiversity.py first

## Inputs

- per-sample allele/allele-proportion inputs: sampleX_geneY_paired_reads.txt and geneX_proportions.csv (8 loci) produced by the ADP stage
- candidate strain type set built as the Cartesian product of alleles called per locus
- 'editDist' folder of edit-distance matrices, one per gene, over the 1726 known alleles / 753 strain types
- solver parameters: relative gap (default 5%), time limit (default 600 s), objective-component flag, output folder name

## Outputs

- sampleX_strainsAndProportions.csv per sample: inferred strain types and their proportions
- screened/logged intermediate solver output for tracking
- for simulation studies: precision, recall, Total Variation Distance, soft-precision/soft-recall (TP within 5 SNPs), and Earth-Mover's distance per evolutionary mechanism

## How to apply

Formulate the ADP as a 0-1 ILP per locus (allele selection at each of the 8 housekeeping genes) and the SDP as a MILP minimizing summed edit-distance weights of novel strains plus total proportion errors; both are solved with the Python API of IBM's CPLEX 12.6.3.0. For the SDP, build candidate strain types as the Cartesian product of alleles called per locus and weight each candidate by N times its minimum edit distance (aflc/editdistance) to the closest of the 753 known strain types, using the provided 'editDist' matrices; use error bound ε=0.1 and tolerance δ=0.001. Run the solver via globalStrainDiversity.py with '-g' (relative gap %, default 5) and '-timelim' (seconds, default 600), noting the program stops if and only if BOTH criteria are satisfied; '-oc' switches between the full objective ('all') and one omitting proportion/error components ('noPropAndErr'). If the 0-1 ADP ILP has multiple optimal solutions, select one with GAML, a likelihood-based method, before computing allele proportions.

## Related tools

- **CPLEX 12.6.3.0** (Solves both the 0-1 ADP ILP and the SDP MILP through its Python API; requires academic license and CPLEX-Python module installation)
- **globalStrainDiversity.py / alleleDiversity.py (WGS-TB/MLST pipeline)** (Wrap CPLEX invocation, expose gap/time-limit/objective-component/output-folder arguments, and write per-sample strain-and-proportion CSVs) — https://github.com/WGS-TB/MLST
- **editdistance (aflc)** (Computes Levenshtein distance weighting of novel candidate strains against the 753 known strain types) — https://github.com/aflc/editdistance
- **GAML** (Likelihood-based selection among multiple optimal 0-1 ILP solutions (ADP post-processing))
- **PyEMD** (Earth-Mover's distance used to evaluate recovered strain distributions against simulated ground truth) — https://github.com/garydoranjr/pyemd

## Examples

```
python globalStrainDiversity.py -o strainsAndProp -oc all -g 5 -timelim 600 -pathToDistMat editDist
```

## Evaluation signals

- The solver terminates only when both the relative gap (-g, default 5%) and the time limit (-timelim, default 600 s) are satisfied — check logs for gap attainment
- On SDP simulations with perfect allele calls, reproduced precision/recall should match Table 2's SDP block (e.g., EvoMod1 precision 0.85±0.28, EvoMod3 precision 0.76±0.21, TVD 0.22±0.24)
- The paper's qualitative criterion: SDP maintains precision and recall of almost 75% even for the recombination model (EvoMod3) given perfect allele calls
- Output folder contains one sampleX_strainsAndProportions.csv per sample, each with strain types whose proportions are consistent with the Stage-1 geneX_proportions.csv allele fractions
- Novel (previously unobserved) strains should be minimized in the objective: known-strain candidates carry weight 0, novel candidates are weighted by N × minimum edit distance

## Limitations

- Both the ADP and SDP are NP-hard (reductions from 3-dimensional matching and 3-partition), so the 5% relative-gap / 600-second stopping rule yields good-accuracy solutions rather than guaranteed optima
- Multiple optimal 0-1 ILP solutions require GAML post-processing; closely related strain clusters are handled by ad-hoc post-processing rather than a principled method
- Objective formulation choice ('all' vs 'noPropAndErr') changes results; and the solution quality depends on the quality of Stage-1 allele calls — imperfect allele calls degrade SDP recovery

## Evidence

- [methods] Solver formulation and version: "Both ILPs are formulated and solved using the Python API of IBM’s CPLEX 12.6.3.0."
- [abstract] MILP framework: "We solve both problems using mixed integer linear programming (MILP)."
- [methods] GAML post-processing of ties: "If the above 0-1 ILP has multiple optimal solutions, we resort to a likelihood based method to select one, namely GAML [17]"
- [readme] Stopping criteria and defaults: "`-g` specifies the relative gap in % to stop the solver, default is 5%. `-timelim` specifies the maximum time limit in seconds before stopping the solver, default is 600 seconds."
- [readme] Joint-satisfaction stopping condition: "The program will stop if and only if `-g` and `-timelim` are both satisfied."
- [readme] Objective-component switch: "The `-oc` argument specifies different formulation of the objective function, 'all' includes all components of the objective function, 'noPropAndErr' omits the proportion and error component."
