---
name: mlst-strain-diversity-deconvolution
description: Use when per-locus allele sets and allele proportions are available for
  MLST samples and you must infer which known/novel strain types explain the observed
  within-host diversity.
when_to_use_negative:
- You only have raw reads and have not yet run Stage 1 (allele calling and allele
  proportion computation per locus); the SDP requires per-locus allele proportions
  as input.
- The pathogen has no MLST database of known strain types/alleles to compare against;
  the method relies on reference strain types and edit-distance weighting.
- You need per-locus allele identification rather than whole-sample strain composition
  — the Allele Diversity Problem (alleleDiversity.py) addresses that stage instead.
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3299
tools:
- name: CPLEX 12.6.3.0
  role: MILP solver used via its Python API to formulate and solve the Strain Diversity
    Problem
- name: globalStrainDiversity.py
  role: Pipeline script that formulates the SDP MILP and writes per-sample strain/proportion
    CSV outputs
  repo: https://github.com/WGS-TB/MLST
- name: editdistance (aflc/editdistance)
  role: Computes Levenshtein edit distances used to weight candidate novel strains
    against the 753 known strain types
  repo: https://github.com/aflc/editdistance
- name: PyEMD (garydoranjr/pyemd)
  role: Earth-Mover's distance used to evaluate accuracy of inferred strain distributions
    against ground truth
  repo: https://github.com/garydoranjr/pyemd
- name: Bowtie v0.12.7
  role: Upstream read mapper aligning sample reads to the known allele database, feeding
    Stage 1 inputs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:9e681ee6cd287ded6b6727963be15e24f91ed6ef7d2470c9c41e294fe16fe0f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGS-TB/MLST
  tools:
  - Bowtie v0.12.7
  - CPLEX 12.6.3.0
  - PyEMD
  - PyEMD (garydoranjr/pyemd)
  - Python 2.7
  - editdistance
  - editdistance (aflc/editdistance)
  - globalStrainDiversity.py
  - pandas
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# mlst-strain-diversity-deconvolution

## Summary

Solves the Strain Diversity Problem (SDP) — associating each MLST sample with a set of strain types and proportions from per-locus allele calls via mixed integer linear programming — minimizing novel strains while respecting observed allele proportions.

## When to use

Use when per-locus allele sets and allele proportions are available for MLST samples and you must infer which known/novel strain types explain the observed within-host diversity.

## When NOT to use

- You only have raw reads and have not yet run Stage 1 (allele calling and allele proportion computation per locus); the SDP requires per-locus allele proportions as input.
- The pathogen has no MLST database of known strain types/alleles to compare against; the method relies on reference strain types and edit-distance weighting.
- You need per-locus allele identification rather than whole-sample strain composition — the Allele Diversity Problem (alleleDiversity.py) addresses that stage instead.

## Inputs

- Per-sample per-locus allele calls and allele proportions (geneX_proportions.csv files produced by the allele diversity stage)
- edit distance matrices for each gene (folder named 'editDist', computed with the aflc/editdistance package)
- B. burgdorferi MLST scheme reference (8 housekeeping genes, 1726 known alleles, 753 strain types)

## Outputs

- sampleX_strainsAndProportions.csv per sample, listing inferred strain types and their proportions
- Solver intermediate output for tracking progress

## How to apply

Take as input the per-locus allele proportions from Stage 1 (Allele Diversity Problem solved with alleleDiversity.py), build the candidate strain type set as the Cartesian product of called alleles per locus, and weight each candidate by N times its minimum edit distance to the closest of the 753 known B. burgdorferi strain types (known strains get weight 0) using the provided 'editDist' distance matrices. Then run globalStrainDiversity.py, which formulates and solves the SDP as an MILP with the Python API of IBM CPLEX 12.6.3.0, using an objective that combines summed weights of novel strains with total proportion errors; a flag (-oc) can switch to a formulation omitting the proportion-and-error component. Control solver termination with a relative gap (-g, default 5%) and a time limit (-timelim, default 600 seconds); the program stops only when both are satisfied. The method is designed to minimize the number of previously unobserved strains while staying close to observed strains and respecting allele proportions.

## Related tools

- **CPLEX 12.6.3.0** (MILP solver used via its Python API to formulate and solve the Strain Diversity Problem)
- **globalStrainDiversity.py** (Pipeline script that formulates the SDP MILP and writes per-sample strain/proportion CSV outputs) — https://github.com/WGS-TB/MLST
- **editdistance (aflc/editdistance)** (Computes Levenshtein edit distances used to weight candidate novel strains against the 753 known strain types) — https://github.com/aflc/editdistance
- **PyEMD (garydoranjr/pyemd)** (Earth-Mover's distance used to evaluate accuracy of inferred strain distributions against ground truth) — https://github.com/garydoranjr/pyemd
- **Bowtie v0.12.7** (Upstream read mapper aligning sample reads to the known allele database, feeding Stage 1 inputs)

## Examples

```
python globalStrainDiversity.py -o strainsAndProp -oc all -g 5 -timelim 600 -pathToDistMat editDist
```

## Evaluation signals

- Output folder contains a sampleX_strainsAndProportions.csv for every sample in the input variantsAndProp/variantsAndProp directories.
- On simulated samples with perfect allele calls, reproduced SDP statistics should match Table 2's SDP simulation block (e.g., EvoMod1 precision 0.85±0.28, EvoMod3 precision 0.76±0.21, TVD 0.22±0.24).
- Paper's qualitative criterion: with perfect allele calls the SDP maintains precision and recall of almost 75% even for the recombination model (EvoMod3).
- Summed weights of inferred novel strains should be low — known strains receive weight 0, so a solution dominated by high-distance novel candidates signals a misconfiguration of the editDist matrices.
- Solver terminates only when both the relative gap (-g, default 5%) and the time limit (-timelim, default 600s) conditions are met; verify reported gap in intermediate output.

## Limitations

- Closely related strain clusters are handled by ad-hoc post-processing rather than a principled method; more principled approaches remain unexplored.
- The SDP decision version is NP-complete (reduction from the 3-partition problem), so solver runs may need gap/time-limit termination rather than exact optima.
- The stop condition is the conjunction of gap and time limit: if the relative gap is above the threshold after the time limit elapses, the program continues until the gap is reached, so runs can exceed the nominal time limit.

## Evidence

- [abstract] Next, we associate to each sample a set of strain types using the alleles and the strain proportions obtained in the first step.: "Next, we associate to each sample a set of strain types using the alleles and the strain proportions obtained in the first step."
- [methods] Both ILPs are formulated and solved using the Python API of IBM's CPLEX 12.6.3.0.: [evidence span withheld — anchor status: not_found]
- [methods] For this simulation we selected random strain type distributions and tested the ability of our SDP method to recover the true diversity given perfect allele calls.: "For this simulation we selected random strain type distributions and tested the ability of our SDP method to recover the true diversity given perfect allele calls."
- [readme] The -g specifies the relative gap in % to stop the solver, default is 5%.: "The `-g` specifies the relative gap in % to stop the solver, default is 5%. `-timelim` specifies the maximum time limit in seconds before stopping the solver, default is 600 seconds."
- [discussion] At the moment, our approach to this problem is to post-process the result of our pipeline to identify clusters of closely related strains.: "At the moment, our approach to this problem is to post-process the result of our pipeline to identify clusters of closely related strains, but other more principled approaches should be explored"
