---
name: simulation-based-benchmark-evaluation
description: Use when a variant-calling or clustering method must be validated against
  simulated data with a known ground-truth mutation list, especially to test scalability
  to larger sample sizes.
when_to_use_negative:
- The data is real (non-simulated) patient or environmental metagenomic data with
  no ground-truth mutation list — recovery-based TP/FP scoring is impossible
- The analysis goal is downstream strain/clonal inference rather than benchmarking
  detection performance
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: MetaQuad
  role: Detects informative shared variants on the simulated cellSNP input using a
    density clustering model; called variants are scored against ground truth
  repo: https://github.com/holab-hku/MetaQuad
- name: cellsnp-lite
  role: Piles up per-sample variant allele and depth counts from the simulated BAMs
    to form the cellSNP folder input to MetaQuad
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: Python
  role: Open-source implementation language of MetaQuad; used to run the benchmark
    comparison script
  repo: https://github.com/holab-hku/MetaQuad
- name: bwa
  role: Aligns simulated shotgun metagenomic reads to the reference database producing
    .filtered.sorted.bam files
- name: samtools
  role: Computes coverage statistics (samtools coverage) used to verify mean depth
    of the simulated data (e.g. 11.8)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:af6749eff8af9eadf7e94f025bed8e810cd4c85d85bff790b44b975aa7b8e76b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad
  tools:
  - MetaQuad
  - Python
  - bwa
  - cellsnp-lite
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# simulation-based-benchmark-evaluation

## Summary

Benchmark a variant-discovery method on simulated shotgun metagenomic data with known ground truth, scaling to large sample sizes (e.g. n=1000 at mean depth 11.8) and scoring recovered true positives against false positives. Used to demonstrate that a tool's performance holds when the benchmark is scaled 10-fold.

## When to use

Use when a variant-calling or clustering method must be validated against simulated data with a known ground-truth mutation list, especially to test scalability to larger sample sizes.

## When NOT to use

- The data is real (non-simulated) patient or environmental metagenomic data with no ground-truth mutation list — recovery-based TP/FP scoring is impossible
- The analysis goal is downstream strain/clonal inference rather than benchmarking detection performance

## Inputs

- cellSNP-lite output folder for the simulated dataset (sparse AD/DP matrices, cellsnp-lite run with recommended minMAF=0.02 and minCOUNT=100)
- Simulated ground-truth mutation list (e.g. 7638 true positive mutations)
- Simulated shotgun metagenomic BAM files aligned with bwa or bowtie2 (suffix .filtered.sorted.bam)

## Outputs

- BIC_params.csv from MetaQuad listing per-variant deltaBIC and num_of_clusters
- Filtered list of informative shared mutations (num_of_clusters >= 2)
- True-positive / false-positive counts versus ground truth (e.g. 7598 of 7638 TP, 0 FP)

## How to apply

Generate (or obtain) a simulated shotgun metagenomic dataset with a known ground-truth mutation list — in the MetaQuad benchmark this was a 1000-sample simulation (10-fold per group) at mean depth 11.8, processed with cellSNP-lite. Run the tool under evaluation on the cellSNP output using a sample-proportional threshold: MetaQuad with minSample set to 5% of the total sample size (e.g. --minSample=50 for n=1000), mirroring the command 'metaquad -i cellSNP -o Metaquad --minSample=50'. Read the produced BIC_params.csv and retain variants with num_of_clusters >= 2 as informative shared mutations, per the recommended filter. Compare called variants against the ground-truth list (7638 true variants) and count true positives and false positives. Correctness is judged by recovering the expected number of true positives (7598/7638 in the n=1000 case) with zero false positives, confirming scalability to large datasets.

## Related tools

- **MetaQuad** (Detects informative shared variants on the simulated cellSNP input using a density clustering model; called variants are scored against ground truth) — https://github.com/holab-hku/MetaQuad
- **cellsnp-lite** (Piles up per-sample variant allele and depth counts from the simulated BAMs to form the cellSNP folder input to MetaQuad) — https://github.com/single-cell-genetics/cellsnp-lite
- **Python** (Open-source implementation language of MetaQuad; used to run the benchmark comparison script) — https://github.com/holab-hku/MetaQuad
- **bwa** (Aligns simulated shotgun metagenomic reads to the reference database producing .filtered.sorted.bam files)
- **samtools** (Computes coverage statistics (samtools coverage) used to verify mean depth of the simulated data (e.g. 11.8))

## Examples

```
metaquad -i cellSNP -o Metaquad --minSample=50
```

## Evaluation signals

- True-positive recovery matches the expected value: 7598 of 7638 ground-truth mutations recovered at n=1000, mean depth 11.8
- Zero false positives called against the simulated ground truth
- minSample threshold equals 5% of total sample size (50 for n=1000), matching the recommended parameterization
- Retained variants all have num_of_clusters >= 2 in BIC_params.csv
- Mean sequencing depth of the simulated dataset confirmed at ~11.8 (e.g. via samtools coverage)

## Limitations

- Results depend on the simulation design (per-group 10-fold scaling, mean depth 11.8); performance at other depths or sample compositions is not established by this benchmark alone
- A higher minCOUNT in cellsnp-lite is recommended to reduce computation time for large datasets, implying runtime cost scales with sample size
- Higher cellsnp-lite thresholds may miss low-frequency variants, bounding achievable true-positive recovery

## Evidence

- [results] On the 1000-sample simulation at mean depth 11.8, MetaQuad with a minSample threshold of 50 (5% of total sample size) recovered 7598 of 7638 true positive mutations with no false positives: [evidence span withheld — anchor status: not_found]
- [readme] Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2.: "Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2."
- [readme] Pile up variants using [cellsnp-lite](https://github.com/single-cell-genetics/cellsnp-lite). <br/> with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100.: "Pile up variants using [cellsnp-lite](https://github.com/single-cell-genetics/cellsnp-lite). <br/> with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100."
- [abstract] MetaQuad utilizes a density-based clustering technique to effectively distinguish between shared variants and non-polymorphic sites using shotgun metagenomic data.: "MetaQuad utilizes a density-based clustering technique to effectively distinguish between shared variants and non-polymorphic sites using shotgun metagenomic data."
