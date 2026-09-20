---
name: mixed-integer-linear-program-formulation
description: Use when assigning strain types to samples from per-locus allele sets
  and proportions (geneX_proportions.csv) against a known MLST strain database, and
  novel-strain detection under proportion constraints is required.
when_to_use_negative:
- The first-stage ADP has not been run and no per-locus allele sets/proportions exist
  for the samples
- The organism has no curated MLST strain-type database, so no known strains Ω or
  edit-distance weighting can be defined
- Only fractional allele composition per locus is needed — solving the Allele Diversity
  Problem ILP (first stage) is the appropriate formulation instead
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3305
- http://edamontology.org/topic_0622
tools:
- name: CPLEX 12.6.3.0
  role: Python API used to formulate and solve both the ADP 0-1 ILP and the SDP MILP
    with gap/time-limit stopping criteria
- name: editdistance
  role: Computes Levenshtein edit distance between candidate novel strains and closest
    known strains to weight novel-strain variables ak
  repo: https://github.com/aflc/editdistance
- name: PyEMD
  role: Used for Earth Mover's Distance calculations in pipeline evaluation (Total
    Variation/EMD metrics against true strain distributions)
  repo: https://github.com/garydoranjr/pyemd
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:4de31a44be4cedec5335bee7f5fe8fd233dc8b9328126700372c4037743f4be1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aflc/editdistance
  tools:
  - CPLEX 12.6.3.0
  - PyEMD
  - Python 2.7
  - editdistance
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3305
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mixed-integer-linear-program-formulation

## Summary

Formulates and solves the Strain Diversity Problem (and Allele Diversity Problem) as a mixed integer linear program that associates each Borrelia burgdorferi MLST sample with a minimal set of strain types, penalizing novel strains by edit distance to known strains while bounding deviations from the observed allele proportions.

## When to use

Use when assigning strain types to samples from per-locus allele sets and proportions (geneX_proportions.csv) against a known MLST strain database, and novel-strain detection under proportion constraints is required.

## When NOT to use

- The first-stage ADP has not been run and no per-locus allele sets/proportions exist for the samples
- The organism has no curated MLST strain-type database, so no known strains Ω or edit-distance weighting can be defined
- Only fractional allele composition per locus is needed — solving the Allele Diversity Problem ILP (first stage) is the appropriate formulation instead

## Inputs

- Per-sample, per-locus allele proportion CSVs (geneX_proportions.csv) from the ADP first stage
- Database of known strain types Ω for the B. burgdorferi MLST scheme (8 housekeeping genes, 1726 alleles, 753 strain types)
- Per-gene edit-distance matrices (provided as the 'editDist' folder via -pathToDistMat)

## Outputs

- One sampleX_strainsAndProportions.csv per sample listing inferred strain types and their proportions (default output folder 'strainsAndProp')

## How to apply

Load the ADP allele distributions per sample and per locus plus the known strain-type database Ω (B. burgdorferi scheme: 8 housekeeping genes, 1726 alleles, 753 strain types). For each sample, enumerate candidate strains as the Cartesian product of selected alleles across the 8 loci and weight each candidate wj = N·min d(s, Sj) using Levenshtein edit distance (aflc/editdistance) to the closest known strain. Build the MILP with binary strain-usage variables ak, proportion variables πih ∈ [0,1], and error variables eijk ∈ [0,ε] with ε = 0.1, minimizing Σ wk·ak (novel strains only) + Σ eijk; constrain allele-proportion deviations by ε, per-sample proportions to sum to 1, ak ≥ πih, the tolerance constraint 0 ≤ ak − (1/|{πih}|)·Σ πih ≤ 1 − δ with δ = 0.001, and linearize absolute errors via eijk ≥ Tijk − pijk and eijk ≥ pijk − Tijk. Solve with the CPLEX 12.6.3.0 Python API, stopping only when both the relative gap (default 5% via -g; the real-data run used 10%) and the time limit (default 600 s via -timelim; real run 24 h) are satisfied; write one sampleX_strainsAndProportions.csv per sample.

## Related tools

- **CPLEX 12.6.3.0** (Python API used to formulate and solve both the ADP 0-1 ILP and the SDP MILP with gap/time-limit stopping criteria)
- **editdistance** (Computes Levenshtein edit distance between candidate novel strains and closest known strains to weight novel-strain variables ak) — https://github.com/aflc/editdistance
- **PyEMD** (Used for Earth Mover's Distance calculations in pipeline evaluation (Total Variation/EMD metrics against true strain distributions)) — https://github.com/garydoranjr/pyemd

## Examples

```
python globalStrainDiversity.py -o strainsAndProp -oc all -timelim 86400 -g 10 -pathToDistMat editDist
```

## Evaluation signals

- Per-sample output file sampleX_strainsAndProportions.csv exists in the -o folder for every sample in the input
- For each sample, the per-locus strain proportions πih sum to 1 and allele-proportion deviations are within ε = 0.1
- Solver terminated only when both the relative gap (-g, e.g. 5–10%) and the time limit (-timelim, e.g. 600 s or 24 h) conditions were satisfied
- On SDP simulations with perfect allele calls, precision and recall (soft, ≤5 SNP differences) of ~0.85–0.91 across EvoMod models and low Total Variation Distance to the true distribution are reproduced
- Novel-strain variables ak for known strains are 0 (only novel strains contribute Σ wk·ak to the objective), consistent with the objective definition

## Limitations

- The SDP is NP-hard (decision version NP-complete via reduction from 3-partition), so the solver may stop at a nonzero optimality gap on large instances; handling of closely related strain clusters relies on ad-hoc post-processing rather than a principled method
- Solver settings trade accuracy for time: default stop is 5% gap / 600 s, while the real-data run required a 10% gap and 24 h
- Requires Python 2.7 and an academic CPLEX license; formulation assumes an 8-locus MLST scheme

## Evidence

- [methods] Both ILPs are formulated and solved using the Python API of IBM's CPLEX 12.6.3.0.: "Both ILPs are formulated and solved using the Python API of IBM’s CPLEX 12.6.3.0."
- [abstract] Minimize number of previously unobserved strains while staying close to observed strains and respecting allele proportions: "Next, we associate to each sample a set of strain types using the alleles and the strain proportions obtained in the first step. We achieve this by using the smallest possible number of previously"
- [methods] SDP decision version NP-complete: "its decision version is proven to be NP-complete in Additional file 1, by a reduction from the 3-partition problem."
- [readme] Solver stopping criteria and output folder: "`-g` specifies the relative gap in % to stop the solver, default is 5%. `-timelim` specifies the maximum time limit in seconds before stopping the solver, default is 600 seconds."
- [readme] Edit distance package: "In computing edit distance, we used the package by https://github.com/aflc/editdistance"
