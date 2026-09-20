---
name: quality-mask-region-annotation
description: Use when you have PANDAseq per-correction records plus original FASTQ
  quality strings and need to quantify how many mismatched bases were quality-masked
  by upstream (CASAVA) processing rather than genuinely conflicting.
when_to_use_negative:
- You only have the assembled consensus FASTQ and not the original per-read quality
  strings — the mask classification cannot be recovered.
- Your goal is to assess overall assembly error rate (0.02-0.08% introduced errors)
  rather than the provenance of uncertain corrections.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: PANDAseq
  role: Produces the overlapping assembled sequence and mismatch/error-correction
    records whose quality-mask status is annotated
  repo: https://github.com/neufeld/pandaseq
- name: Illumina
  role: Upstream platform whose CASAVA quality-score masking convention (lowest quality
    at end-of-read positions) defines the annotation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:ddf659628a47b8963e30f20fa630f61e054d42e984dfeedf1e862521de21d060
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - Illumina
  - PANDAseq
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# quality-mask-region-annotation

## Summary

Classifies each PANDAseq-corrected mismatched base pair by whether its CASAVA PHRED quality character falls in the end-of-read quality-masked region of the forward read, the reverse read, both, or neither, to test whether uncertain error corrections concentrate in reads with many low-quality bases.

## When to use

Use when you have PANDAseq per-correction records plus original FASTQ quality strings and need to quantify how many mismatched bases were quality-masked by upstream (CASAVA) processing rather than genuinely conflicting.

## When NOT to use

- You only have the assembled consensus FASTQ and not the original per-read quality strings — the mask classification cannot be recovered.
- Your goal is to assess overall assembly error rate (0.02-0.08% introduced errors) rather than the provenance of uncertain corrections.

## Inputs

- Assembled sequences from a PANDAseq run at quality threshold 0.9 (e.g. task_001.expected_outputs[0] pure-template library output)
- Per-correction records for each corrected overlap position
- Original CASAVA FASTQ/QSEQ quality strings for the forward and reverse reads

## Outputs

- correction_mask_classification.csv with category counts and fractions for masked-in-both, masked-in-exactly-one, and masked-in-neither corrections

## How to apply

Load the assembled sequences and per-correction records from a quality-threshold-0.9 PANDAseq run (e.g. on the pure-template library). For each corrected overlap position, decode the CASAVA PHRED quality character of the conflicting forward- and reverse-read bases into error probabilities via the 10^(-A/10) conversion. Annotate each conflicting base as quality-masked or not, using the CASAVA convention that end-of-read positions carry the lowest quality score as the mask. Assign each correction to one of three categories — masked in both reads, masked in exactly one read, masked in neither read — compute the fraction in each, and verify the fractions sum to 1. In the control library data this showed only 20% of mismatched bases were masked in both reads, so most corrections involved one high-quality and one low-quality base.

## Related tools

- **PANDAseq** (Produces the overlapping assembled sequence and mismatch/error-correction records whose quality-mask status is annotated) — https://github.com/neufeld/pandaseq
- **Illumina** (Upstream platform whose CASAVA quality-score masking convention (lowest quality at end-of-read positions) defines the annotation)

## Examples

```
python classify_masks.py --corrections pandaseq_q0.9_corrections.tsv --fastq-fwd fwd.fastq --fastq-rev rev.fastq --out correction_mask_classification.csv
```

## Evaluation signals

- Fractions across the three mask categories (both / exactly one / neither) sum to 1 over all classified corrections.
- Every correction record maps to a resolvable overlap position with decodable PHRED characters in both original reads.
- Fraction masked in both reads reproduces the expected ballpark (20% in the control library data).
- Decoded error probabilities are consistent with the 10^(-A/10) conversion applied to the CASAVA PHRED characters.
- End-of-read positions are consistently flagged as the quality mask per the CASAVA convention.

## Limitations

- Uncertain error corrections come from reads with many low-quality bases, so the mask categories are not independent of overall read quality.
- The CASAVA mask is only defined at end-of-read positions carrying the lowest quality score; mid-read low-quality bases are not masked by upstream processing.
- Only overlap-region mismatches that PANDAseq chose to correct are classified; discarded read pairs with no suitable overlap are excluded.

## Evidence

- [abstract] Uncertain error corrections come from reads with many low-quality bases identified by upstream processing.: "Uncertain error corrections come from reads with many low-quality bases identified by upstream processing."
- [discussion] Masked-region mismatches drive incorrect assemblies: "incorrect assemblies have low quality scores, as these mismatches occur in quality-masked regions of both reads, and are discarded"
- [methods] Quality-score based alignment scoring: [evidence span withheld — anchor status: not_found]
