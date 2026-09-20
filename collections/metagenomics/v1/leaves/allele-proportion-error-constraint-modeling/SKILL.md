---
name: allele-proportion-error-constraint-modeling
description: Use when reconstructing a multi-sample strain decomposition from per-locus
  allele distributions (geneX_proportions.csv) where you must minimize previously
  unobserved strains while respecting allele proportions within a tolerance.
when_to_use_negative:
- Input is raw reads rather than the allele/proportion output of the first-stage allele
  diversity ILP — the read mapping and allele calling must happen first
- The goal is only fractional allele composition per locus (the Allele Diversity Problem),
  which uses a separate 0-1 ILP with GAML post-processing instead of this SDP MILP
tools:
- name: CPLEX 12.6.3.0
  role: Python API used to formulate and solve the SDP MILP with the error/proportion
    constraint structure
- name: editdistance (aflc/editdistance)
  role: Computes Levenshtein edit distance used to weight novel candidate strains
    by their distance to the closest known strain
  repo: https://github.com/aflc/editdistance
- name: WGS-TB/MLST BorreliaPipeline
  role: Provides globalStrainDiversity.py, the script implementing this MILP and its
    CLI parameters
  repo: https://github.com/WGS-TB/MLST
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:2433a5c383487a6b5fa731daed6a2869775c898bf37694e8b2112b586bbd09d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aflc/editdistance
  tools:
  - CPLEX 12.6.3.0
  - Python 2.7
  - WGS-TB/MLST BorreliaPipeline
  - editdistance
  - editdistance (aflc/editdistance)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# allele-proportion-error-constraint-modeling

## Summary

Formulates the strain-type assignment MILP in which binary strain-usage variables, per-sample allele proportion variables, and bounded error variables jointly minimize novel-strain weights plus deviations from observed allele proportions. This is the second stage of the MLST strain-deconvolution framework for B. burgdorferi.

## When to use

Use when reconstructing a multi-sample strain decomposition from per-locus allele distributions (geneX_proportions.csv) where you must minimize previously unobserved strains while respecting allele proportions within a tolerance.

## When NOT to use

- Input is raw reads rather than the allele/proportion output of the first-stage allele diversity ILP — the read mapping and allele calling must happen first
- The goal is only fractional allele composition per locus (the Allele Diversity Problem), which uses a separate 0-1 ILP with GAML post-processing instead of this SDP MILP

## Inputs

- Per-sample, per-locus allele proportion files (geneX_proportions.csv) produced by the first-stage ADP ILP
- Database of known B. burgdorferi MLST strain types Ω (8 housekeeping genes, 1726 alleles, 753 strain types)
- Per-gene edit-distance matrices (provided in the 'editDist' folder)

## Outputs

- One sampleX_strainsAndProportions.csv per sample in the output folder (default 'strainsAndProp'), containing inferred strain types and their proportions

## How to apply

Load the ADP allele distributions produced per sample and per locus (geneX_proportions.csv) plus the database of known B. burgdorferi strain types (8 housekeeping genes, 753 strain types). For each sample, enumerate candidate strain types as the Cartesian product of selected alleles across the 8 loci and weight each candidate novel strain wj = N · min d(s, Sj) using Levenshtein edit distance (aflc/editdistance) to the closest known strain. Build the MILP with binary strain-usage variables ak, proportion variables πih in [0,1], and error variables eijk in [0,ε] with ε=0.1; the objective is Σ wk·ak (novel strains only) plus Σ eijk. Add the constraints that allele proportion deviations are bounded by ε, per-sample proportions sum to 1, ak ≥ πih, the tolerance constraint 0 ≤ ak − (1/|{πih}|)·Σ πih ≤ 1 − δ with δ=0.001, and the error linearization eijk ≥ Tijk − pijk and eijk ≥ pijk − Tijk. Solve with the CPLEX 12.6.3.0 Python API, stopping only when both the relative gap (default 5% via -g; the real-data run used 10% gap and 24 h) and the time limit (default 600 s via -timelim) are satisfied.

## Related tools

- **CPLEX 12.6.3.0** (Python API used to formulate and solve the SDP MILP with the error/proportion constraint structure)
- **editdistance (aflc/editdistance)** (Computes Levenshtein edit distance used to weight novel candidate strains by their distance to the closest known strain) — https://github.com/aflc/editdistance
- **WGS-TB/MLST BorreliaPipeline** (Provides globalStrainDiversity.py, the script implementing this MILP and its CLI parameters) — https://github.com/WGS-TB/MLST

## Examples

```
python globalStrainDiversity.py -o strainsAndProp -g 5 -timelim 600 -pathToDistMat editDist
```

## Evaluation signals

- Output folder contains one sampleX_strainsAndProportions.csv per input sample with strain types and proportions summing to 1
- Error variables stay within the bound eijk ∈ [0, 0.1]; the solver stops only when both the relative gap (-g) and time limit (-timelim) conditions are met
- On simulated SDP data, soft precision/recall (≤5 SNP tolerance) of ~0.85–0.91 across EvoMod models and low Total Variation Distance against the true strain distribution
- On the 24-tick real dataset, the run recovers 60 novel and 10 existing strains as candidates explaining sample diversity

## Limitations

- The real-data run required a relaxed setting (10% gap, 24 h time limit) versus the defaults (5%, 600 s), so convergence can be very slow on large problems
- Closely related strain clusters are handled only by ad-hoc post-processing; more principled clustering approaches remain unexplored
- The Strain Diversity Problem is NP-hard, so the MILP is solved heuristically within a gap tolerance rather than to proven optimality

## Evidence

- [methods] Both ILPs are formulated and solved using the Python API of IBM's CPLEX 12.6.3.0.: "Both ILPs are formulated and solved using the Python API of IBM’s CPLEX 12.6.3.0."
- [methods] We now describe an MILP that solves the SDP.: "We now describe an MILP that solves the SDP."
- [methods] In computing edit distance, we used the package by https://github.com/aflc/editdistance: "In computing edit distance, we used the package by https://github.com/aflc/editdistance"
- [readme] -g specifies the relative gap in % to stop the solver, default is 5%. -timelim specifies the maximum time limit in seconds before stopping the solver, default is 600 seconds.: "`-g` specifies the relative gap in % to stop the solver, default is 5%. `-timelim` specifies the maximum time limit in seconds before stopping the solver, default is 600 seconds."
- [readme] The program will stop if and only if -g and -timelim are both satisfied.: "The program will stop if and only if `-g` and `-timelim` are both satisfied."
