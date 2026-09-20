---
name: assembly-quality-threshold-filtering
description: Use when paired-end Illumina FASTQ reads have been assembled (or must
  be filtered) and you need to set an accept/discard cutoff on the PANDAseq assembly
  quality score to control sequence recovery, as for the 673 845-read M. capsulatus
  16S rRNA library.
when_to_use_negative:
- Input is already an assembled FASTA or feature table without per-base PHRED quality
  strings — there is nothing to rescore against a threshold.
- Read pairs have no suitable overlap and are discarded by the assembler before scoring
  — lowering the quality threshold cannot recover them.
- Sequencing is not Illumina paired-end with CASAVA-style quality masks (e.g. SAM/BAM,
  long reads) — use pandaseq-sam or a different QC approach instead.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: PANDAseq
  role: Assembles overlapping paired-end reads, computes the geometric-mean assembly
    quality score, and applies the accept/discard threshold filter
  repo: https://github.com/neufeld/pandaseq
- name: PANDAseq-sam
  role: Same assembly and quality-threshold filtering for SAM/BAM formatted input
  repo: https://github.com/neufeld/pandaseq-sam
- name: Illumina
  role: Upstream sequencing platform producing the paired-end FASTQ reads with CASAVA
    quality encoding that the score depends on
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:750ec0358bd7da813f5dc46a986e83a3cb319111881bf9741ed35f0ef3d863c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - Illumina
  - PANDAseq
  - PANDAseq-sam
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# assembly-quality-threshold-filtering

## Summary

Filter PANDAseq-assembled Illumina paired-end reads by the probabilistic assembly quality score, tuning the acceptance threshold (e.g. 0.9 vs 0.6) to trade sequence recovery against error rate.

## When to use

Use when paired-end Illumina FASTQ reads have been assembled (or must be filtered) and you need to set an accept/discard cutoff on the PANDAseq assembly quality score to control sequence recovery, as for the 673 845-read M. capsulatus 16S rRNA library.

## When NOT to use

- Input is already an assembled FASTA or feature table without per-base PHRED quality strings — there is nothing to rescore against a threshold.
- Read pairs have no suitable overlap and are discarded by the assembler before scoring — lowering the quality threshold cannot recover them.
- Sequencing is not Illumina paired-end with CASAVA-style quality masks (e.g. SAM/BAM, long reads) — use pandaseq-sam or a different QC approach instead.

## Inputs

- Paired-end Illumina FASTQ reads with CASAVA PHRED quality characters (e.g. the 673 845 paired-end 108-base M. capsulatus 16S rRNA reads from pandaseq_sampledata.tar)
- PANDAseq quality-score threshold values (e.g. p = 0.9 and p = 0.6)

## Outputs

- Assembled FASTQ sequences passing the quality threshold (with per-sequence quality scores)
- assembled_sequence_counts.csv with counts per threshold (expected: 598 775 at 0.9, 652 249 at 0.6)
- quality_scores.tsv with columns sequence_id, quality_score, accept_0.9, accept_0.6

## How to apply

Run PANDAseq (`pandaseq -f forward.fastq -r reverse.fastq`) on the paired-end FASTQ files with the assembly quality-score threshold set to 0.9 (the paper's default for the M. capsulatus pure-template library, yielding 598 775 of 673 845 reads). PANDAseq scores each assembly as the geometric mean of per-base match probabilities computed from CASAVA-encoded PHRED error probabilities, compensating for variable assembled lengths; sequences meeting the threshold are accepted, and pairs with no suitable overlap or failing length/uncalled-base constraints are discarded. Lowering the threshold to 0.6 re-admits assemblies whose overlap-region disagreements are dominated by doubly quality-masked mismatches (two arbitrary low-quality base calls per mismatch), which the paper showed increases recovery by ~9% (652 249 sequences). Record counts for both thresholds, write them to assembled_sequence_counts.csv, and compare against the reported values to confirm reproduction. Rationale: incorrect assemblies have inherently low scores because their mismatches occur in quality-masked regions of both reads, so the threshold is the primary error-recovery lever.

## Related tools

- **PANDAseq** (Assembles overlapping paired-end reads, computes the geometric-mean assembly quality score, and applies the accept/discard threshold filter) — https://github.com/neufeld/pandaseq
- **PANDAseq-sam** (Same assembly and quality-threshold filtering for SAM/BAM formatted input) — https://github.com/neufeld/pandaseq-sam
- **Illumina** (Upstream sequencing platform producing the paired-end FASTQ reads with CASAVA quality encoding that the score depends on)

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq
```

## Evaluation signals

- Reproducing the reference numbers on the M. capsulatus sample data: 598 775 sequences at threshold 0.9 and 652 249 at threshold 0.6 (a 9% increase).
- Every quality_score value in quality_scores.tsv lies in (0, 1] and the accept flag exactly equals (quality_score >= threshold).
- quality_scores.tsv has one row per input read pair with fields sequence_id, quality_score, accept_0.9, accept_0.6.
- Count at threshold 0.6 is strictly greater than the count at 0.9 on the same input, since the filter is nested.
- Discarded low-scoring assemblies, when inspected, carry mismatches concentrated in quality-masked regions of both reads.

## Limitations

- Sequences with many low-quality bases yield uncertain error corrections, so threshold choice trades recovered reads against possibly incorrect assemblies.
- Assemblies whose overlap-region base disagreements fall in the quality-masked regions of both reads are automatically driven below the threshold and discarded, even if they may not be truly incorrect.
- No PANDAseq software version number is documented in the paper, complicating exact reproduction of threshold behaviour across releases.

## Evidence

- [results] Of these, 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9.: "Of these, 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9."
- [results] We assembled the same single-template data with a quality threshold of 0.6 and this increased the number of sequences assembled by 9%, yielding 652 249 sequences.: "We assembled the same single-template data with a quality threshold of 0.6 and this increased the number of sequences assembled by 9%, yielding 652 249 sequences."
- [methods] PANDAseq enables users to reject sequences based on low quality score, lengths that are too short or too long, or the presence of uncalled bases.: "PANDAseq enables users to reject sequences based on low quality score, lengths that are too short or too long, or the presence of uncalled bases."
- [discussion] incorrect assemblies have low quality scores, as these mismatches occur in quality-masked regions of both reads, and are discarded: "incorrect assemblies have low quality scores, as these mismatches occur in quality-masked regions of both reads, and are discarded"
- [readme] pandaseq -f forward.fastq -r reverse.fastq: "The short version is:

	pandaseq -f forward.fastq -r reverse.fastq"
