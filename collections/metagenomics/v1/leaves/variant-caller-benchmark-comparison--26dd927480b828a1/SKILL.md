---
name: variant-caller-benchmark-comparison
description: Use when you need to empirically compare shared-variant callers on shotgun
  metagenomic data with known ground truth, especially to quantify false positive
  reduction and F1 trends across mean sequencing depths.
when_to_use_negative:
- Input is real (non-simulated) metagenomic data without a ground-truth variant set
  — F1/TPR/FP cannot be computed and the benchmark design does not apply.
- The goal is single-sample variant calling rather than detecting shared informative
  variants across a population of shotgun metagenomic samples.
- Data are single-cell mitochondrial (mtDNA) datasets, where MQuad rather than MetaQuad
  is the appropriate tool.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaQuad
  role: Primary caller under evaluation; detects shared informative variants using
    a density clustering model on cellSNP-lite output
  repo: https://github.com/holab-hku/MetaQuad
- name: cellsnp-lite
  role: Piles up variants from the simulated alignments to produce the cellSNP folder
    consumed by MetaQuad (minMAF=0.02, minCOUNT=100)
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: MQuad
  role: Predecessor tool whose output format MetaQuad extends; benchmarking reuses
    its cellSNP-folder input convention
  repo: https://github.com/single-cell-genetics/MQuad
- name: Python
  role: Runtime environment for MetaQuad; installed via pip from GitHub
  repo: https://github.com/holab-hku/MetaQuad.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:3c9cce09a3fb5677da2fb3ca96b696b7de4b570c411da77f36607c2902fe5c5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad
  tools:
  - MQuad
  - MetaQuad
  - Python
  - cellsnp-lite
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# variant-caller-benchmark-comparison

## Summary

A benchmarking skill for comparing MetaQuad against state-of-the-art variant callers (BCFtools, GATK HaplotypeCaller, VarScan2, MetaSNV, inStrain) on simulated shotgun metagenomic datasets across varying mean depths, scoring F1, true positive rate, and false positive counts against a simulated ground-truth variant set.

## When to use

Use when you need to empirically compare shared-variant callers on shotgun metagenomic data with known ground truth, especially to quantify false positive reduction and F1 trends across mean sequencing depths.

## When NOT to use

- Input is real (non-simulated) metagenomic data without a ground-truth variant set — F1/TPR/FP cannot be computed and the benchmark design does not apply.
- The goal is single-sample variant calling rather than detecting shared informative variants across a population of shotgun metagenomic samples.
- Data are single-cell mitochondrial (mtDNA) datasets, where MQuad rather than MetaQuad is the appropriate tool.

## Inputs

- Simulated shotgun metagenomic sample alignments (.filtered.sorted.bam files) with a known ground-truth variant set
- cellSNP-lite output folder (AD/DP pileup matrices) from the simulated BAMs

## Outputs

- Per-method, per-depth benchmark table of F1 score, true positive rate, and false positive SNP count
- Depth-trend comparison showing F1 approaching 1 at high mean depth

## How to apply

First generate simulated shotgun metagenomic alignments (e.g., n=100 samples) with a known ground-truth variant set and vary the mean depth. Pile up variants from the .filtered.sorted.bam files using cellsnp-lite with the recommended parameters minMAF = 0.02 and minCOUNT = 100. Run MetaQuad on the cellSNP output folder (e.g., metaquad -i cellSNP -o Metaquad --minSample=5 for n=100) and accept variants with num_of_clusters >= 2. Run each comparator (BCFtools, GATK HaplotypeCaller, VarScan2, MetaSNV, inStrain) on the same datasets at each depth. Match each caller's output against the ground truth per method and per depth to compute F1 score, true positive rate, and false positive count, then assemble the comparative table.

## Related tools

- **MetaQuad** (Primary caller under evaluation; detects shared informative variants using a density clustering model on cellSNP-lite output) — https://github.com/holab-hku/MetaQuad
- **cellsnp-lite** (Piles up variants from the simulated alignments to produce the cellSNP folder consumed by MetaQuad (minMAF=0.02, minCOUNT=100)) — https://github.com/single-cell-genetics/cellsnp-lite
- **MQuad** (Predecessor tool whose output format MetaQuad extends; benchmarking reuses its cellSNP-folder input convention) — https://github.com/single-cell-genetics/MQuad
- **Python** (Runtime environment for MetaQuad; installed via pip from GitHub) — https://github.com/holab-hku/MetaQuad.git

## Examples

```
metaquad -i cellSNP -o Metaquad --minSample=5
```

## Evaluation signals

- MetaQuad shows significantly reduced false positive SNP counts relative to BCFtools, GATK HaplotypeCaller, VarScan2, MetaSNV, and inStrain.
- True positive rates are not greatly affected relative to competing methods.
- All callers trend toward F1 near 1 and near-100% true positive rates as mean depth increases.
- MetaQuad parameters match the recommended settings: minMAF=0.02 and minCOUNT=100 for pileup, minSample = 5% of total sample size, num_of_clusters >= 2.
- Cell count/depth scenario (e.g., n=100 samples) uses --minSample=5, consistent with the 5% rule.

## Limitations

- The benchmark conclusion is grounded in simulated datasets; performance on real data must be assessed separately.
- Higher cellsnp-lite minMAF/minCOUNT values are recommended to reduce computation time for subsequent steps, which may trade sensitivity.
- MetaQuad is tailored to shotgun metagenomic data (an extension of MQuad); it is not validated for other data types.

## Evidence

- [abstract] MetaQuad significantly reduces the number of false positive SNPs without greatly affecting the true positive rate compared with state-of-the-art methods.: "Empirical comparisons with other state-of-the-art methods show that MetaQuad significantly reduces the number of false positive SNPs without greatly affecting the true positive rate."
- [intro] Recommended pileup parameters.: [evidence span withheld — anchor status: not_found]
- [intro] Recommended MetaQuad detection parameters.: "Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2."
- [readme] Installation and invocation from README.: "MetaQuad takes the output folder of cellSNP-lite: metaquad  -i cellSNP -o Metaquad --minSample=5"
- [readme] Alignment step from README.: "Filter and align reads from shotgun metagenomic sequencing to a reference database using alignment tools such as bwa or bowtie2."
