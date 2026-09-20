---
name: illumina-phred-quality-decoding
description: Use when you need to characterize which reads PANDAseq's error corrections
  come from — e.g. testing whether uncertain corrections concentrate in reads with
  many low-quality (CASAVA-masked) bases rather than high-quality positions.
when_to_use_negative:
- Input is already an assembled feature table or count matrix with no per-base quality
  strings — quality decoding cannot be performed
- Reads were not produced by Illumina/CASAVA processing with the end-of-read lowest-quality
  mask convention, so the masking annotation rule does not apply
- The goal is assembly itself (overlap detection and sequence reconstruction), not
  post-hoc characterization of where corrections occur
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: PANDAseq
  role: Produces the assembled sequences and per-correction records at quality threshold
    0.9 whose mismatch quality context is decoded and classified
  repo: https://github.com/neufeld/pandaseq
- name: Illumina
  role: Source platform whose CASAVA quality-score masking convention (end-of-read
    lowest quality) defines the 'quality-masked' annotation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:054e2cfc07502e7af3053b1fc1276dc72b6db5f2d8da41290e5e9c4e090ac37c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - Illumina
  - PANDAseq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# illumina-phred-quality-decoding

## Summary

Decodes Illumina CASAVA PHRED quality characters of conflicting forward/reverse bases at corrected overlap positions into error probabilities (10^(-A/10)), then classifies each mismatched base pair by whether it was quality-masked in neither, one, or both reads. Used to test PANDAseq's claim that uncertain error corrections come from reads with many low-quality bases identified by upstream processing.

## When to use

Use when you need to characterize which reads PANDAseq's error corrections come from — e.g. testing whether uncertain corrections concentrate in reads with many low-quality (CASAVA-masked) bases rather than high-quality positions.

## When NOT to use

- Input is already an assembled feature table or count matrix with no per-base quality strings — quality decoding cannot be performed
- Reads were not produced by Illumina/CASAVA processing with the end-of-read lowest-quality mask convention, so the masking annotation rule does not apply
- The goal is assembly itself (overlap detection and sequence reconstruction), not post-hoc characterization of where corrections occur

## Inputs

- Assembled sequences and per-correction records from a PANDAseq run at quality threshold 0.9 (task_001.expected_outputs[0], pure-template control library)
- Illumina paired-end FASTQ quality strings (CASAVA PHRED characters) for the conflicting forward- and reverse-read bases

## Outputs

- correction_mask_classification.csv containing counts and fractions of corrections classified as masked in both reads, exactly one read, or neither read (fractions must sum to 1)

## How to apply

Load the assembled sequences and per-correction records from a quality-threshold-0.9 PANDAseq run (e.g. on a pure-template control library). For each corrected overlap position, decode the CASAVA PHRED quality character of the conflicting forward- and reverse-read bases into error probabilities using the 10^(-A/10) conversion. Annotate each conflicting base as quality-masked or not, applying the CASAVA convention that end-of-read positions carry the lowest quality score as the mask. Assign each correction to one of three categories — masked in both reads, masked in exactly one read, or masked in neither — then compute and verify the category fractions sum to 1. This exploits the PANDAseq model in which mismatch probability is scored from the paired quality scores, so corrections involving one high-quality and one low-quality base dominate rather than both-low-quality cases.

## Related tools

- **PANDAseq** (Produces the assembled sequences and per-correction records at quality threshold 0.9 whose mismatch quality context is decoded and classified) — https://github.com/neufeld/pandaseq
- **Illumina** (Source platform whose CASAVA quality-score masking convention (end-of-read lowest quality) defines the 'quality-masked' annotation)

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq
```

## Evaluation signals

- The three category fractions (masked in both, exactly one, neither) sum to 1 across all classified corrections
- In the control library data, the masked-in-both-reads fraction should be ~20%, with the majority of corrections involving one high-quality and one low-quality base
- Decoded error probabilities match 10^(-A/10) for the CASAVA PHRED character of each conflicting base
- End-of-read positions are consistently flagged as masked per the CASAVA convention before classification
- correction_mask_classification.csv contains both counts and fractions for all three categories

## Limitations

- Only ~20% of mismatched bases in the control library were quality-masked in both reads, so this skill does not support a blanket assumption that all uncertain corrections are both-low-quality cases
- Masking is inferred from the CASAVA end-of-read lowest-quality convention, which may not capture other upstream masking schemes
- Corrections to uncalled bases and mismatched bases behave differently in PANDAseq scoring; the analysis targets mismatched overlap positions

## Evidence

- [abstract] Uncertain error corrections come from reads with many low-quality bases identified by upstream processing.: "PANDAseq assembles paired-end reads rapidly and with the correction of most errors. Uncertain error corrections come from reads with many low-quality bases identified by upstream processing."
- [discussion] incorrect assemblies have low quality scores, as these mismatches occur in quality-masked regions of both reads, and are discarded: "incorrect assemblies have low quality scores, as these mismatches occur in quality-masked regions of both reads, and are discarded"
- [methods] calculate probability that true nucleotides match using quality scores: [evidence span withheld — anchor status: not_found]
