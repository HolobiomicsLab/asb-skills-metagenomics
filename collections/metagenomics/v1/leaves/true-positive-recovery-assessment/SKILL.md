---
name: true-positive-recovery-assessment
description: Use when a simulated metagenomic benchmark has a known ground-truth mutation
  list and you must quantify how many true shared variants a caller (e.g. MetaQuad)
  recovers and whether it introduces false positives.
when_to_use_negative:
- Input is real (non-simulated) metagenomic data with no ground-truth mutation list,
  so true-positive recovery cannot be counted
- Data are single-cell mitochondrial variants intended for clonal substructure inference
  — use MQuad rather than this metagenomic benchmark
- Sample size is very small so the recommended minSample = 5% of total sample size
  threshold becomes degenerate
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: MetaQuad
  role: Density-clustering caller whose shared informative variant calls are evaluated
    against the simulated ground truth using the minSample threshold scaled to 5%
    of sample size
  repo: https://github.com/holab-hku/MetaQuad
- name: cellsnp-lite
  role: Produces the pileup output folder that serves as the input for MetaQuad in
    the benchmark workflow
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: Python
  role: Open-source implementation language of MetaQuad, used to run the caller and
    compare calls against ground truth
  repo: https://github.com/holab-hku/MetaQuad
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:1948dbc8727701e6df059a0bd60c2b8713cabc482a8a5aafd5ba8f3c6666d16c
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
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# true-positive-recovery-assessment

## Summary

Benchmark skill for verifying that MetaQuad recovers simulated ground-truth shared informative variants at scale, as demonstrated by scaling the simulated benchmark 10-fold to n = 1000 samples at mean depth 11.8 and counting recovered true positives versus false positives.

## When to use

Use when a simulated metagenomic benchmark has a known ground-truth mutation list and you must quantify how many true shared variants a caller (e.g. MetaQuad) recovers and whether it introduces false positives.

## When NOT to use

- Input is real (non-simulated) metagenomic data with no ground-truth mutation list, so true-positive recovery cannot be counted
- Data are single-cell mitochondrial variants intended for clonal substructure inference — use MQuad rather than this metagenomic benchmark
- Sample size is very small so the recommended minSample = 5% of total sample size threshold becomes degenerate

## Inputs

- cellSNP-lite output folder for the simulated dataset scaled to 1000 samples at mean depth 11.8
- Simulated ground-truth mutation list (7638 true variants)
- MetaQuad BIC_params.csv with per-variant num_of_clusters annotations

## Outputs

- True positive count (recovered simulated shared mutations, e.g. 7598 of 7638)
- False positive count (expected: 0)
- Filtered list of informative shared mutations with num_of_clusters >= 2

## How to apply

Start from the cellSNP-lite output folder for the scaled simulation (1000 samples, 10-fold per group, mean depth 11.8) and run MetaQuad with minSample set to 5% of the total sample size — 50 for n = 1000, mirroring 'metaquad -i cellSNP -o Metaquad --minSample=50'. Read the produced BIC_params.csv and retain variants with num_of_clusters >= 2 as informative shared mutations, per the recommended MetaQuad filtering rule. Compare the called variants against the simulated ground-truth list (7638 true variants) and tally recovered true positives and any false positives. The skill is validated when the caller recovers the expected fraction of true mutations (7598 of 7638 in the paper's large-sample run) with no false positives, confirming scalability at unchanged depth.

## Related tools

- **MetaQuad** (Density-clustering caller whose shared informative variant calls are evaluated against the simulated ground truth using the minSample threshold scaled to 5% of sample size) — https://github.com/holab-hku/MetaQuad
- **cellsnp-lite** (Produces the pileup output folder that serves as the input for MetaQuad in the benchmark workflow) — https://github.com/single-cell-genetics/cellsnp-lite
- **Python** (Open-source implementation language of MetaQuad, used to run the caller and compare calls against ground truth) — https://github.com/holab-hku/MetaQuad

## Examples

```
metaquad -i cellSNP -o Metaquad --minSample=50
```

## Evaluation signals

- True positive recovery: 7598 of 7638 simulated ground-truth mutations recovered on the n = 1000 simulation at mean depth 11.8
- Zero false positives called after filtering BIC_params.csv to num_of_clusters >= 2
- minSample threshold equals 5% of the total sample size (50 for n = 1000), matching the recommended parameter
- Only variants present in BIC_params.csv are counted; calls absent from the ground-truth list are counted as false positives
- Scalability check: performance at n = 1000 and depth 11.8 matches the smaller-sample benchmark at the same depth

## Limitations

- The accuracy claim (7598/7638 TP, 0 FP) holds only at the simulated mean depth of 11.8; performance at lower depth is not established here
- False positive rate is assessed only against the simulated ground truth, not against real metagenomic samples
- Recommended minSample parameter must scale with sample size (5% of total), so thresholds are not transferable across cohort sizes without adjustment

## Evidence

- [intro] Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2.: "Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2."
- [abstract] MetaQuad significantly reduces the number of false positive SNPs without greatly affecting the true positive rate.: "MetaQuad significantly reduces the number of false positive SNPs without greatly affecting the true positive rate."
- [readme] metaquad -i cellSNP -o Metaquad --minSample=5: "MetaQuad takes the output folder of cellSNP-lite: 

metaquad  -i cellSNP -o Metaquad --minSample=5"
- [readme] Pile up variants using cellsnp-lite with recommended parameters minMAF = 0.02 and minCOUNT = 100.: "with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100. A higher value is recommended to reduce computation time for subsequent steps."
- [readme] BIC_params.csv: A spreadsheet containing detailed parameters/statistics of all variants, sorted from highest deltaBIC to lowest: [evidence span withheld — anchor status: external_doc]
