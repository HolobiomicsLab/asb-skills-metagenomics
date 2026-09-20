---
name: strain-type-distribution-milp-deconvolution
description: Use when per-locus allele proportions have already been inferred for
  multiple samples and you need a global, multi-sample strain-type decomposition that
  can include strains not present in the MLST database.
when_to_use_negative:
- Input is raw reads rather than already-computed allele proportions — run the Bowtie
  mapping and allele diversity ILP stages first
- No reference MLST strain/allele database is available for the organism, since novel-strain
  weights depend on edit distance to the 753 known strain types
- De novo strain discovery without any known reference strains is required — the paper
  notes de novo methods such as DESMAN serve that setting instead
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_3299
tools:
- name: CPLEX 12.6.3.0
  role: Formulates and solves the Strain Diversity Problem MILP via its Python API,
    with gap and time-limit stopping criteria
  repo: https://github.com/WGS-TB/MLST
- name: GAML
  role: Likelihood-based selection among multiple optimal solutions of the upstream
    allele diversity 0-1 ILP before strain deconvolution
- name: editdistance (aflc)
  role: Computes Levenshtein distances used to weight novel strains by their distance
    to the 753 known strain types
  repo: https://github.com/aflc/editdistance
- name: PyEMD
  role: Earth Mover's Distance calculations used in evaluating distance between allele/strain
    distributions
  repo: https://github.com/garydoranjr/pyemd
- name: Bowtie v0.12.7
  role: Maps sample reads onto the 1726 known alleles, producing the per-locus mapping
    information that feeds the pipeline
  repo: https://github.com/WGS-TB/MLST
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:65e917d27e95b9fb61a9a0f903cb02c3caf5ff324d40ec3d5cbdc8b79356b231
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGS-TB/MLST
  tools:
  - Bowtie v0.12.7
  - CPLEX 12.6.3.0
  - GAML
  - PyEMD
  - Python 2.7
  - editdistance (aflc)
  - fastq-dump
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# strain-type-distribution-milp-deconvolution

## Summary

Solves the Strain Diversity Problem as a mixed integer linear program that maps per-sample allele distributions from the 8-gene B. burgdorferi MLST scheme (1726 alleles, 753 known strain types) onto a minimal set of strain types, allowing novel strains weighted by edit distance to known types, with error bounds ε=0.1 per allele proportion and smallest-proportion tolerance δ=0.001.

## When to use

Use when per-locus allele proportions have already been inferred for multiple samples and you need a global, multi-sample strain-type decomposition that can include strains not present in the MLST database.

## When NOT to use

- Input is raw reads rather than already-computed allele proportions — run the Bowtie mapping and allele diversity ILP stages first
- No reference MLST strain/allele database is available for the organism, since novel-strain weights depend on edit distance to the 753 known strain types
- De novo strain discovery without any known reference strains is required — the paper notes de novo methods such as DESMAN serve that setting instead

## Inputs

- Per-sample per-locus allele proportion CSV files (geneX_proportions.csv) produced by the allele diversity ILP stage
- B. burgdorferi MLST database of 8 housekeeping genes with 1726 known alleles and 753 known strain types
- Edit distance matrices per gene (editDist folder, computed with the aflc/editdistance package)
- MILP parameters: error bound ε=0.1 per allele proportion, smallest-proportion tolerance δ=0.001, relative gap %, time limit in seconds

## Outputs

- sampleX_strainsAndProportions.csv files listing strain types and their proportions for each sample
- Count of inferred novel vs. existing strains (paper reports 60 novel and 10 existing across 24 tick samples)

## How to apply

First run the allele diversity stage per sample (alleleDiversity.py) to obtain geneX_proportions.csv files, then aggregate these allele distributions across all samples. Solve the Strain Diversity Problem MILP with CPLEX 12.6.3.0 (globalStrainDiversity.py), giving novel-strain weights via edit distance to the 753 known strain types, with error bound ε=0.1 per allele proportion and smallest-proportion tolerance δ=0.001. Stop the solver when the relative gap reaches the threshold (-g, in %) or the time limit (-timelim, in seconds) is reached; the paper used a 10% relative gap or 24-hour limit on the 24 real tick samples. The rationale is to minimize the number of selected strains — preferring existing database types over novel ones — while staying consistent with observed allele proportions. Outputs are written as sampleX_strainsAndProportions.csv listing each sample's strain types and their proportions; multiple optimal solutions of the upstream allele ILP are disambiguated with GAML before this stage.

## Related tools

- **CPLEX 12.6.3.0** (Formulates and solves the Strain Diversity Problem MILP via its Python API, with gap and time-limit stopping criteria) — https://github.com/WGS-TB/MLST
- **GAML** (Likelihood-based selection among multiple optimal solutions of the upstream allele diversity 0-1 ILP before strain deconvolution)
- **editdistance (aflc)** (Computes Levenshtein distances used to weight novel strains by their distance to the 753 known strain types) — https://github.com/aflc/editdistance
- **PyEMD** (Earth Mover's Distance calculations used in evaluating distance between allele/strain distributions) — https://github.com/garydoranjr/pyemd
- **Bowtie v0.12.7** (Maps sample reads onto the 1726 known alleles, producing the per-locus mapping information that feeds the pipeline) — https://github.com/WGS-TB/MLST

## Examples

```
python globalStrainDiversity.py -o strainsAndProp -g 10 -timelim 86400 -pathToDistMat editDist
```

## Evaluation signals

- The inferred candidate set for the 24 real tick samples contains 60 novel and 10 existing strains, matching the reported counts
- Solver terminates only when both the relative gap threshold (-g) and the time limit (-timelim) conditions are satisfied
- Each output sampleX_strainsAndProportions.csv contains strain types paired with proportions consistent with the per-locus allele proportions within the ε=0.1 error bound
- No selected strain proportion falls below the smallest-proportion tolerance δ=0.001
- On simulated SDP tests with known strain-type distributions, the MILP recovers the true diversity given perfect allele calls

## Limitations

- Closely related strain clusters are handled by ad-hoc post-processing rather than a principled method; more principled approaches remain unexplored
- Both the Allele Diversity and Strain Diversity Problems are NP-hard, so very large instances may terminate at the gap/time limit without proving optimality
- The solver stop rule requires BOTH the gap and time-limit conditions (e.g. gap 5% and 600s default), which can force long runtimes if the gap remains above threshold

## Evidence

- [methods] 60 novel and 10 existing strains were inferred to be potential candidates: "60 novel and 10 existing strains were inferred to be potential candidates for explaining the strain diversity in this large sample of ticks."
- [methods] Both ILPs are formulated and solved using the Python API of IBM's CPLEX: [evidence span withheld — anchor status: not_found]
- [methods] multiple optimal solutions resolved with GAML: "If the above 0-1 ILP has multiple optimal solutions, we resort to a likelihood based method to select one, namely GAML [17]"
- [readme] the csv file contains the strains and their proportions identified in sampleX: "The folder contains a list of `sampleX_strainsAndProportions.csv` with sampleX corresponds a particular sample, and the csv file contains the strains and their proportions identified in sampleX."
- [readme] -g specifies the relative gap in % to stop the solver, default is 5%: "`-g` specifies the relative gap in % to stop the solver, default is 5%. `-timelim` specifies the maximum time limit in seconds before stopping the solver, default is 600 seconds."
- [discussion] ad-hoc post-processing of closely related strain clusters: "At the moment, our approach to this problem is to post-process the result of our pipeline to identify clusters of closely related strains, but other more principled approaches should be explored"
