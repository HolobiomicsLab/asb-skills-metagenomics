---
name: f1-and-false-positive-rate-computation
description: Use when benchmarking variant callers on simulated shotgun metagenomic
  datasets and you need per-method, per-depth F1, true positive rate, and false positive
  counts against a known ground-truth variant set.
when_to_use_negative:
- Input is real (non-simulated) metagenomic data with no simulated ground-truth variant
  set to match against
- Variant caller outputs have not yet been generated — this skill evaluates calls,
  it does not produce them
- The goal is variant discovery itself rather than quantitative comparison of caller
  performance
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: MetaQuad
  role: Benchmarked variant caller under evaluation; run on cellSNP folder with --minSample=5
    and num_of_clusters >= 2 to produce calls scored for F1, TPR, and FP
  repo: https://github.com/holab-hku/MetaQuad
- name: cellsnp-lite
  role: Piles up variants from simulated alignments (minMAF = 0.02, minCOUNT = 100)
    to produce the cellSNP input for MetaQuad
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: Python
  role: Language of MetaQuad; used to match caller outputs to ground truth and compute
    per-depth F1, TPR, and FP metrics
- name: bwa
  role: Alignment tool for filtering and aligning simulated shotgun metagenomic reads
    to the reference database
- name: bowtie2
  role: Alternative alignment tool for producing .filtered.sorted.bam files
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:97ea10ff7fff800b89371b7c83ae1079d4a0292bd2964a9a58169cd2f6362e1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad
  tools:
  - MetaQuad
  - Python
  - bowtie2
  - bwa
  - cellsnp-lite
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# f1-and-false-positive-rate-computation

## Summary

Computes F1 score, true positive rate, and false positive counts for variant callers (MetaQuad, BCFtools, GATK HaplotypeCaller, VarScan2, MetaSNV, inStrain) by matching each caller's output to a simulated ground-truth variant set, across varying mean depths.

## When to use

Use when benchmarking variant callers on simulated shotgun metagenomic datasets and you need per-method, per-depth F1, true positive rate, and false positive counts against a known ground-truth variant set.

## When NOT to use

- Input is real (non-simulated) metagenomic data with no simulated ground-truth variant set to match against
- Variant caller outputs have not yet been generated — this skill evaluates calls, it does not produce them
- The goal is variant discovery itself rather than quantitative comparison of caller performance

## Inputs

- Simulated shotgun metagenomic alignment files (.filtered.sorted.bam) from n=100 samples with known ground-truth variant set
- cellSNP-lite output folder (pileup with minMAF = 0.02 and minCOUNT = 100)
- Variant call outputs from BCFtools, GATK HaplotypeCaller, VarScan2, MetaSNV, and inStrain at each mean depth
- Simulated ground-truth variant set

## Outputs

- Per-method, per-depth F1 score, true positive rate, and false positive count table
- Comparative results table reproducing the depth trend reported in the paper

## How to apply

Generate or obtain simulated shotgun metagenomic alignments (e.g. n=100 samples) for which the true variant set is known. Run each caller — MetaQuad (on cellsnp-lite output with minMAF = 0.02, minCOUNT = 100, --minSample=5, num_of_clusters >= 2) and the comparators BCFtools, GATK HaplotypeCaller, VarScan2, MetaSNV, and inStrain — on the same datasets at each mean depth. Match each caller's output against the simulated ground-truth variant set and compute F1 score, true positive rate, and false positive count per method and per depth. Assemble the per-method, per-depth metrics into a comparative table. Accept the result when the benchmark reproduces the paper's reported pattern: MetaQuad significantly reduces false positive SNPs without greatly affecting the true positive rate, and F1 trends toward 1 at high mean depth.

## Related tools

- **MetaQuad** (Benchmarked variant caller under evaluation; run on cellSNP folder with --minSample=5 and num_of_clusters >= 2 to produce calls scored for F1, TPR, and FP) — https://github.com/holab-hku/MetaQuad
- **cellsnp-lite** (Piles up variants from simulated alignments (minMAF = 0.02, minCOUNT = 100) to produce the cellSNP input for MetaQuad) — https://github.com/single-cell-genetics/cellsnp-lite
- **Python** (Language of MetaQuad; used to match caller outputs to ground truth and compute per-depth F1, TPR, and FP metrics)
- **bwa** (Alignment tool for filtering and aligning simulated shotgun metagenomic reads to the reference database)
- **bowtie2** (Alternative alignment tool for producing .filtered.sorted.bam files)

## Evaluation signals

- MetaQuad's F1 score trends toward 1 as mean depth increases
- MetaQuad shows a significantly lower false positive SNP count than BCFtools, GATK HaplotypeCaller, VarScan2, MetaSNV, and inStrain
- True positive rates are not greatly affected relative to state-of-the-art methods (near 100% TPR at high mean depth)
- Metrics are computed per method AND per mean depth, and every caller is evaluated on the identical simulated datasets
- Each caller's output is matched against the simulated ground-truth variant set rather than a proxy reference

## Limitations

- Benchmark pattern (F1 near 1 at high depth, reduced false positives) is established on simulated datasets; performance on real data with unknown ground truth is not directly measured by this evaluation
- All competing tools approach perfect performance at high mean depth, so discrimination between methods may require examining low-depth ranges or false positive counts
- cellsnp-lite pileup parameters (minMAF = 0.02, minCOUNT = 100) trade sensitivity for computation time; a higher minCOUNT is recommended to reduce downstream computation

## Evidence

- [abstract] Empirical comparisons with other state-of-the-art methods show that MetaQuad significantly reduces the number of false positive SNPs without greatly affecting the true positive rate.: "Empirical comparisons with other state-of-the-art methods show that MetaQuad significantly reduces the number of false positive SNPs without greatly affecting the true positive rate."
- [results] MetaQuad outperformed all competing variant callers, and all tools approached perfect performance (F1 near 1 and almost 100% true positive rates) as mean depth increased.: [evidence span withheld — anchor status: not_found]
- [other] Match each caller's output against the simulated ground-truth variant set and compute F1 score, true positive rate, and false positive count per method and per depth.: [evidence span withheld — anchor status: not_found]
- [readme] Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2.: "Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2."
