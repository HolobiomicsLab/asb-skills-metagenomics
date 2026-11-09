---
name: mixed-integer-program-solution-post-processing
description: Use when an ILP/MILP (e.g. the Allele Diversity Problem solved with CPLEX)
  returns multiple optimal solutions and you must (a) pick one biologically plausible
  solution and (b) derive fractional allele abundances from the selected solution.
when_to_use_negative:
- The ILP has a unique optimal solution — GAML-based tie-breaking is unnecessary and
  proportions can be computed directly from that solution
- The downstream goal is the Stage-2 Strain Diversity Problem rather than per-locus
  allele calling; that stage is solved by a separate MILP (globalStrainDiversity.py)
- Input data are de novo assemblies without a reference allele database — this post-processing
  relies on known MLST alleles and read-allele mapping scores
tools:
- name: CPLEX 12.6.3.0
  role: Python API used to formulate and solve the 0-1 ILP whose multiple optima are
    post-processed
  repo: https://github.com/WGS-TB/MLST
- name: GAML
  role: Likelihood-based probabilistic genome assembly model used to select one solution
    among multiple ILP optima
  repo: https://github.com/WGS-TB/MLST
- name: Python 2.7
  role: Language in which all pipeline scripts, including alleleDiversity.py, are
    written
  repo: https://github.com/WGS-TB/MLST
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:8a189442d121e7e163004b314517c9e3748e34ef8bcee76be2bb89c0565b50b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGS-TB/MLST
  tools:
  - Bowtie v0.12.7
  - CPLEX 12.6.3.0
  - GAML
  - Python 2.7
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mixed-integer-program-solution-post-processing

## Summary

Resolves the degeneracy of a 0-1 ILP solution by selecting a single optimal solution with the likelihood-based assembler GAML, then converts the selected allele set into per-locus allele proportions by weighting reads across the h alleles attaining the chosen optimal mapping score.

## When to use

Use when an ILP/MILP (e.g. the Allele Diversity Problem solved with CPLEX) returns multiple optimal solutions and you must (a) pick one biologically plausible solution and (b) derive fractional allele abundances from the selected solution.

## When NOT to use

- The ILP has a unique optimal solution — GAML-based tie-breaking is unnecessary and proportions can be computed directly from that solution
- The downstream goal is the Stage-2 Strain Diversity Problem rather than per-locus allele calling; that stage is solved by a separate MILP (globalStrainDiversity.py)
- Input data are de novo assemblies without a reference allele database — this post-processing relies on known MLST alleles and read-allele mapping scores

## Inputs

- Multiple optimal solutions of the 0-1 ILP over binary variables xj (allele aj selected) and yik (read ri explained via mapping score qik), built from sampleX_geneY_paired_reads.txt files in the variantsAndProp folder
- Per-locus read-allele incidence matrix M with normalized Phred-score-based mismatch weights (non-mapping reads set to infinity)

## Outputs

- A single GAML-selected optimal allele set per locus
- Per-locus normalized allele proportions written to 8 files named geneX_proportions.csv per sample in the sample's variantsAndProp folder

## How to apply

After solving the 0-1 ILP for the Allele Diversity Problem with the CPLEX 12.6.3.0 Python API, check whether multiple optimal solutions exist. If so, use GAML, a probabilistic model for genome assembly, as a likelihood-based method to select one of the optima. Then compute allele proportions: discard reads that map equally well onto all selected alleles, and for each retained read add weight 1/h to each of the h alleles attaining the chosen optimal mapping score, then normalize the allele weights to sum to 1. Write the alleles and their normalized proportions for each of the 8 B. burgdorferi MLST loci to geneX_proportions.csv in the sample's variantsAndProp folder.

## Related tools

- **CPLEX 12.6.3.0** (Python API used to formulate and solve the 0-1 ILP whose multiple optima are post-processed) — https://github.com/WGS-TB/MLST
- **GAML** (Likelihood-based probabilistic genome assembly model used to select one solution among multiple ILP optima) — https://github.com/WGS-TB/MLST
- **Python 2.7** (Language in which all pipeline scripts, including alleleDiversity.py, are written) — https://github.com/WGS-TB/MLST

## Examples

```
python alleleDiversity.py -s SRR2034336
```

## Evaluation signals

- Running alleleDiversity.py on a sample produces exactly 8 geneX_proportions.csv files (one per MLST locus) in that sample's variantsAndProp folder
- Allele proportions in each geneX_proportions.csv are normalized (sum to 1) and contain only alleles from the selected ILP optimum
- ILP size and resources remain feasible: paper reports ~20,112 variables per gene on average and ~1.5GB peak RAM (reference sample SRR2034336, ~33 min for all 8 genes on a 4-CPU machine)
- On simulated data, average B-MLST precision is at the reported level of 0.98 +/- 0.011
- Intermediate tracking output is printed to screen during the run, as documented for alleleDiversity.py

## Limitations

- GAML tie-breaking is only triggered when the 0-1 ILP has multiple optimal solutions; a unique optimum bypasses this step
- Reads mapping equally well onto all selected alleles are discarded from proportion estimation, which reduces the effective read support per locus
- Closely related strain clusters are handled by ad-hoc post-processing rather than a principled method, and supplementary details are only in Additional file 1

## Evidence

- [methods] GAML selection among ILP optima: "If the above 0-1 ILP has multiple optimal solutions, we resort to a likelihood based method to select one, namely GAML [17]"
- [methods] Allele proportion computation: "once the alleles have been identified for a given locus, we compute the proportion of each allele."
- [methods] CPLEX solving of both ILPs: "Both ILPs are formulated and solved using the Python API of IBM’s CPLEX 12.6.3.0."
- [readme] Output files of the allele diversity script: "create 8 CSV files (for each locus) in each of the sample folder in `variantsAndProp`, named as `geneX_proportions.csv`"
- [methods] GAML described as probabilistic assembly model: "GAML [17], a probabilistic model for genome assembly."
