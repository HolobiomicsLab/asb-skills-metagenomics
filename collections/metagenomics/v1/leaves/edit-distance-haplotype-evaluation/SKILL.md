---
name: edit-distance-haplotype-evaluation
description: Use when evaluating deconvolved allele or strain-type calls against known
  ground truth, especially when called sequences may be near-misses (novel strains
  differing by a few SNPs) rather than exact matches.
when_to_use_negative:
- When the ground truth is unknown (real tick samples) — these metrics require true
  strain type/allele compositions; for real data use diversity-level interpretation
  instead
- When comparing sequences where positional indel/alignment semantics matter beyond
  Levenshtein distance (the metric is plain edit distance, not alignment-aware)
- When abundance estimation quality alone is needed without sequence identity context
  — plain TVD suffices and edit-distance soft matching adds no information
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: editdistance
  role: Fast C++/Cython Levenshtein distance computation between allele and strain
    sequences for soft TP assignment and distance matrices
  repo: https://github.com/aflc/editdistance
- name: PyEMD
  role: Earth-Mover's Distance computation over edit-distance-ground-distance distributions
    of strain proportions
  repo: https://github.com/garydoranjr/pyemd
- name: Kallisto
  role: Baseline abundance-estimation method whose per-locus allele calls are scored
    with the same precision/recall/TVD metrics
- name: strainEST
  role: Baseline strain-composition method scored with soft-precision, soft-recall
    and EMD across EvoMod simulations
  repo: https://github.com/compmetagen/strainest
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:8e936891d19ed353e59a51259bd88932433e8cb02d49c0b7039ed4207800d637
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aflc/editdistance
  tools:
  - ART
  - Bowtie v0.12.7
  - GAML
  - Kallisto
  - PyEMD
  - Python 2.7
  - editdistance
  - strainEST
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# edit-distance-haplotype-evaluation

## Summary

Evaluates inferred allele and strain-type (haplotype) calls from the two-stage MLST deconvolution pipeline using sequence edit distance as the comparison metric, combining exact precision/recall with soft-precision/soft-recall (calls within a SNP tolerance count as TP), Total Variation Distance, and Earth-Mover's Distance.

## When to use

Use when evaluating deconvolved allele or strain-type calls against known ground truth, especially when called sequences may be near-misses (novel strains differing by a few SNPs) rather than exact matches.

## When NOT to use

- When the ground truth is unknown (real tick samples) — these metrics require true strain type/allele compositions; for real data use diversity-level interpretation instead
- When comparing sequences where positional indel/alignment semantics matter beyond Levenshtein distance (the metric is plain edit distance, not alignment-aware)
- When abundance estimation quality alone is needed without sequence identity context — plain TVD suffices and edit-distance soft matching adds no information

## Inputs

- Called allele lists and proportions per sample per locus (e.g. geneX_proportions.csv from the ADP stage)
- Called strain types and proportions per sample (sampleX_strainsAndProportions.csv from the SDP stage)
- Ground-truth simulated strain type distributions and true allele compositions
- B. burgdorferi MLST allele database (8 housekeeping genes, 1726 alleles, 753 strain types)
- Precomputed per-gene edit distance matrices (editDist folder)

## Outputs

- Per-locus precision, recall, and TVD (e.g. clpA 0.99 ± 0.009 for B-MLST vs Kallisto)
- Soft-precision, soft-recall, and EMD for strain-type calls
- Aggregated means ± standard deviations across parameter combinations (comparison CSV)
- Per-statistic PNG boxplots/summary plots in the results folder

## How to apply

Compute Levenshtein edit distance between each called allele/strain type and the true sequences using the aflc/editdistance package (Hyyrö bit-parallel algorithm). A called strain counts as a true positive under the soft criteria when it differs from the true strain type by at most 5 SNPs; also compute exact precision (TP/(TP+FP)) and recall (TP/(TP+FN)). For proportions, compute Total Variation Distance per gene (clpA, clpX, nifS, pepX, pyrG, recG, rplB, uvrA) and Earth-Mover's Distance with the garydoranjr/pyemd package using edit-distance matrices as the ground distance. In the SDP stage itself, edit distance also enters the objective: novel strain types are penalized by weights derived from edit distance with an error bound ε=0.1 and tolerance δ=0.001, and EvoMod simulations bound allele edit distance at d=15. Compare against baselines (Kallisto, strainEST) and aggregate means ± standard deviations across parameter combinations (coverage c∈{30,100,300}, edit distance d∈{5,10,15,20,25}).

## Related tools

- **editdistance** (Fast C++/Cython Levenshtein distance computation between allele and strain sequences for soft TP assignment and distance matrices) — https://github.com/aflc/editdistance
- **PyEMD** (Earth-Mover's Distance computation over edit-distance-ground-distance distributions of strain proportions) — https://github.com/garydoranjr/pyemd
- **Kallisto** (Baseline abundance-estimation method whose per-locus allele calls are scored with the same precision/recall/TVD metrics)
- **strainEST** (Baseline strain-composition method scored with soft-precision, soft-recall and EMD across EvoMod simulations) — https://github.com/compmetagen/strainest

## Examples

```
import editdistance; d = editdistance.eval(called_allele, true_allele); tp = d <= 5
```

## Evaluation signals

- B-MLST precision exceeds Kallisto at every locus (e.g. clpA 0.99 ± 0.009 vs 0.97 ± 0.014; nifS 0.96 vs 0.89) while TVD remains comparable
- On ADP simulations precision lands ~0.92–0.96 and recall ~0.90–0.92 across EvoMod mechanisms
- Full-pipeline SDP degradation is reproducible: exact precision/recall drop from ~0.8 to ~0.3–0.5 and TVD rises to ~0.6–0.8 relative to perfect-allele-input SDP
- Every reported statistic is accompanied by a mean and standard deviation over parameter combinations (c∈{30,100,300}, d∈{5,10,15,20,25}, 40 iterations per combination)
- EMD computations use the per-gene edit-distance matrices from the editDist folder as ground distances

## Limitations

- Soft TP assignment uses an arbitrary 5-SNP tolerance for strain types, which can mask biologically meaningful near-miss errors
- Full-pipeline results show that small ADP-stage errors dramatically degrade downstream SDP strain typing, so strong per-locus allele metrics do not guarantee accurate strain-type recovery
- Evaluation depends on simulated EvoMod ground truth; real-sample accuracy (e.g. the 60 novel and 10 existing strains inferred from 24 tick samples) cannot be validated with these metrics
- PyEMD requires Python 2 and numpy/scipy, matching the pipeline's Python 2.7 dependency

## Evidence

- [other] Soft precision/recall definition: [evidence span withheld — anchor status: not_found]
- [readme] Edit distance package citation: "In computing edit distance, we used the package by https://github.com/aflc/editdistance"
- [methods] EMD package citation: "https://github.com/garydoranjr/pyemd."
- [other] Baseline comparison finding: [evidence span withheld — anchor status: not_found]
- [methods] MLST database scale: "the traditional B. burgdorferi MLST scheme [11] composed of 8 housekeeping genes having a combined total of 1726 known alleles"
