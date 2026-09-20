---
name: assembly-quality-score-geometric-mean-aggregation
description: Use when scoring assembled Illumina paired-end (FASTQ, PHRED/CASAVA qualities)
  sequences by overlap base agreement and filtering them against a quality threshold
  such as p = 0.9 or p = 0.6.
when_to_use_negative:
- Input reads have no overlapping region, so there is no overlap evidence from which
  to compute match probabilities (PANDAseq discards such pairs)
- Input is already an assembled sequence or feature table without per-base quality
  strings — no per-base error probabilities can be derived
- The goal is generic read QC (e.g. adapter trimming or FastQC-style reports) rather
  than scoring the correctness of a paired-end overlap assembly
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0102
tools:
- name: PANDAseq
  role: Assembles the paired-end reads and computes the overlap match probabilities,
    overall quality score, and threshold filtering implemented by this skill
  repo: https://github.com/neufeld/pandaseq
- name: PANDAseq-sam
  role: Extends PANDAseq quality-scored assembly to SAM/BAM-formatted input
  repo: https://github.com/neufeld/pandaseq-sam
- name: Illumina
  role: Sequencing platform whose paired-end reads, quality encoding, and CASAVA quality
    masks define the score inputs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:53f556b9904559540d9fb5fa286f705582f78b1553849a9114909e2c3653b5a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - Illumina
  - PANDAseq
  - PANDAseq-sam
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# assembly-quality-score-geometric-mean-aggregation

## Summary

Reconstructs PANDAseq-style assembly quality scoring: per-base match probabilities in the paired-end overlap (with CASAVA quality-mask handling) are aggregated over the full assembled sequence via a geometric mean, and the score is compared to acceptance thresholds (0.9 and 0.6) to accept or discard assemblies.

## When to use

Use when scoring assembled Illumina paired-end (FASTQ, PHRED/CASAVA qualities) sequences by overlap base agreement and filtering them against a quality threshold such as p = 0.9 or p = 0.6.

## When NOT to use

- Input reads have no overlapping region, so there is no overlap evidence from which to compute match probabilities (PANDAseq discards such pairs)
- Input is already an assembled sequence or feature table without per-base quality strings — no per-base error probabilities can be derived
- The goal is generic read QC (e.g. adapter trimming or FastQC-style reports) rather than scoring the correctness of a paired-end overlap assembly

## Inputs

- Illumina paired-end FASTQ (or qseq/SAM/BAM via PANDAseq-sam) forward and reverse reads with PHRED quality strings
- Per-pair optimal overlap assignment from PANDAseq assembly
- CASAVA quality-mask information marking low-quality tail positions
- Quality threshold values (p = 0.9 and p = 0.6 as in the paper)

## Outputs

- Per-sequence assembly quality score in (0, 1] (geometric mean of per-base match probabilities)
- Two-column acceptance table (e.g. quality_scores.tsv with sequence_id, quality_score, accept_0.9, accept_0.6)
- Set of accepted versus discarded assembled sequences

## How to apply

For each paired-end read, convert CASAVA quality characters to error probabilities ε = 10^(-A/10). In the overlap identified by PANDAseq's optimal-overlap search, compute Pr[X̂=Ŷ|X=Y] = (1-εX)(1-εY) + εXεY/3 for agreeing bases and Pr[X̂=Ŷ|X≠Y] = (1/3)(1-εX)εY + (1/3)(1-εY)εX + (2/9)εXεY for disagreements, using 1/4 for uncalled (N) bases. Apply the quality-mask rule: a mismatch inside the CASAVA-masked tail of both reads yields two arbitrary low-quality base calls and drives the score down, so masked-vs-unmasked comparisons use the unmasked probability when bases match and 1/4 when they do not. Aggregate per-base scores as the geometric mean over the full assembled sequence to compensate for variable lengths, then flag each sequence as accepted if score >= 0.9 and separately if >= 0.6, mirroring the paper's thresholds.

## Related tools

- **PANDAseq** (Assembles the paired-end reads and computes the overlap match probabilities, overall quality score, and threshold filtering implemented by this skill) — https://github.com/neufeld/pandaseq
- **PANDAseq-sam** (Extends PANDAseq quality-scored assembly to SAM/BAM-formatted input) — https://github.com/neufeld/pandaseq-sam
- **Illumina** (Sequencing platform whose paired-end reads, quality encoding, and CASAVA quality masks define the score inputs)

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq
```

## Evaluation signals

- quality_scores.tsv exists and contains one row per input pair with fields sequence_id, quality_score, accept_0.9, accept_0.6
- Every quality_score lies in (0, 1] (geometric mean of probabilities cannot leave this range)
- Each accept flag exactly equals the threshold comparison: accept_0.9 == (quality_score >= 0.9), accept_0.6 == (quality_score >= 0.6)
- Assemblies whose mismatches fall in the quality-masked regions of both reads score low and are discarded at p = 0.9
- Simulated/known-good pairs should assemble with scores greater than 0.9, as in the paper's 1 350 602 synthesized sequences

## Limitations

- Mismatches confined to the CASAVA-masked tails of both reads produce two arbitrary base calls and drive scores below the acceptance threshold, so otherwise valid assemblies can be discarded
- PANDAseq corrections are uncertain for reads with many low-quality bases
- The score alone does not catch errors PANDAseq itself introduces (~0.02-0.08% of output reads); length and uncalled-base constraints are checked separately

## Evidence

- [methods] Once the overlap is selected, the output sequence is constructed and an overall quality score is calculated.: "Once the overlap is selected, the output sequence is constructed and an overall quality score is calculated."
- [discussion] Low-score discards for masked-region mismatches: "incorrect assemblies have low quality scores, as these mismatches occur in quality-masked regions of both reads, and are discarded"
- [results] 0.9 threshold acceptance rate: "Of these, 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9."
- [results] 0.6 threshold comparison: "We assembled the same single-template data with a quality threshold of 0.6 and this increased the number of sequences assembled by 9%, yielding 652 249 sequences."
- [readme] README tool scope: "PANDASEQ is a program to align Illumina reads, optionally with PCR primers embedded in the sequence, and reconstruct an overlapping sequence."
