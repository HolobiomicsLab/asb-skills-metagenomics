---
name: overlap-mismatch-correction-classification
description: 'Use when auditing PANDAseq-corrected overlap mismatches: you have per-correction
  records plus per-base PHRED/CASAVA quality characters and need to quantify corrections
  that are masked in both, one, or neither read.'
when_to_use_negative:
- The reads have no per-base quality scores (e.g. FASTA-only input), since the mask
  classification depends entirely on PHRED quality characters
- Corrections are outside the paired-end overlap region or the library has no known
  ground truth, making the mask-status fractions uninterpretable
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: PANDAseq
  role: Assembles overlapping paired-end Illumina reads with probabilistic error correction;
    provides the per-correction mismatch records being classified
  repo: https://github.com/neufeld/pandaseq
- name: Illumina
  role: Source of the paired-end reads and CASAVA quality-mask convention (lowest
    quality score at end-of-read positions)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:109526be274cba44078153f1d311c5f42ee6a5b55c4c1d2c73547fdabc639808
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - Illumina
  - PANDAseq
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# overlap-mismatch-correction-classification

## Summary

Classifies each mismatch PANDAseq corrects in the paired-end overlap region by whether the conflicting forward and reverse bases were quality-masked (CASAVA low-quality end-of-read positions), to test the claim that uncertain error corrections come from reads with many low-quality bases.

## When to use

Use when auditing PANDAseq-corrected overlap mismatches: you have per-correction records plus per-base PHRED/CASAVA quality characters and need to quantify corrections that are masked in both, one, or neither read.

## When NOT to use

- The reads have no per-base quality scores (e.g. FASTA-only input), since the mask classification depends entirely on PHRED quality characters
- Corrections are outside the paired-end overlap region or the library has no known ground truth, making the mask-status fractions uninterpretable

## Inputs

- Assembled overlap sequences with per-correction records from a PANDAseq run (e.g. quality threshold 0.9 on a pure-template control library)
- Per-base CASAVA PHRED quality characters for forward and reverse Illumina FASTQ/QSEQ reads

## Outputs

- correction_mask_classification.csv containing counts and fractions of corrections masked in both reads, exactly one read, or neither read

## How to apply

Load the assembled sequences and per-correction records from a quality-threshold run (e.g. 0.9) of PANDAseq on the library of interest, such as the pure-template control library. For each corrected overlap position, decode the CASAVA PHRED quality character of the conflicting forward- and reverse-read bases into error probabilities using the 10^(-A/10) conversion. Annotate each conflicting base as quality-masked or not, using the CASAVA convention that end-of-read positions carry the lowest quality score as the mask. Assign each correction to one of three categories — masked in both reads, masked in exactly one read, or masked in neither read — and compute category fractions, verifying they sum to 1. The rationale is the paper's finding that incorrect assemblies are those with mismatches in quality-masked regions of both reads, while uncertain corrections in the control library were largely mixed-quality (only 20% masked in both reads).

## Related tools

- **PANDAseq** (Assembles overlapping paired-end Illumina reads with probabilistic error correction; provides the per-correction mismatch records being classified) — https://github.com/neufeld/pandaseq
- **Illumina** (Source of the paired-end reads and CASAVA quality-mask convention (lowest quality score at end-of-read positions))

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq  # then classify each overlap-mismatch correction by CASAVA quality-mask status of both bases
```

## Evaluation signals

- The three category fractions (masked in both, exactly one, neither) sum to 1 across all classified corrections
- Every corrected overlap position maps to a pair of decodable PHRED characters on both reads
- Masked positions correspond to end-of-read bases carrying the lowest CASAVA quality score
- Fractions reproduce the reported control-library split (only 20% of mismatched bases masked in both reads, majority one high- and one low-quality base)
- Output CSV contains both counts and fractions per category

## Limitations

- Uncertain corrections come from reads with many low-quality bases, so category assignments are probabilistic rather than definitive calls of which base is correct
- PANDAseq discards reads with no suitable overlap and rejects assemblies with uncalled bases or low quality scores, so corrections available for classification are a filtered subset
- The CASAVA mask convention (lowest quality at end-of-read) is instrument/pipeline-specific and may not apply to non-CASAVA quality encodings

## Evidence

- [abstract] Uncertain error corrections come from reads with many low-quality bases identified by upstream processing.: "PANDAseq assembles paired-end reads rapidly and with the correction of most errors. Uncertain error corrections come from reads with many low-quality bases identified by upstream processing."
- [methods] To score alignments, we calculate the probability that the true nucleotides are the same, given the observed nucleotides, X and Y.: [evidence span withheld — anchor status: not_found]
- [discussion] incorrect assemblies have low quality scores, as these mismatches occur in quality-masked regions of both reads, and are discarded: "incorrect assemblies have low quality scores, as these mismatches occur in quality-masked regions of both reads, and are discarded"
- [readme] pandaseq -f forward.fastq -r reverse.fastq: "The short version is:

	pandaseq -f forward.fastq -r reverse.fastq"
