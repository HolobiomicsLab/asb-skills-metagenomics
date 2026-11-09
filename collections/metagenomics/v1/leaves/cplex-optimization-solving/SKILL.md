---
name: cplex-optimization-solving
description: Use when inferring strain-type mixtures or allele compositions from MLST
  allele distributions requires solving an NP-hard optimization (ADP or SDP) with
  mixed integer linear programming, and when solver gap/time limits must be configured
  to balance accuracy against runtime.
when_to_use_negative:
- When the data are whole genomes rather than MLST allele sequences — the ILP formulations
  are specific to multi-locus allele/proportion structure
- When reference strain types are entirely unavailable, since the SDP objective requires
  weights based on edit distance to known strains
- When a de novo mixture method is preferred and no known allele database exists (e.g.
  DESMAN-style approaches)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3305
tools:
- name: CPLEX 12.6.3.0
  role: MILP/ILP solver used via its Python API to formulate and solve both the Allele
    Diversity Problem and Strain Diversity Problem, with configurable relative gap
    and time limit stopping criteria
  repo: https://github.com/WGS-TB/MLST
- name: Python 2.7
  role: Language in which all pipeline scripts (alleleDiversity.py, globalStrainDiversity.py)
    are written and which hosts the CPLEX Python API calls
  repo: https://github.com/WGS-TB/MLST
- name: editdistance
  role: Computes Levenshtein edit distances between candidate and known strain types
    to weight novel-strain penalties (w_j = N · min d(s, S_j)) in the SDP objective
  repo: https://github.com/aflc/editdistance
- name: PyEMD
  role: Used for Earth Mover's Distance calculations in the evaluation of the strain
    deconvolution results (Total Variation / distributional distance metrics)
  repo: https://github.com/garydoranjr/pyemd
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:77ab3711d22a21db5721f1f6ed8ee1801efe6779042edc208a7b07548f560653
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGS-TB/MLST
  tools:
  - CPLEX 12.6.3.0
  - PyEMD
  - Python 2.7
  - editdistance
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cplex-optimization-solving

## Summary

Formulating and solving the mixed integer linear programs (MILP/ILP) underlying the Allele Diversity Problem and Strain Diversity Problem for MLST-based strain deconvolution, using the Python API of IBM CPLEX 12.6.3.0. It converts combinatorial strain-mixture inference into constrained optimization with binary strain-usage variables, allele proportion variables, and error linearization.

## When to use

Use when inferring strain-type mixtures or allele compositions from MLST allele distributions requires solving an NP-hard optimization (ADP or SDP) with mixed integer linear programming, and when solver gap/time limits must be configured to balance accuracy against runtime.

## When NOT to use

- When the data are whole genomes rather than MLST allele sequences — the ILP formulations are specific to multi-locus allele/proportion structure
- When reference strain types are entirely unavailable, since the SDP objective requires weights based on edit distance to known strains
- When a de novo mixture method is preferred and no known allele database exists (e.g. DESMAN-style approaches)

## Inputs

- geneX_proportions.csv allele distributions (alleles and proportions per sample per locus) from the ADP stage
- Database of known B. burgdorferi MLST strain types Ω (8 housekeeping genes, 1726 alleles, 753 strain types)
- Edit-distance matrices per gene ('editDist' folder) computed with the aflc/editdistance package
- Sample paired-read variant files (sampleX_geneY_paired_reads.txt) extracted from SAM files after Bowtie mapping

## Outputs

- Per-locus allele and proportion files geneX_proportions.csv per sample (ADP solution)
- Per-sample strain type and proportion files sampleX_strainsAndProportions.csv in the output folder (default 'strainsAndProp')
- Selected strain types (novel and known) with their estimated proportions per sample

## How to apply

First formulate the problem as an ILP/MILP with binary usage variables (a_k for strains), continuous proportion variables (π_ih in [0,1] summing to 1 per sample), and error variables (e_ijk in [0, ε] with ε=0.1), linearizing absolute errors via e_ijk ≥ T_ijk − p_ijk and e_ijk ≥ p_ijk − T_ijk. For the SDP, enumerate candidate strain types as the Cartesian product of selected alleles across the 8 housekeeping loci, weighting each candidate by w_j = N · min d(s, S_j) using Levenshtein edit distance (aflc/editdistance package), so that novel strains are penalized in the objective Σ w_k·a_k + Σ e_ijk. Add the tolerance constraint 0 ≤ a_k − (1/|{π_ih}|)·Σ π_ih ≤ 1 − δ with δ=0.001 and bound allele proportion deviations by ε. Build the model with the CPLEX Python API (CPLEX 12.6.3.0) and solve with the relative gap (default 5% via -g; the real-data run used 10%) and time limit (default 600 s via -timelim; real data used 24 h) both satisfied before stopping. Write per-locus results as geneX_proportions.csv (ADP stage) and per-sample sampleX_strainsAndProportions.csv (SDP stage).

## Related tools

- **CPLEX 12.6.3.0** (MILP/ILP solver used via its Python API to formulate and solve both the Allele Diversity Problem and Strain Diversity Problem, with configurable relative gap and time limit stopping criteria) — https://github.com/WGS-TB/MLST
- **Python 2.7** (Language in which all pipeline scripts (alleleDiversity.py, globalStrainDiversity.py) are written and which hosts the CPLEX Python API calls) — https://github.com/WGS-TB/MLST
- **editdistance** (Computes Levenshtein edit distances between candidate and known strain types to weight novel-strain penalties (w_j = N · min d(s, S_j)) in the SDP objective) — https://github.com/aflc/editdistance
- **PyEMD** (Used for Earth Mover's Distance calculations in the evaluation of the strain deconvolution results (Total Variation / distributional distance metrics)) — https://github.com/garydoranjr/pyemd

## Examples

```
python globalStrainDiversity.py -o strainsAndProp -oc all -timelim 600 -g 5 -pathToDistMat editDist
```

## Evaluation signals

- Per-sample output file sampleX_strainsAndProportions.csv is produced for every sample processed by globalStrainDiversity.py
- Proportion variables π_ih satisfy the per-sample sum-to-1 constraint and deviations from allele proportions remain bounded by ε = 0.1
- Solver terminates only when both the relative gap criterion (default 5%) and the time limit (default 600 s) are satisfied
- On simulated SDP data, soft precision/recall (allowing ≤5 SNP differences) reach ~0.85–0.91 across EvoMod models and Total Variation Distance is low
- Applied to the 24 real tick samples, the pipeline infers 60 novel and 10 existing candidate strains, consistent with the published result

## Limitations

- Both the Allele Diversity Problem and Strain Diversity Problem are NP-hard, so large instances require substantial compute (real-data run used 10% gap and 24 h time limit)
- The SDP solver may return solutions only within a relative optimality gap (5% default, 10% for real data) rather than proven optima
- Closely related strain clusters are handled by ad-hoc post-processing rather than a principled method within the MILP itself
- The pipeline requires Python 2.7 and an academic CPLEX license, and depends on packages (editdistance, PyEMD) that only support Python 2

## Evidence

- [methods] Both ILPs are formulated and solved using the Python API of IBM’s CPLEX 12.6.3.0.: "Both ILPs are formulated and solved using the Python API of IBM’s CPLEX 12.6.3.0."
- [methods] The Strain Diversity Problem is NP-hard: "its decision version is proven to be NP-complete in Additional file 1, by a reduction from the 3-partition problem."
- [readme] Solver stopping criteria: "The program will stop if and only if `-g` and `-timelim` are both satisfied."
- [readme] Edit distance implementation used: "In computing edit distance, we used the package by https://github.com/aflc/editdistance"
- [methods] Real-data strain inference result: "60 novel and 10 existing strains were inferred to be potential candidates for explaining the strain diversity in this large sample of ticks."
