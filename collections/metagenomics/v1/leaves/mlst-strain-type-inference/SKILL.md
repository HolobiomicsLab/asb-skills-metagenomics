---
name: mlst-strain-type-inference
description: Use when per-sample MLST allele sets and allele proportions (geneX_proportions.csv)
  are available and you need to jointly infer which known and novel strain types,
  with proportions, explain the samples.
when_to_use_negative:
- Input is raw reads or unmapped sequencing data — the first-stage ADP solver (alleleDiversity.py)
  must have already produced geneX_proportions.csv allele sets and proportions
- No reference strain-type/allele database exists for the organism (the MILP is reference-based,
  like known-strain approaches rather than de novo methods such as DESMAN)
- A quick single-locus allele composition estimate is all that is needed — use the
  ADP ILP stage instead of the global strain-type MILP
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: CPLEX 12.6.3.0
  role: MILP solver used via its Python API to formulate and solve the Strain Diversity
    Problem ILP/MILP
- name: editdistance (aflc/editdistance)
  role: Computes Levenshtein edit distance between candidate novel strains and the
    closest known strains to weight novel-strain usage variables
  repo: github.com/aflc/editdistance
- name: WGS-TB/MLST pipeline (globalStrainDiversity.py)
  role: Python 2.7 script that loads allele proportions, builds the SDP MILP, runs
    CPLEX with gap/time-limit stopping rules, and writes strainsAndProp outputs
  repo: https://github.com/WGS-TB/MLST
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:613d9d8e6117591cec443492137ed27ca1355fd0e425a243cabbadf6567ae324
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CPLEX 12.6.3.0
  - Python 2.7
  - WGS-TB/MLST pipeline (globalStrainDiversity.py)
  - editdistance
  - editdistance (aflc/editdistance)
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# mlst-strain-type-inference

## Summary

Solves the Strain Diversity Problem (SDP) by formulating and solving a mixed integer linear program (MILP) that assigns each sample a set of strain types from per-sample allele sets and allele proportions, minimizing previously unobserved strains while respecting allele proportions. Implemented in Python 2.7 with the CPLEX 12.6.3.0 Python API and editdistance for weighting novel strains.

## When to use

Use when per-sample MLST allele sets and allele proportions (geneX_proportions.csv) are available and you need to jointly infer which known and novel strain types, with proportions, explain the samples.

## When NOT to use

- Input is raw reads or unmapped sequencing data — the first-stage ADP solver (alleleDiversity.py) must have already produced geneX_proportions.csv allele sets and proportions
- No reference strain-type/allele database exists for the organism (the MILP is reference-based, like known-strain approaches rather than de novo methods such as DESMAN)
- A quick single-locus allele composition estimate is all that is needed — use the ADP ILP stage instead of the global strain-type MILP

## Inputs

- Per-sample, per-locus allele distribution CSVs (geneX_proportions.csv) produced by the first-stage ADP solver
- Database of known strain types Ω (B. burgdorferi MLST scheme: 8 housekeeping genes, 1726 alleles, 753 strain types)
- Edit distance matrices per gene (editDist folder, computed with aflc/editdistance)

## Outputs

- One sampleX_strainsAndProportions.csv per sample in the output folder (default 'strainsAndProp'), listing the inferred strain types and their proportions
- MILP solution log with intermediate solver progress (gap, time)

## How to apply

Load the ADP allele distributions (geneX_proportions.csv per sample and locus) plus the database of known strain types Ω (e.g. the B. burgdorferi MLST scheme of 8 housekeeping genes, 1726 alleles, 753 strain types). For each sample, enumerate candidate strain types as the Cartesian product of selected alleles across the 8 loci and weight each novel candidate by wj = N · min d(s, Sj), using Levenshtein edit distance (aflc/editdistance) to the closest known strain — this prefers unobserved strains close to observed ones. Build the MILP with binary strain-usage variables ak, proportion variables πih in [0,1], and error variables eijk in [0,ε] with ε=0.1; the objective is Σ wk·ak (novel strains only) + Σ eijk, with constraints bounding allele proportion deviations by ε, forcing per-sample proportions to sum to 1, ak ≥ πih, the tolerance constraint 0 ≤ ak − (1/|{πih}|)·Σ πih ≤ 1 − δ with δ=0.001, and error linearization eijk ≥ |Tijk − pijk|. Solve with the CPLEX 12.6.3.0 Python API, stopping only when both the relative gap (-g, default 5%) and time limit (-timelim, default 600 s; the real-data run used 10% gap and 24 h) are satisfied, then write sampleX_strainsAndProportions.csv files. The 'all' objective formulation includes proportion and error components; 'noPropAndErr' omits them.

## Related tools

- **CPLEX 12.6.3.0** (MILP solver used via its Python API to formulate and solve the Strain Diversity Problem ILP/MILP)
- **editdistance (aflc/editdistance)** (Computes Levenshtein edit distance between candidate novel strains and the closest known strains to weight novel-strain usage variables) — github.com/aflc/editdistance
- **WGS-TB/MLST pipeline (globalStrainDiversity.py)** (Python 2.7 script that loads allele proportions, builds the SDP MILP, runs CPLEX with gap/time-limit stopping rules, and writes strainsAndProp outputs) — https://github.com/WGS-TB/MLST

## Examples

```
python globalStrainDiversity.py -o strainsAndProp -oc all -g 10 -timelim 86400 -pathToDistMat editDist
```

## Evaluation signals

- Precision and recall against simulated true strain distributions under soft matching (allowing ≤5 SNP differences), with SDP simulation achieving ~0.85–0.91 across EvoMod models
- Total Variation Distance between inferred and true strain proportions in simulation studies
- Output folder contains exactly one sampleX_strainsAndProportions.csv per sample, each with proportions summing to 1
- Solver terminates only when both the relative gap (-g) and time limit (-timelim) conditions are satisfied
- On the 24-sample real tick dataset, the solution should recover roughly the same scale of diversity (60 novel and 10 existing strains reported in the paper)

## Limitations

- MILP solving may be expensive: the real-data run required a 10% relative gap and a 24-hour time limit despite the default 5%/600 s settings
- Closely related strain clusters are handled by ad-hoc post-processing of the pipeline output rather than a principled method
- Formulated for MLST frameworks with a known allele/strain-type database (developed and evaluated on B. burgdorferi); the SDP decision version is NP-complete
- Pipeline depends on Python 2.7-era tooling (CPLEX 12.6.3.0, pyemd requires Python 2)

## Evidence

- [methods] Both ILPs are formulated and solved using the Python API of IBM’s CPLEX 12.6.3.0.: "Both ILPs are formulated and solved using the Python API of IBM’s CPLEX 12.6.3.0."
- [abstract] Next, we associate to each sample a set of strain types using the alleles and the strain proportions obtained in the first step.: "Next, we associate to each sample a set of strain types using the alleles and the strain proportions obtained in the first step. We achieve this by using the smallest possible number of previously"
- [methods] 60 novel and 10 existing strains were inferred to be potential candidates for explaining the strain diversity in this large sample of ticks.: "60 novel and 10 existing strains were inferred to be potential candidates for explaining the strain diversity in this large sample of ticks."
- [readme] The program will stop if and only if -g and -timelim are both satisfied.: "The program will stop if and only if `-g` and `-timelim` are both satisfied."
- [readme] The folder contains a list of sampleX_strainsAndProportions.csv with sampleX corresponds a particular sample.: "The folder contains a list of `sampleX_strainsAndProportions.csv` with sampleX corresponds a particular sample, and the csv file contains the strains and their proportions identified in sampleX."
- [discussion] At the moment, our approach to this problem is to post-process the result of our pipeline to identify clusters of closely related strains.: "At the moment, our approach to this problem is to post-process the result of our pipeline to identify clusters of closely related strains, but other more principled approaches should be explored"
