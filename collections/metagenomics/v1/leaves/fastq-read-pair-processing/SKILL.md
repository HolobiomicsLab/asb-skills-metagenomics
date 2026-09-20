---
name: fastq-read-pair-processing
description: Use when input is paired-end Illumina FASTQ (or CASAVA/QSEQ) reads whose
  forward and reverse mates overlap an amplicon (e.g. 16S rRNA hypervariable regions)
  and reads are being discarded (~40%) due to uncalled or miscalled bases under naive
  perfect-match merging.
when_to_use_negative:
- Read pairs do not overlap (adapter-only libraries with little expected overlap)
  — the probabilistic overlap search is wasted and pairs will be discarded
- Single-end reads or long-read data where no paired-end overlap signal exists
- Data already processed by a paired-end assembler (e.g. already-merged contigs or
  a feature table)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0622
tools:
- name: PANDAseq
  role: 'Primary tool: scores all candidate overlaps probabilistically from quality
    values and reconstructs error-corrected merged reads'
  repo: https://github.com/neufeld/pandaseq
- name: PANDAseq-sam
  role: PANDAseq extension that runs the same overlap-based assembly on SAM/BAM formatted
    files
  repo: https://github.com/neufeld/pandaseq-sam
- name: SeqPrep
  role: Alternative adapter-trimming/merging tool that merges overlapping reads via
    Needleman-Wunsch-style alignment with strict mismatch-fraction cutoffs
  repo: https://github.com/jstjohn/SeqPrep
- name: Stitch
  role: Alternative multi-threaded FASTQ paired-end assembler (no longer maintained;
    README points users to PANDAseq)
  repo: https://github.com/audy/stitch
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:09a1acb6bf3195b96f72992acafeacb5f6b18a6925171f917c56008bb326b5d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - PANDAseq
  - PANDAseq-sam
  - SeqPrep
  - Stitch
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# fastq-read-pair-processing

## Summary

Probabilistically assembles overlapping Illumina paired-end FASTQ reads (as in PANDAseq) by scoring every candidate overlap with base-error probabilities derived from Phred quality values, selecting the maximum-likelihood overlap, and correcting mismatches and uncalled bases in the overlap region.

## When to use

Use when input is paired-end Illumina FASTQ (or CASAVA/QSEQ) reads whose forward and reverse mates overlap an amplicon (e.g. 16S rRNA hypervariable regions) and reads are being discarded (~40%) due to uncalled or miscalled bases under naive perfect-match merging.

## When NOT to use

- Read pairs do not overlap (adapter-only libraries with little expected overlap) — the probabilistic overlap search is wasted and pairs will be discarded
- Single-end reads or long-read data where no paired-end overlap signal exists
- Data already processed by a paired-end assembler (e.g. already-merged contigs or a feature table)

## Inputs

- Forward and reverse Illumina paired-end FASTQ files (or SAM/BAM via pandaseq-sam)
- Per-base Phred quality strings in Phred+33 (CASAVA 1.8) or Phred+64 (pre-1.8) encoding
- Optional PCR primer sequences embedded in reads (PANDAseq first locates amplification primers if specified)

## Outputs

- Assembled (merged) contig sequences per read pair with corrected bases in the overlap region
- Per-pair overlap probability and overall assembly quality score
- Discarded/singleton read pairs where no suitable overlap or filter criterion is met

## How to apply

Convert each read's CASAVA-encoded ASCII quality (Phred+64 pre-1.8, Phred+33 for CASAVA 1.8) into error probabilities ε = 10^(-A/10). For each candidate overlap c in [1, min(|F|,|R|)), compute Pr[F,R|c] as the product of per-base probabilities: matches score (1-εX)(1-εY) + εXεY/3, mismatches score (1/3)(1-εX)εY + (1/3)(1-εY)εX + (2/9)εXεY, and any uncalled N scores 1/4 so N bases always match. Select the overlap c maximizing Pr[F,R|c], then reconstruct the joined sequence keeping the higher-quality base at disagreements and treating N as matching; discard the pair if no overlap meets the threshold. Finally compute the overall assembly quality score and apply user filters (e.g. reject quality < 0.9, too-short/too-long lengths, or remaining uncalled bases). This approach exploits Illumina-specific properties such as the low probability of gap-inclusion, so no gap penalties are modeled.

## Related tools

- **PANDAseq** (Primary tool: scores all candidate overlaps probabilistically from quality values and reconstructs error-corrected merged reads) — https://github.com/neufeld/pandaseq
- **PANDAseq-sam** (PANDAseq extension that runs the same overlap-based assembly on SAM/BAM formatted files) — https://github.com/neufeld/pandaseq-sam
- **SeqPrep** (Alternative adapter-trimming/merging tool that merges overlapping reads via Needleman-Wunsch-style alignment with strict mismatch-fraction cutoffs) — https://github.com/jstjohn/SeqPrep
- **Stitch** (Alternative multi-threaded FASTQ paired-end assembler (no longer maintained; README points users to PANDAseq)) — https://github.com/audy/stitch

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq
```

## Evaluation signals

- For validation pairs with a known amplicon, the selected overlap must reproduce the expected joined sequence
- Assembled sequences meet the paper's quality criterion: assembly quality score >= 0.9 (598,775 of 673,845 M. capsulatus reads passed; simulated data all passed)
- Error rate of the assembler itself must be far below the error-correction rate (paper: ~0.02-0.08% of output reads had errors introduced, while 5-8% of sequences had errors corrected)
- Correct handling of encoding: Phred+64 vs Phred+33 quality strings must yield ε = 10^(-A/10) consistently; uncalled N bases must score 1/4 (always match), never penalize the alignment
- Recovered assembly counts should exceed naive perfect-match merging (paper: 4-50% more sequences, e.g. 652,249 vs baseline at quality threshold 0.6, a 9% increase)

## Limitations

- Uncertain error corrections arise from reads with many low-quality bases identified by upstream processing; the model cannot rescue reads that fail entirely
- Approximately 40% of sequences may be discarded due to uncalled or miscalled bases if the naive pipeline is used instead, and no suitable overlap leads to read-pair discarding
- PANDAseq makes use of Illumina-specific properties including the low probability of gap-inclusion, so it is not appropriate for platforms/reads with frequent indels
- The probabilistic scoring is multiplicative over the overlap; a very short or low-quality overlap region can yield ambiguous maximum-likelihood overlaps

## Evidence

- [intro] PANDAseq determines the proper amount of overlap and reconstructs the entire sequence by correcting errors in the overlapping region: "determines the proper amount of overlap and reconstructs the entire sequence by correcting errors in the overlapping region"
- [methods] Once the overlap is selected, the output sequence is constructed and an overall quality score is calculated: "Once the overlap is selected, the output sequence is constructed and an overall quality score is calculated."
- [results] Of these, 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9.: "Of these, 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9."
- [readme] PANDASEQ is a program to align Illumina reads, optionally with PCR primers embedded in the sequence, and reconstruct an overlapping sequence.: "PANDASEQ is a program to align Illumina reads, optionally with PCR primers embedded in the sequence, and reconstruct an overlapping sequence."
- [intro] approximately 40% of the sequences were discarded due to uncalled or miscalled bases: "approximately 40% of the sequences were discarded due to uncalled or miscalled bases."
