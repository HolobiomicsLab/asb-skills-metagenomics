---
name: strain-calling-accuracy-metrics-computation
description: Use when a deconvolution pipeline that infers allele distributions and
  strain types must be validated against ground truth from simulated samples, requiring
  both set-membership and proportion-based accuracy measures.
when_to_use_negative:
- When no ground-truth strain or allele composition exists (real samples such as the
  24 tick samples) — metrics require known truth from simulation
- When evaluating abundance estimators against a single reference strain rather than
  mixed within-host populations; tools like Kallisto or strainEST baselines have different
  target quantities
- When comparing strains at the whole-genome level rather than MLST allele/strain-type
  level, where 5-SNP soft matching on housekeeping genes is not meaningful
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3299
tools:
- name: CPLEX
  role: Solves the ADP and SDP MILPs whose outputs (allele and strain calls) are scored
    by these metrics
  repo: https://github.com/WGS-TB/MLST
- name: editdistance
  role: Computes Levenshtein distance between allele strings, used both for strain-type
    novelty weights and for 5-SNP soft true-positive classification
  repo: https://github.com/aflc/editdistance
- name: PyEMD
  role: Computes earth mover's distance between true and inferred proportion distributions
  repo: https://github.com/garydoranjr/pyemd
- name: ART
  role: Simulates reads from known strain distributions so that ground truth is available
    for metric computation
  repo: https://github.com/WGS-TB/MLST
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:fb9a7b31ce07f06f707e28a2b869d462e9d738836b7b46696b2e7a82792cabf4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGS-TB/MLST
  tools:
  - ART
  - Bowtie v0.12.7
  - CPLEX
  - GAML
  - PyEMD
  - Python 2.7
  - editdistance
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# strain-calling-accuracy-metrics-computation

## Summary

A set of evaluation metrics — precision, recall, total variation distance (TVD), earth mover's distance (EMD), and soft-precision/soft-recall with a 5-SNP tolerance — used to quantify how accurately an ADP (allele diversity) and SDP (strain diversity) pipeline recovers known allele compositions and strain types from simulated B. burgdorferi MLST samples.

## When to use

Use when a deconvolution pipeline that infers allele distributions and strain types must be validated against ground truth from simulated samples, requiring both set-membership and proportion-based accuracy measures.

## When NOT to use

- When no ground-truth strain or allele composition exists (real samples such as the 24 tick samples) — metrics require known truth from simulation
- When evaluating abundance estimators against a single reference strain rather than mixed within-host populations; tools like Kallisto or strainEST baselines have different target quantities
- When comparing strains at the whole-genome level rather than MLST allele/strain-type level, where 5-SNP soft matching on housekeeping genes is not meaningful

## Inputs

- True simulated strain-type distributions per EvoMod mechanism (EvoMod1/2/2e/2n/3) over the B. burgdoferi MLST scheme (8 genes, 1726 alleles, 753 strain types)
- ADP-inferred allele sets and allele proportions (geneX_proportions.csv files)
- SDP-called strain types and strain proportions per sample (sampleX_strainsAndProportions.csv files)
- Pairwise edit-distance matrices for each gene (editDist folder)

## Outputs

- Aggregated precision, recall, TVD, and EMD statistics per EvoMod mechanism for the ADP stage
- Exact and soft (5-SNP tolerance) precision/recall plus TVD for the SDP stage, for both perfect-allele input and full-pipeline (chained ADP errors) configurations
- Comparison table (analogous to Table 2) quantifying SDP degradation from ADP-stage errors

## How to apply

After running the two-stage MILP pipeline (ADP solved per sample and locus with CPLEX 12.6.3.0, then SDP on the inferred allele distributions), compare called alleles/strains against the true simulated distributions generated with EvoMod1/2/2e/2n/3 mechanisms. Compute per-sample precision (fraction of called strains that are true) and recall (fraction of true strains recovered); call a strain a TP under the soft variant when it differs from a true strain type by at most 5 SNPs, using the editdistance package for edit distances. Compute TVD as the total variation distance between true and inferred proportion vectors and EMD with PyEMD to capture proportion misallocation. Aggregate statistics per EvoMod mechanism and per pipeline configuration (perfect input vs. chained ADP output) to reveal error propagation, e.g. exact SDP precision/recall dropping from ~0.8 to ~0.3-0.5 with TVD rising to ~0.6-0.8 when ADP errors feed forward.

## Related tools

- **CPLEX** (Solves the ADP and SDP MILPs whose outputs (allele and strain calls) are scored by these metrics) — https://github.com/WGS-TB/MLST
- **editdistance** (Computes Levenshtein distance between allele strings, used both for strain-type novelty weights and for 5-SNP soft true-positive classification) — https://github.com/aflc/editdistance
- **PyEMD** (Computes earth mover's distance between true and inferred proportion distributions) — https://github.com/garydoranjr/pyemd
- **ART** (Simulates reads from known strain distributions so that ground truth is available for metric computation) — https://github.com/WGS-TB/MLST

## Examples

```
python run_sim.py -i 40 -c 30 -b ../bowtieFolder/ -s ../../samtools -a ../../art_illumina
```

## Evaluation signals

- ADP-stage metrics fall in the reported ranges: precision ~0.92-0.96 and recall ~0.90-0.92 across EvoMod mechanisms
- SDP on perfect allele calls achieves exact precision/recall around ~0.8, while full-pipeline SDP degrades to ~0.3-0.5 with TVD rising to ~0.6-0.8 — reproducing the Table 2 pattern
- Soft-precision/soft-recall (5-SNP tolerance) should be strictly higher than exact precision/recall, since near-identical strain calls count as true positives
- TVD and EMD must be non-negative, with TVD = 0 only when true and inferred proportion vectors match exactly
- Per-locus evaluation across all 8 housekeeping genes should each contain calls from the geneX_proportions.csv outputs

## Limitations

- Metrics depend on simulation assumptions (EvoMod mutation parameters m=2, edit distance d≤15), so absolute values may not transfer to real within-host diversity
- The 5-SNP soft TP threshold is an arbitrary tolerance; closely related strain clusters required ad-hoc post-processing rather than a principled method
- Python 2.7 tooling (editdistance, PyEMD) is deprecated and the PyEMD wrapper explicitly does not support Python 3

## Evidence

- [methods] Metrics definition for soft matching: [evidence span withheld — anchor status: not_found]
- [results] SDP degradation finding: [evidence span withheld — anchor status: not_found]
- [abstract] Simulated-data accuracy claim: "Our method performs accurately on simulated data and generates results on a real data set of Borrelia burgdorferi genomes"
- [methods] Edit distance package used: "In computing edit distance, we used the package by https://github.com/aflc/editdistance"
- [readme] EMD package used in pipeline README: "We also use the package PyEMD for EMD calculations, where we hereby acknowledge Gary Doran for his PyEMD package"
