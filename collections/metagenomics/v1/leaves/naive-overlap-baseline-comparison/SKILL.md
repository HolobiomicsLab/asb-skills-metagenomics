---
name: naive-overlap-baseline-comparison
description: Use when benchmarking sequence recovery of paired-end Illumina assembly
  against a naive perfect-overlap baseline on a control library (e.g. SRA024100),
  to reproduce reported 4-50% yield gains from relaxing quality thresholds 0.9 to
  0.6.
when_to_use_negative:
- Input is already an assembled or clustered sequence table — no paired-end overlap
  assembly or baseline comparison is needed
- Reads have no expected overlap between pairs (e.g. short inserts / long reads),
  so a perfect-overlap baseline is meaningless
- The goal is accuracy/error-rate evaluation rather than yield recovery comparison
  (use simulated data with known errors instead)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: PANDAseq
  role: Probabilistic paired-end assembler run at quality-score thresholds 0.9 and
    0.6 to measure sequence recovery over the naive baseline
  repo: https://github.com/neufeld/pandaseq
- name: Illumina
  role: Sequencing platform that produced the paired-end control-library reads being
    compared across assemblies
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:957c6afb64f85218895b77293ec0eef5e16e729adf6cbaeeef45affe7d445d4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - Illumina
  - PANDAseq
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# naive-overlap-baseline-comparison

## Summary

Quantifies how much a probabilistic paired-end assembler (PANDAseq) recovers over a naive perfect-overlap baseline by assembling the same control-library read pairs (SRA024100) at multiple quality-score thresholds and computing relative yield increases.

## When to use

Use when benchmarking sequence recovery of paired-end Illumina assembly against a naive perfect-overlap baseline on a control library (e.g. SRA024100), to reproduce reported 4-50% yield gains from relaxing quality thresholds 0.9 to 0.6.

## When NOT to use

- Input is already an assembled or clustered sequence table — no paired-end overlap assembly or baseline comparison is needed
- Reads have no expected overlap between pairs (e.g. short inserts / long reads), so a perfect-overlap baseline is meaningless
- The goal is accuracy/error-rate evaluation rather than yield recovery comparison (use simulated data with known errors instead)

## Inputs

- Paired-end Illumina FASTQ reads of the pooled composite control library (NCBI SRA accession SRA024100)
- PANDAseq installed (build requires zlib and libbz2 development packages)

## Outputs

- Assembled sequence counts for three assemblies (PANDAseq q=0.9, PANDAseq q=0.6, naive baseline)
- pooled_yield_comparison.csv with counts and computed percent yield increases versus naive assembly

## How to apply

Download the paired-end Illumina reads of the composite control library built from mixed pure bacterial cultures (NCBI SRA accession SRA024100). Assemble read pairs three ways: (1) PANDAseq with assembly quality score threshold 0.9, (2) PANDAseq with threshold relaxed to 0.6 to increase recovery, and (3) the naive perfect-overlap baseline, which discards any pairs whose sequenced bases do not match exactly in the overlap. Count retained sequences for each run and compute the relative yield increase versus naive (PANDAseq minus naive, divided by naive). Write counts and percent increases for all three assemblies to pooled_yield_comparison.csv. Rationale: approximately 40% of naive-assembled sequences are discarded due to uncalled or miscalled bases, which PANDAseq corrects probabilistically using overlap data; the paper reports ~3.9% gain at threshold 0.9, ~50% overall (85% within abundant clusters) at 0.6.

## Related tools

- **PANDAseq** (Probabilistic paired-end assembler run at quality-score thresholds 0.9 and 0.6 to measure sequence recovery over the naive baseline) — https://github.com/neufeld/pandaseq
- **Illumina** (Sequencing platform that produced the paired-end control-library reads being compared across assemblies)

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq -t 0.9 -w pandaseq_q09.fastq && pandaseq -f forward.fastq -r reverse.fastq -t 0.6 -w pandaseq_q06.fastq
```

## Evaluation signals

- Yield increase of PANDAseq at threshold 0.9 over naive assembly is approximately 3.9%
- Yield increase at threshold 0.6 falls within the reported 4-50% range (~50% overall; 85% within the most abundant clusters)
- Naive baseline counts are strictly <= PANDAseq counts at both thresholds (PANDAseq corrects errors that naive discards)
- pooled_yield_comparison.csv contains counts and percent increases for all three assemblies and is internally consistent (increase = (PANDAseq - naive)/naive)

## Limitations

- Read pairs with no suitable overlap are discarded entirely, so recovery is bounded by overlap detectability
- Uncertain error corrections come from reads with many low-quality bases identified by upstream processing; relaxing the threshold too far can admit incorrect assemblies with low quality scores
- The comparison is against a perfect-overlap baseline; results are specific to control/composite libraries where true overlap exists

## Evidence

- [abstract] Control library assembly showed a 4-50% increase in assembled sequences over naive assembly: "Assembly of control libraries showed a 4-50% increase in the number of assembled sequences over naïve assembly"
- [intro] ~40% of sequences discarded by naive assembly due to uncalled or miscalled bases: "approximately 40% of the sequences were discarded due to uncalled or miscalled bases."
- [results] Threshold 0.9 assembly count on the control-style data: "598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9"
- [results] Relaxing threshold to 0.6 increases assembled sequence recovery: "We assembled the same single-template data with a quality threshold of 0.6 and this increased the number of sequences assembled by 9%, yielding 652 249 sequences."
- [methods] Naive baseline discards pairs without suitable overlap: "If no suitable overlap is found, then the read pair is discarded."
- [readme] PANDAseq CLI usage from the project README: "PANDASEQ is a program to align Illumina reads, optionally with PCR primers embedded in the sequence, and reconstruct an overlapping sequence."
