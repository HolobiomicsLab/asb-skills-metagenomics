---
name: control-library-benchmarking
description: Use when quantifying how many more assembled sequences an error-correcting
  paired-end assembler recovers than a strict perfect-match baseline on a control
  library with known composition.
when_to_use_negative:
- Input data is a single-template library without mixed pure bacterial cultures —
  the 4-50% yield range was established on the pooled composite control library, not
  single-template data (which showed a 9% gain at threshold 0.6)
- The goal is error-rate assessment of assemblies rather than yield/recovery benchmarking
  — use the paper's Table 1 mismatch analysis (0.02-0.08% introduced errors) instead
- Reads are not paired-end Illumina reads with overlapping regions, so no overlap-based
  assembly comparison is possible
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: PANDAseq
  role: Probabilistic error-correcting paired-end assembler run at quality thresholds
    0.9 and 0.6 to count recovered sequences
  repo: https://github.com/neufeld/pandaseq
- name: Illumina
  role: Sequencing platform that generated the paired-end reads of the composite control
    library
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:72b8318a54ee294872d5ea159e5ec73c5dfb4eb75eba6cd086714ca866746d2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - Illumina
  - PANDAseq
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# control-library-benchmarking

## Summary

Benchmarks a probabilistic paired-end assembler (PANDAseq) against a naive perfect-overlap baseline on a pooled composite control library (NCBI SRA024100), quantifying relative yield gains at quality thresholds 0.9 and 0.6.

## When to use

Use when quantifying how many more assembled sequences an error-correcting paired-end assembler recovers than a strict perfect-match baseline on a control library with known composition.

## When NOT to use

- Input data is a single-template library without mixed pure bacterial cultures — the 4-50% yield range was established on the pooled composite control library, not single-template data (which showed a 9% gain at threshold 0.6)
- The goal is error-rate assessment of assemblies rather than yield/recovery benchmarking — use the paper's Table 1 mismatch analysis (0.02-0.08% introduced errors) instead
- Reads are not paired-end Illumina reads with overlapping regions, so no overlap-based assembly comparison is possible

## Inputs

- Paired-end Illumina FASTQ reads from the pooled composite control library (NCBI SRA accession SRA024100)
- PANDAseq assembler (built from github.com/neufeld/pandaseq)
- Naive perfect-overlap assembly baseline (baseline_naive)

## Outputs

- pooled_yield_comparison.csv with assembled-sequence counts for PANDAseq (threshold 0.9), PANDAseq (threshold 0.6), and naive baseline, plus computed percent yield increases
- Relative yield increase percentages versus naive assembly (expected ~3.9% at 0.9; ~50% overall, ~85% in abundant clusters, at 0.6)

## How to apply

Download the paired-end Illumina reads of the composite control library built from mixed pure bacterial cultures (NCBI SRA accession SRA024100). Assemble read pairs three ways: (1) PANDAseq with assembly quality score threshold 0.9, (2) PANDAseq with the threshold relaxed to 0.6 to recover reads with low-quality overlap bases, and (3) a naive perfect-overlap baseline that discards any pair whose sequenced bases mismatch in the overlap. Count retained sequences for each assembly and compute the relative yield increase versus naive as (PANDAseq_count - naive_count) / naive_count. Write counts and percent increases to pooled_yield_comparison.csv; validate that the 0.9-threshold gain is near 3.9% and the 0.6-threshold gain is roughly 50% overall (85% within the most abundant clusters), within the paper's reported 4-50% range.

## Related tools

- **PANDAseq** (Probabilistic error-correcting paired-end assembler run at quality thresholds 0.9 and 0.6 to count recovered sequences) — https://github.com/neufeld/pandaseq
- **Illumina** (Sequencing platform that generated the paired-end reads of the composite control library)

## Examples

```
pandaseq -f SRA024100_1.fastq -r SRA024100_2.fastq -t 0.9 -w pooled_q09.fasta && pandaseq -f SRA024100_1.fastq -r SRA024100_2.fastq -t 0.6 -w pooled_q06.fasta
```

## Evaluation signals

- PANDAseq at threshold 0.9 yields approximately 3.9% more assembled sequences than the naive perfect-overlap baseline on SRA024100
- PANDAseq at threshold 0.6 yields roughly 50% more assembled sequences than naive (with ~85% more sequences in the most abundant clusters)
- Both computed percent increases fall within the paper's reported 4-50% range over naive assembly
- pooled_yield_comparison.csv contains counts for all three assemblies (0.9-threshold, 0.6-threshold, naive) and each percent increase equals (PANDAseq - naive) / naive
- Lowering the threshold from 0.9 to 0.6 must strictly increase the number of assembled sequences

## Limitations

- Approximately 40% of sequences are discarded by naive perfect-overlap assembly due to uncalled or miscalled bases, defining the ceiling of recoverable yield for baselines
- Relaxing the quality threshold from 0.9 to 0.6 trades strictness for recovery; uncertain error corrections come from reads with many low-quality bases
- The reported gains are specific to the control library composition; the single-template M. capsulatus dataset showed only a 9% increase at threshold 0.6
- No PANDAseq software version is provided in the article, complicating exact reproducibility

## Evidence

- [abstract] 4-50% yield gain over naive: "Assembly of control libraries showed a 4-50% increase in the number of assembled sequences over naïve assembly"
- [results] dataset accession: "made from mixed pure bacterial cultures (NCBI SRA accession SRA024100)"
- [results] threshold 0.6 recovery: "We assembled the same single-template data with a quality threshold of 0.6 and this increased the number of sequences assembled by 9%, yielding 652 249 sequences."
- [intro] naive baseline loss: "approximately 40% of the sequences were discarded due to uncalled or miscalled bases."
- [readme] PANDAseq usage: "pandaseq -f forward.fastq -r reverse.fastq"
