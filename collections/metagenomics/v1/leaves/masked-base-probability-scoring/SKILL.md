---
name: masked-base-probability-scoring
description: Use when auditing which overlap mismatches PANDAseq corrected probabilistically,
  to quantify whether uncertain corrections are driven by low-quality (masked) bases
  flagged by upstream Illumina processing.
when_to_use_negative:
- Data were not processed by CASAVA-style upstream quality masking (end-of-read lowest-quality
  masking convention), so the mask annotation is undefined.
- The input is raw paired-end FASTQ before assembly — corrections do not exist yet;
  run PANDAseq first.
- Sequences lack known truth (mixed-template data), so mismatched bases cannot be
  confidently identified as correction events.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: PANDAseq
  role: Probabilistic paired-end assembler whose overlap-mismatch corrections are
    classified; run with quality threshold 0.9 on the pure-template library to produce
    per-correction records.
  repo: https://github.com/neufeld/pandaseq
- name: Illumina
  role: Sequencing platform whose CASAVA quality scores and end-of-read masking convention
    define which bases are quality-masked.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:98a7faafbdb7afc7541565c44ac291101d49af0d0a16393d8af69a4a23c597d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - Illumina
  - PANDAseq
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# masked-base-probability-scoring

## Summary

Classifies PANDAseq's uncertain overlap-mismatch corrections by whether the conflicting bases were quality-masked (CASAVA end-of-read low-quality positions) in the forward read, reverse read, both, or neither, to test the claim that uncertain corrections come from reads with many low-quality bases.

## When to use

Use when auditing which overlap mismatches PANDAseq corrected probabilistically, to quantify whether uncertain corrections are driven by low-quality (masked) bases flagged by upstream Illumina processing.

## When NOT to use

- Data were not processed by CASAVA-style upstream quality masking (end-of-read lowest-quality masking convention), so the mask annotation is undefined.
- The input is raw paired-end FASTQ before assembly — corrections do not exist yet; run PANDAseq first.
- Sequences lack known truth (mixed-template data), so mismatched bases cannot be confidently identified as correction events.

## Inputs

- PANDAseq assembled sequences with per-correction records from a quality-threshold-0.9 run on a pure-template library
- Per-base CASAVA PHRED quality strings for the forward and reverse Illumina reads

## Outputs

- correction_mask_classification.csv with counts and fractions of corrections masked in both reads, exactly one read, or neither read

## How to apply

Load the assembled sequences and per-correction records from a PANDAseq run at a defined quality threshold (e.g. 0.9) on a pure-template library, so every overlap position has a known truth. For each corrected overlap position, decode the CASAVA PHRED quality character of the conflicting forward- and reverse-read bases into error probabilities via the 10^(-A/10) conversion. Annotate each conflicting base as quality-masked or not, using the CASAVA convention that end-of-read positions carry the lowest quality score as the mask. Assign each correction to one of three categories — masked in both reads, masked in exactly one read, or masked in neither read — compute the fraction in each category, and verify the fractions sum to 1 across all classified corrections. This distinguishes corrections where both bases were dubious (real uncertainty) from mixed-quality corrections (one high-quality, one low-quality base), which PANDAseq resolves probabilistically from the overlap.

## Related tools

- **PANDAseq** (Probabilistic paired-end assembler whose overlap-mismatch corrections are classified; run with quality threshold 0.9 on the pure-template library to produce per-correction records.) — https://github.com/neufeld/pandaseq
- **Illumina** (Sequencing platform whose CASAVA quality scores and end-of-read masking convention define which bases are quality-masked.)

## Examples

```
pandaseq -f control_R1.fastq.gz -r control_R2.fastq.gz -q 0.9 -B > corrected_overlap_stats.csv
```

## Evaluation signals

- Fractions of the three mask categories (both reads, exactly one read, neither read) sum to 1 across all classified corrections.
- Every correction's base pair is assigned to exactly one of the three categories (no unclassified or doubly-classified events).
- Expected result on the control library: only ~20% of mismatched bases were quality-masked in both reads, so the majority fall in the mixed-quality category.
- Quality-to-probability decoding spot-checks: PHRED character A must convert as 10^(-A/10) and end-of-read CASAVA positions must be annotated as masked.

## Limitations

- Uncertain corrections arise from reads with many low-quality bases; classification of individual corrections may not capture whole-read quality context.
- The mask convention depends on the CASAVA pipeline (end-of-read lowest quality score as the mask); other preprocessing pipelines will not have the same masking pattern.
- In the control library only 20% of mismatched bases were masked in both reads, so conclusions about both-masked corrections rest on a minority of events.
- Mismatches where one base is high-quality and one low-quality dominate, meaning the both-masked fraction is a small denominator and sensitive to a few events.

## Evidence

- [abstract] Uncertain error corrections come from reads with many low-quality bases identified by upstream processing.: "PANDAseq assembles paired-end reads rapidly and with the correction of most errors. Uncertain error corrections come from reads with many low-quality bases identified by upstream processing."
- [discussion] Masked-region mismatches drive quality scores.: "incorrect assemblies have low quality scores, as these mismatches occur in quality-masked regions of both reads, and are discarded"
- [methods] Quality-score-based mismatch probability scoring.: [evidence span withheld — anchor status: not_found]
- [results] Quality threshold 0.9 assembly filter.: "598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9"
