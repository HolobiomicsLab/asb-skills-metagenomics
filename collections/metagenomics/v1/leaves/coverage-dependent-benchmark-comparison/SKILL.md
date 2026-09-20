---
name: coverage-dependent-benchmark-comparison
description: Use when validating an HGT/structural-variant detection method on WGSIM-simulated
  metagenomic reads where Reconstruction Accuracy must be compared against a baseline
  tool across coverage levels to confirm a coverage-dependent performance trend.
when_to_use_negative:
- Input data is real (non-simulated) metagenomic reads without known ground-truth
  transferred segments, so Reconstruction Accuracy cannot be computed.
- Only one coverage level is available, making the coverage-dependent trend criterion
  (ΔRA decreasing with coverage) untestable.
- The comparison target is not a detection/reconstruction accuracy metric (e.g. runtime
  or memory benchmarking), which this ΔRA-based skill does not address.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: LEMON
  role: Graph-based method under evaluation; detects HGT breakpoints, identifies transferred
    segments, and reconstructs the local strain whose RA is computed
  repo: https://github.com/lichen2018/LEMON
- name: LUMPY
  role: Baseline third-party SV/HGT detection tool against which LEMON's RA is differenced
    at each coverage level
- name: WGSIM
  role: Generates simulated paired-end reads (5X and 30X coverage) from true local
    strains to produce the benchmark inputs
- name: BWA
  role: Maps paired-end shotgun reads against reference genomes to produce the BAM
    inputs used for RA computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:a0a36f82e85856d5349a404c00a86151adbb4b7005a875822bc5d2fc0eb3d43a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - BWA
  - LEMON
  - LUMPY
  - WGSIM
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# coverage-dependent-benchmark-comparison

## Summary

Benchmark a graph-based HGT reconstruction method (LEMON) against a standard SV caller (LUMPY) by computing ΔRA = RA_LEMON − RA_LUMPY from simulated Reconstruction Accuracy values at multiple sequencing coverages (5X and 30X), verifying that all ΔRA are positive and decrease as coverage increases (Fig. 5a).

## When to use

Use when validating an HGT/structural-variant detection method on WGSIM-simulated metagenomic reads where Reconstruction Accuracy must be compared against a baseline tool across coverage levels to confirm a coverage-dependent performance trend.

## When NOT to use

- Input data is real (non-simulated) metagenomic reads without known ground-truth transferred segments, so Reconstruction Accuracy cannot be computed.
- Only one coverage level is available, making the coverage-dependent trend criterion (ΔRA decreasing with coverage) untestable.
- The comparison target is not a detection/reconstruction accuracy metric (e.g. runtime or memory benchmarking), which this ΔRA-based skill does not address.

## Inputs

- Simulated Reconstruction Accuracy (RA) values for LEMON and LUMPY at 5X and 30X coverage (upstream reproduction outputs from WGSIM-simulated reads, 4 repetitions)
- Coverage level metadata (5X, 30X) associating each RA value with its simulation condition

## Outputs

- delta_ra_summary.csv containing per-coverage ΔRA values and pass/fail verdicts for the positivity and decreasing-trend criteria

## How to apply

Load the simulated Reconstruction Accuracy (RA) values produced by running LEMON and LUMPY on WGSIM-simulated paired-end reads (4 repetitions) at 5X and 30X coverage from the upstream reproduction outputs. For each coverage level and repetition compute ΔRA = RA_LEMON − RA_LUMPY, matching the quantity reported in Fig. 5a of the paper. Apply the positivity criterion: every ΔRA value must be greater than 0, showing LEMON achieves higher RA than LUMPY on reference sequences of the microbiome. Apply the trend criterion: ΔRA at 5X must exceed ΔRA at 30X, confirming ΔRA decreases as coverage increases. Write per-coverage ΔRA values and pass/fail verdicts for both criteria to delta_ra_summary.csv; the task succeeds only if all ΔRA are positive and decreasing with coverage.

## Related tools

- **LEMON** (Graph-based method under evaluation; detects HGT breakpoints, identifies transferred segments, and reconstructs the local strain whose RA is computed) — https://github.com/lichen2018/LEMON
- **LUMPY** (Baseline third-party SV/HGT detection tool against which LEMON's RA is differenced at each coverage level)
- **WGSIM** (Generates simulated paired-end reads (5X and 30X coverage) from true local strains to produce the benchmark inputs)
- **BWA** (Maps paired-end shotgun reads against reference genomes to produce the BAM inputs used for RA computation)

## Examples

```
python LEMON/Scripts/get_raw_bkp.py -r metagenomic_reference.fa -u unique_reads.bam -o raw_bkp.tsv -t 4
```

## Evaluation signals

- Every computed ΔRA value (at both 5X and 30X, across all 4 repetitions) is strictly greater than 0.
- ΔRA at 5X is greater than ΔRA at 30X, i.e. ΔRA decreases monotonically with increasing coverage, matching the Fig. 5a finding.
- delta_ra_summary.csv exists, is parseable, and contains both the per-coverage ΔRA values and explicit pass/fail verdicts for the positivity and trend criteria.
- RA inputs trace back to reproduction outputs generated with WGSIM-simulated reads and 4 repetitions; repetition count matches the upstream protocol.
- Sign convention is consistent: ΔRA is computed as RA_LEMON − RA_LUMPY, not the reverse.

## Limitations

- Conclusions hold for simulated gut metagenomics data on reference sequences of the microbiome; they do not by themselves validate performance on real HMP/IBD samples.
- The comparison is restricted to LUMPY as baseline and to the evaluated coverage levels (5X and 30X); results at other coverages or against other baselines are not established.
- Reconstruction Accuracy depends on upstream steps (BWA alignment, breakpoint detection) running with default parameters as in the paper; deviations propagate into ΔRA.

## Evidence

- [methods] Simulation results indicate that LEMON identifies transferred segments accurately: "The simulated results indicate that the method can identify transferred segments accurately on reference sequences of the microbiome."
- [methods] All true local strains were used to generated 20X paired-end reads with WGSIM: "All true local strains were used to generated 20X paired-end reads with WGSIM [28] as an input of LEMON."
- [methods] LEMON depends on LUMPY and third-party tools: "Most third-party tools used in this article are set default parameters, including BWA, LUMPY [25], iRep [26], and STAR-Fusion [27]."
- [readme] README describes LEMON's purpose and pipeline used in the benchmark: "It is a software takes use of existing shotgun NGS datasets to detect HGT breakpoints, identify the transferred genome segments, and reconstructs the inserted local strain."
