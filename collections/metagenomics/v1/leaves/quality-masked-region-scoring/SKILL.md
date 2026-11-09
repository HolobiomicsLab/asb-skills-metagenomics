---
name: quality-masked-region-scoring
description: Use when reproducing or auditing PANDAseq assembly quality scores from
  CASAVA FASTQ pairs, or when explaining why assemblies with mismatches in quality-masked
  overlap regions fail a quality-threshold accept/discard check.
when_to_use_negative:
- Sequences were not assembled from overlapping paired-end reads (e.g. single-end
  reads or non-overlapping pairs) — no overlap-region match probabilities exist to
  score
- Input reads are not in FASTQ with PHRED quality strings (e.g. FASTA-only input),
  since ε cannot be derived from the CASAVA quality characters
- The goal is adapter trimming rather than overlap scoring — SeqPrep-style adapter
  detection uses mismatch-fraction cutoffs (e.g. -m 0.02), not PANDAseq's probabilistic
  match scoring
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: PANDAseq
  role: Implements the overlap match-probability scoring, quality-mask handling, geometric-mean
    assembly quality score, and 0.9/0.6 threshold filtering
  repo: https://github.com/neufeld/pandaseq
- name: PANDAseq-sam
  role: Extends the same PANDAseq assembly and quality filtering to SAM/BAM formatted
    paired-end input
  repo: https://github.com/neufeld/pandaseq-sam
- name: Illumina
  role: Source of CASAVA quality-encoded paired-end reads whose masked tail regions
    feed the quality-mask handling rule
- name: SeqPrep
  role: Alternative overlapping assembler using mismatch-fraction heuristics rather
    than probabilistic quality scoring, useful for comparison
  repo: https://github.com/jstjohn/SeqPrep/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:579818a7b8b5f907f29ea418b40697de9e3e1aab7105f4cffbacf08f52bec5e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - Illumina
  - PANDAseq
  - PANDAseq-sam
  - SeqPrep
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# quality-masked-region-scoring

## Summary

Reconstructs PANDAseq's assembly quality-score computation for Illumina paired-end reads, where per-base match probabilities in the overlap are combined into a geometric-mean score used to accept (≥0.9 or ≥0.6) or discard assemblies, explaining why mismatches in doubly quality-masked regions yield low scores.

## When to use

Use when reproducing or auditing PANDAseq assembly quality scores from CASAVA FASTQ pairs, or when explaining why assemblies with mismatches in quality-masked overlap regions fail a quality-threshold accept/discard check.

## When NOT to use

- Sequences were not assembled from overlapping paired-end reads (e.g. single-end reads or non-overlapping pairs) — no overlap-region match probabilities exist to score
- Input reads are not in FASTQ with PHRED quality strings (e.g. FASTA-only input), since ε cannot be derived from the CASAVA quality characters
- The goal is adapter trimming rather than overlap scoring — SeqPrep-style adapter detection uses mismatch-fraction cutoffs (e.g. -m 0.02), not PANDAseq's probabilistic match scoring

## Inputs

- Illumina paired-end FASTQ reads with CASAVA-encoded PHRED quality strings (e.g. 108-base M. capsulatus reads from SRA024100)
- Quality threshold(s) for acceptance, e.g. p = 0.9 and p = 0.6

## Outputs

- Per-sequence overall assembly quality score in (0, 1]
- Two-column acceptance table (quality_scores.tsv) with fields sequence_id, quality_score, accept_0.9, accept_0.6
- Discard decisions for assemblies whose mismatches fall in doubly quality-masked regions

## How to apply

Load Illumina paired-end FASTQ reads and convert each CASAVA-encoded PHRED quality character to an error probability ε via 10^(-A/10), where A is the ASCII quality value. For each overlap-region base pair, compute Pr[X̂=Ŷ|X=Y] = (1-εX)(1-εY) + εXεY/3 when bases agree, and Pr[X̂=Ŷ|X≠Y] = (1/3)(1-εX)εY + (1/3)(1-εY)εX + (2/9)εXεY when they disagree, using Pr[X̂=Ŷ|Y=N] = 1/4 for uncalled bases. Apply the quality-mask handling: if one base lies in the CASAVA-masked tail region, use the other base's probability on a match or 1/4 on a mismatch; use 1/4 when both bases are masked — this is why doubly-masked mismatches drive scores down. Compute the overall assembly quality score as the geometric mean of per-base scores over the full assembled sequence to compensate for variable lengths, then flag acceptance at thresholds p = 0.9 and p = 0.6, mirroring the paper's reported thresholds (598,775 of 673,845 M. capsulatus reads at ≥0.9; a 9% increase to 652,249 at 0.6).

## Related tools

- **PANDAseq** (Implements the overlap match-probability scoring, quality-mask handling, geometric-mean assembly quality score, and 0.9/0.6 threshold filtering) — https://github.com/neufeld/pandaseq
- **PANDAseq-sam** (Extends the same PANDAseq assembly and quality filtering to SAM/BAM formatted paired-end input) — https://github.com/neufeld/pandaseq-sam
- **Illumina** (Source of CASAVA quality-encoded paired-end reads whose masked tail regions feed the quality-mask handling rule)
- **SeqPrep** (Alternative overlapping assembler using mismatch-fraction heuristics rather than probabilistic quality scoring, useful for comparison) — https://github.com/jstjohn/SeqPrep/

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq
```

## Evaluation signals

- quality_scores.tsv exists with one row per input pair and header fields sequence_id, quality_score, accept_0.9, accept_0.6
- Every quality_score lies in (0, 1]
- Each accept flag exactly equals (quality_score >= threshold) for 0.9 and 0.6 respectively
- Assemblies whose overlap mismatches occur in quality-masked regions of both reads receive low scores and are flagged as rejected at both thresholds
- On clean simulated pairs (e.g. the paper's 1,350,602 synthesized sequences), scores exceed 0.9 for all assemblies

## Limitations

- Uncertain error corrections come from reads with many low-quality bases identified by upstream processing
- Mismatches in quality-masked regions of both reads produce two low-quality, arbitrary base calls per mismatch, driving the score below acceptance thresholds even when the true sequence matches
- No software version number was provided for PANDAseq in the article, complicating exact reproduction

## Evidence

- [methods] quality-mask scoring: [evidence span withheld — anchor status: not_found]
- [results] threshold acceptance: "598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9"
- [discussion] masked-region discards: "incorrect assemblies have low quality scores, as these mismatches occur in quality-masked regions of both reads, and are discarded"
- [readme] PANDAseq usage: "PANDASEQ is a program to align Illumina reads, optionally with PCR primers embedded in the sequence, and reconstruct an overlapping sequence."
- [results] geometric mean rationale: "all 1 350 602 synthesized paired-end sequences assembled with quality scores greater than 0.9 (Figure 2)"
