---
name: assembly-quality-score-thresholding
description: Use when PANDAseq has assembled Illumina overlapping paired-end (e.g.
  16S rRNA amplicon) reads and you must choose how many low-confidence assemblies
  to retain or discard via the assembly quality score cutoff.
when_to_use_negative:
- The reads do not overlap or are not paired-end Illumina reads matched by flow-cell
  template position — there is no overlap-region quality score to threshold.
- You need to filter raw unmerged reads by per-base Phred quality instead of an assembly-level
  quality score.
- The input is already an assembled or clustered sequence set (e.g. an OTU table);
  thresholding has already been applied upstream.
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: PANDAseq
  role: Assembles overlapping paired-end reads, computes the assembly quality score,
    and applies the user-specified quality threshold to retain or reject sequences.
  repo: https://github.com/neufeld/pandaseq
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:4cd4433db13297b14d29115abcac457c657fd427b17e5048d4ec35395e676135
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - PANDAseq
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# assembly-quality-score-thresholding

## Summary

Filter PANDAseq-assembled Illumina paired-end reads by their assembly quality score, using thresholds such as 0.9 for high-confidence output or relaxed 0.6 to increase sequence recovery. The threshold trades recall of assembled sequences against confidence that the overlap region was correctly reconstructed.

## When to use

Use when PANDAseq has assembled Illumina overlapping paired-end (e.g. 16S rRNA amplicon) reads and you must choose how many low-confidence assemblies to retain or discard via the assembly quality score cutoff.

## When NOT to use

- The reads do not overlap or are not paired-end Illumina reads matched by flow-cell template position — there is no overlap-region quality score to threshold.
- You need to filter raw unmerged reads by per-base Phred quality instead of an assembly-level quality score.
- The input is already an assembled or clustered sequence set (e.g. an OTU table); thresholding has already been applied upstream.

## Inputs

- Forward and reverse Illumina FASTQ files of overlapping paired-end reads (e.g. 673,845 paired-end 108-base M. capsulatus reads, or SRA024100 composite control library reads)
- PANDAseq STAT log lines from the assembly run (READS and per-thread filtering counts)

## Outputs

- FASTA/FASTQ of assembled sequences passing the chosen assembly quality score threshold (0.9 strict, 0.6 relaxed)
- Count of assembled sequences and computed coverage fraction (assembled / input read pairs), e.g. 598,775 of 673,845 at threshold 0.9

## How to apply

Run PANDAseq on paired FASTQ inputs (e.g. 'pandaseq -f forward.fastq -r reverse.fastq'), letting the assembler construct the output sequence and calculate an overall assembly quality score from the probability that the two reads' nucleotides agree in the overlap. Filter with a strict threshold of 0.9 as the default: on the M. capsulatus single-template library (673,845 paired-end 108-base reads), 598,775 sequences assembled at score >= 0.9. When maximal recovery matters more than maximum confidence, lower the threshold to 0.6 — this raised recovery by 9% on the same library (652,249 sequences) and gave up to a 50% increase over naive perfect-overlap assembly on the pooled control library SRA024100, with 85% gains in abundant clusters. Count retained sequences by parsing the STAT log lines and compute coverage as assembled output divided by input read pairs; count uncertain error corrections as coming from reads with many low-quality bases identified by upstream processing.

## Related tools

- **PANDAseq** (Assembles overlapping paired-end reads, computes the assembly quality score, and applies the user-specified quality threshold to retain or reject sequences.) — https://github.com/neufeld/pandaseq

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq
```

## Evaluation signals

- At threshold 0.9, coverage should be roughly 598,775/673,845 (~88–96% of the M. capsulatus dataset); deviations of more than a few percent suggest a misconfigured threshold or wrong input.
- Lowering the threshold from 0.9 to 0.6 must increase the assembled count (expect ~9% more on the single-template library: 652,249 sequences); a decrease indicates a parameter error.
- On the pooled control library SRA024100, threshold 0.9 should yield only a small gain (~3.9%) over naive perfect-overlap assembly, while 0.6 should yield up to ~50% overall and ~85% within abundant clusters.
- The recorded wall-clock time should be on the order of minutes (reported ~2 min 25 s) for ~673k read pairs on a desktop computer.
- Assembled output count must never exceed the number of input read pairs (one assembled sequence per pair).

## Limitations

- Strict 0.9 thresholding discards many pairs; naive perfect-overlap assembly discarded ~40% of sequences due to uncalled or miscalled bases, and even PANDAseq at 0.9 leaves ~11% of the single-template library unassembled.
- Uncertain error corrections come from reads with many low-quality bases, so relaxed thresholds (0.6) retain more sequences at the risk of incorporating errors into the overlap.
- Optimal threshold depends on the library: a 0.6 threshold gave 9% more sequences on the single-template library but up to 50% over naive assembly on the composite control library, so the gain is library-dependent.

## Evidence

- [results] strict 0.9 default: "598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9"
- [results] relaxed 0.6 recovery gain: "We assembled the same single-template data with a quality threshold of 0.6 and this increased the number of sequences assembled by 9%, yielding 652 249 sequences."
- [methods] user-controlled rejection criteria: "PANDAseq enables users to reject sequences based on low quality score, lengths that are too short or too long, or the presence of uncalled bases."
- [discussion] rationale for score-based discarding: "incorrect assemblies have low quality scores, as these mismatches occur in quality-masked regions of both reads, and are discarded"
- [abstract] yield over naive assembly: "Assembly of control libraries showed a 4-50% increase in the number of assembled sequences over naïve assembly with"
