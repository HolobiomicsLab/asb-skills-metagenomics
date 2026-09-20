---
name: correction-fraction-tabulation
description: 'Use when auditing PANDAseq output from an Illumina paired-end run: you
  have per-correction records with overlapping conflicting bases and their PHRED quality
  strings, and you want to quantify what fraction of corrections involved quality-masked
  (end-of-read, lowest-score) bases in zero, one, or.'
when_to_use_negative:
- Input is a feature table or already-assembled contigs with no per-base PHRED quality
  strings — the mask classification cannot be decoded.
- Reads were base-called by a pipeline that does not follow the CASAVA convention
  of masking end-of-read positions with the lowest quality score.
- You are evaluating SeqPrep-style adapter-trimming or naive perfect-match overlap
  assembly where corrections are not emitted as per-position records.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3168
tools:
- name: PANDAseq
  role: Produces the assembled sequences and per-correction mismatch records at the
    specified quality threshold (e.g. 0.9) that are classified here
  repo: https://github.com/neufeld/pandaseq
- name: Illumina
  role: Source of the paired-end reads and CASAVA PHRED quality strings whose end-of-read
    masking defines the mask annotation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:8d901e538b8bf1227bb9764cd34c68ecbb570d438c0f913b22ef94f0d7ac9f3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - Illumina
  - PANDAseq
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# correction-fraction-tabulation

## Summary

Classifies PANDAseq's mismatch corrections in the overlap region by CASAVA quality-mask status of the conflicting forward/reverse bases, and tabulates the fraction of corrections falling in each mask category, testing whether uncertain error corrections come from reads with many low-quality bases.

## When to use

Use when auditing PANDAseq output from an Illumina paired-end run: you have per-correction records with overlapping conflicting bases and their PHRED quality strings, and you want to quantify what fraction of corrections involved quality-masked (end-of-read, lowest-score) bases in zero, one, or both reads.

## When NOT to use

- Input is a feature table or already-assembled contigs with no per-base PHRED quality strings — the mask classification cannot be decoded.
- Reads were base-called by a pipeline that does not follow the CASAVA convention of masking end-of-read positions with the lowest quality score.
- You are evaluating SeqPrep-style adapter-trimming or naive perfect-match overlap assembly where corrections are not emitted as per-position records.

## Inputs

- Assembled sequences and per-correction records from a quality-threshold-0.9 PANDAseq run on a pure-template/control library
- Raw forward and reverse FASTQ/QSEQ reads with CASAVA PHRED quality strings for the conflicting overlap bases

## Outputs

- correction_mask_classification.csv with counts and fractions of corrections in each of the three categories (masked in both, exactly one, or neither read)

## How to apply

Load the assembled sequences and per-correction records from a PANDAseq run at an assembly quality threshold (e.g. 0.9) and, for each corrected overlap position, decode the CASAVA PHRED quality character of the conflicting forward- and reverse-read bases into error probabilities via the 10^(-Q/10) conversion. Annotate each conflicting base as quality-masked or not, using the CASAVA convention that end-of-read positions carry the lowest quality score as the mask. Assign each correction to one of three categories — masked in both reads, masked in exactly one read, or masked in neither read — then compute the fraction of corrections per category and verify the fractions sum to 1. In the control library data this revealed that only 20% of mismatched bases were masked in both reads, i.e. most corrections paired one high-quality with one low-quality base, which supports the paper's abstract claim that uncertain corrections come from reads with many low-quality bases flagged by upstream processing.

## Related tools

- **PANDAseq** (Produces the assembled sequences and per-correction mismatch records at the specified quality threshold (e.g. 0.9) that are classified here) — https://github.com/neufeld/pandaseq
- **Illumina** (Source of the paired-end reads and CASAVA PHRED quality strings whose end-of-read masking defines the mask annotation)

## Examples

```
python -c "import pandas as pd; p=10**(-ord(q)/10.0); m=p>0.2; df.assign(cat=['both' if a and b else 'one' if a or b else 'none' for a,b in zip(m_f,m_r)]).to_csv('correction_mask_classification.csv')"
```

## Evaluation signals

- The three category fractions (masked in both, exactly one, neither) sum to exactly 1 across all classified corrections.
- For the control library at threshold 0.9, the 'masked in both reads' fraction should reproduce the reported value of ~20%, with the majority in the one-high/one-low-quality category.
- Each conflicting base's decoded error probability equals 10^(-PHRED/10) for the character in the original FASTQ quality string.
- Every correction in correction_mask_classification.csv traces back to a corrected overlap position in the PANDAseq per-correction records (no orphan rows).
- Mask flags agree with the CASAVA convention that end-of-read positions carry the lowest quality score.

## Limitations

- The ~20% figure was measured on the single-template control library at quality threshold 0.9; results may differ for mixed-template community data or other thresholds.
- The classification depends on the upstream CASAVA masking convention — other base-calling pipelines may not place the lowest quality scores at end-of-read positions.
- PANDAseq's abstract characterizes uncertain corrections as coming from reads with many low-quality bases, but a per-read aggregate (count of low-quality bases per read) is not directly computed by this tabulation, which is per-correction.

## Evidence

- [abstract] Uncertain error corrections come from reads with many low-quality bases identified by upstream processing.: "PANDAseq assembles paired-end reads rapidly and with the correction of most errors. Uncertain error corrections come from reads with many low-quality bases identified by upstream processing."
- [discussion] incorrect assemblies have low quality scores, as these mismatches occur in quality-masked regions of both reads, and are discarded: "incorrect assemblies have low quality scores, as these mismatches occur in quality-masked regions of both reads, and are discarded"
- [methods] To score alignments, we calculate the probability that the true nucleotides, and, are the same, given the observed nucleotides, X and Y.: [evidence span withheld — anchor status: not_found]
- [results] 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9: "Of these, 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9."
