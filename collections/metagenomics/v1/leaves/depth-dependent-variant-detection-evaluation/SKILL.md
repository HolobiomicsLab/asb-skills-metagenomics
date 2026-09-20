---
name: depth-dependent-variant-detection-evaluation
description: Use when comparing metagenomic variant-caller performance across sequencing
  depths on simulated data with a known ground-truth variant set.
when_to_use_negative:
- Real (non-simulated) metagenomic data where no ground-truth variant set exists to
  compute F1, TPR, and false positives
- Single-cell clonal substructure discovery tasks, where MQuad (not MetaQuad's metagenomic
  benchmark) applies
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: MetaQuad
  role: Detect shared informative variants via density-based clustering on the cellSNP
    folder; the method under benchmark
  repo: https://github.com/holab-hku/MetaQuad
- name: cellsnp-lite
  role: Pile up variants from the simulated shotgun metagenomic alignments with minMAF
    = 0.02 and minCOUNT = 100
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: Python
  role: MetaQuad is an open-source Python package; used to run the variant detection
    and assemble benchmark metrics
  repo: https://github.com/holab-hku/MetaQuad
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:cf007f76dbcd97681ea1f323d0dc597739b5f8e43875f547d113220590a7b386
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad
  tools:
  - MetaQuad
  - Python
  - cellsnp-lite
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# depth-dependent-variant-detection-evaluation

## Summary

Benchmark protocol for evaluating shared variant callers (MetaQuad vs BCFtools, GATK HaplotypeCaller, VarScan2, MetaSNV, inStrain) on simulated shotgun metagenomic datasets across varying mean depths, using F1 score, true positive rate, and false positive count against simulated ground truth.

## When to use

Use when comparing metagenomic variant-caller performance across sequencing depths on simulated data with a known ground-truth variant set.

## When NOT to use

- Real (non-simulated) metagenomic data where no ground-truth variant set exists to compute F1, TPR, and false positives
- Single-cell clonal substructure discovery tasks, where MQuad (not MetaQuad's metagenomic benchmark) applies

## Inputs

- Simulated shotgun metagenomic alignments (.filtered.sorted.bam) from n=100 samples at varying mean depths
- Simulated ground-truth variant set
- cellSNP-lite output folder (cellSNP)

## Outputs

- Per-method, per-depth comparative benchmark table of F1 score, true positive rate, and false positive count
- Depth trend showing F1 approaching 1 and true positive rates near 100% at high mean depth

## How to apply

Pile up variants from the simulated n=100-sample shotgun metagenomic alignments using cellsnp-lite with recommended parameters minMAF = 0.02 and minCOUNT = 100, producing a cellSNP output folder. Run MetaQuad on that folder (metaquad -i cellSNP -o Metaquad) with minSample set to 5% of the total sample size (--minSample=5 for n=100) and accept variants with num_of_clusters >= 2. Run each comparator caller on the same simulated datasets at each mean depth, match each caller's output against the simulated ground-truth variants, and compute F1 score, true positive rate, and false positive count per method per depth. Validation: accept the benchmark when MetaQuad significantly reduces false positive SNPs without greatly affecting the true positive rate and F1 trends toward 1 at high mean depth.

## Related tools

- **MetaQuad** (Detect shared informative variants via density-based clustering on the cellSNP folder; the method under benchmark) — https://github.com/holab-hku/MetaQuad
- **cellsnp-lite** (Pile up variants from the simulated shotgun metagenomic alignments with minMAF = 0.02 and minCOUNT = 100) — https://github.com/single-cell-genetics/cellsnp-lite
- **Python** (MetaQuad is an open-source Python package; used to run the variant detection and assemble benchmark metrics) — https://github.com/holab-hku/MetaQuad

## Examples

```
metaquad -i cellSNP -o Metaquad --minSample=5
```

## Evaluation signals

- MetaQuad significantly reduces the number of false positive SNPs relative to BCFtools, GATK HaplotypeCaller, VarScan2, MetaSNV, and inStrain
- True positive rate is not greatly affected relative to state-of-the-art methods
- F1 score trends toward 1 and true positive rates approach 100% as mean depth increases for all tools
- MetaQuad accepts only variants with num_of_clusters >= 2 and minSample set to 5% of total sample size (--minSample=5 for n=100)

## Limitations

- Benchmark relies on simulated n=100-sample datasets; performance on real patient data must be assessed separately (e.g. the H. pylori antibiotic resistance gene application)
- cellsnp-lite minCOUNT = 100 is a recommended parameter; a higher value is recommended to reduce computation time for subsequent steps but may reduce sensitivity at low depths

## Evidence

- [abstract] MetaQuad significantly reduces the number of false positive SNPs without greatly affecting the true positive rate compared with state-of-the-art methods.: "Empirical comparisons with other state-of-the-art methods show that MetaQuad significantly reduces the number of false positive SNPs without greatly affecting the true positive rate."
- [intro] Pile up variants using cellsnp-lite with minMAF = 0.02 and minCOUNT = 100.: "Pile up variants using [cellsnp-lite](https://github.com/single-cell-genetics/cellsnp-lite). <br/> with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100."
- [methods] Run MetaQuad on cellSNP-lite output with minSample = 5%.: "metaquad  -i cellSNP -o Metaquad --minSample=5"
- [intro] Detect informative mutations using MetaQuad with num_of_clusters >= 2.: "Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2."
- [readme] Filter and align reads to a reference database using bwa or bowtie2.: "Filter and align reads from shotgun metagenomic sequencing to a reference database using alignment tools such as bwa or bowtie2. The recommended suffix for the final files is ".filtered.sorted.bam"."
