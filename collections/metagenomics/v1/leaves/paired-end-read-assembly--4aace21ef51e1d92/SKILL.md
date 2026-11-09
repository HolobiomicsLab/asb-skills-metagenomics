---
name: paired-end-read-assembly
description: Use when Illumina paired-end FASTQ (or SAM/BAM) amplicon reads overlap
  and must be merged into full-length sequences with mismatch/uncalled-base correction,
  e.g., 16S rRNA libraries where naive assembly discards ~40% of reads.
when_to_use_negative:
- Input reads do not overlap (fragment length exceeds forward+reverse read span) —
  no suitable overlap exists and all pairs will be discarded
- Input is already an assembled contig or a feature/OTU table rather than raw paired-end
  reads
- Quality information is absent or unusable (e.g., FASTA-only input), since PANDAseq's
  probabilistic error correction relies on per-base quality scores
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: PANDAseq
  role: Primary probabilistic paired-end assembler; finds overlap and corrects errors
    using quality scores
  repo: https://github.com/neufeld/pandaseq
- name: PANDAseq-sam
  role: PANDAseq variant accepting SAM/BAM formatted paired-end input
  repo: https://github.com/neufeld/pandaseq-sam
- name: SeqPrep
  role: Alternative overlapping paired-end merger used for comparison
  repo: https://github.com/jstjohn/SeqPrep
- name: stitch
  role: Alternative paired-end assembler (no longer maintained; README directs users
    to PANDAseq)
  repo: https://github.com/audy/stitch
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:9b1ef9d1aaa92c6b4285a418c66334d260c3a092264415ffd42195f180697f57
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - PANDAseq
  - PANDAseq-sam
  - SeqPrep
  - stitch
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.55
n_claims: 20
---

# paired-end-read-assembly

## Summary

Assemble overlapping Illumina paired-end amplicon reads (e.g., 16S rRNA V3) into a single error-corrected sequence using PANDAseq, which determines primer locations, identifies the optimal overlap, and reconstructs the sequence probabilistically from quality scores.

## When to use

Use when Illumina paired-end FASTQ (or SAM/BAM) amplicon reads overlap and must be merged into full-length sequences with mismatch/uncalled-base correction, e.g., 16S rRNA libraries where naive assembly discards ~40% of reads.

## When NOT to use

- Input reads do not overlap (fragment length exceeds forward+reverse read span) — no suitable overlap exists and all pairs will be discarded
- Input is already an assembled contig or a feature/OTU table rather than raw paired-end reads
- Quality information is absent or unusable (e.g., FASTA-only input), since PANDAseq's probabilistic error correction relies on per-base quality scores

## Inputs

- Paired-end Illumina FASTQ files (forward and reverse; e.g., 108-base 16S rRNA reads)
- Optional SAM/BAM formatted read files (via pandaseq-sam)
- Optional amplification primer sequences embedded in reads

## Outputs

- Assembled consensus sequences passing the quality-score threshold
- Assembler STAT log lines (input READS and per-thread filtering counts) for coverage accounting
- Discarded read pairs (no suitable overlap or failing validation criteria)

## How to apply

Run PANDAseq (`pandaseq -f forward.fastq -r reverse.fastq`), which first locates amplification primers if present, then identifies the optimal overlap between read pairs, and finally reconstructs the complete sequence while correcting errors and checking length and quality constraints. Alignments are scored by calculating the probability that the true nucleotides match given the observed bases and their PHRED quality scores (error probability ε = 10^(-A/10)). Apply a default assembly quality-score threshold of 0.9; if recovery is the priority, lower the threshold to 0.6 — on the M. capsulatus 673,845-pair single-template library this raised output from 598,775 to 652,249 sequences (a 9% increase). PANDAseq discards read pairs for which no suitable overlap is found, and rejects assemblies failing user-specified criteria such as low quality score, length limits, or the presence of uncalled bases. For SAM/BAM input, use the pandaseq-sam variant with the same quality logic.

## Related tools

- **PANDAseq** (Primary probabilistic paired-end assembler; finds overlap and corrects errors using quality scores) — https://github.com/neufeld/pandaseq
- **PANDAseq-sam** (PANDAseq variant accepting SAM/BAM formatted paired-end input) — https://github.com/neufeld/pandaseq-sam
- **SeqPrep** (Alternative overlapping paired-end merger used for comparison) — https://github.com/jstjohn/SeqPrep
- **stitch** (Alternative paired-end assembler (no longer maintained; README directs users to PANDAseq)) — https://github.com/audy/stitch

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq
```

## Evaluation signals

- On the M. capsulatus benchmark (673,845 paired-end 108-base reads), the assembled count at threshold 0.9 should be ~598,775, i.e., ~95.5% of input pairs
- Lowering the threshold to 0.6 on the same library should increase recovery to ~652,249 sequences (a 9% increase)
- On the simulated 1,350,602-pair error-free dataset with real Illumina error masks, all pairs should assemble with quality scores greater than 0.9
- On the pooled control library (SRA024100), PANDAseq at threshold 0.9 should yield roughly 3.9% more assembled sequences than naive perfect-overlap assembly, rising to ~50% overall (85% in abundant clusters) at threshold 0.6
- Verify error rates: only ~0.02–0.08% of output reads should contain errors introduced by the PANDAseq assembly process

## Limitations

- PANDAseq output the smallest fraction of sequences among the assemblers compared in the benchmark (95.5% of the M. capsulatus dataset), reflecting its quality-based filtering stringency
- Uncertain error corrections come from reads with many low-quality bases; the per-read error probability ε is estimated from mismatch counts in known index tags of a defined community dataset
- End-of-read quality scores may be masked to the lowest score under the CASAVA convention, affecting overlap confidence at read ends
- Incorrect assemblies with mismatches in quality-masked regions of both reads get low quality scores and are discarded

## Evidence

- [abstract] PANDAseq assembles paired-end reads rapidly and with the correction of most errors.: "PANDAseq assembles paired-end reads rapidly and with the correction of most errors."
- [methods] workflow definition: "First, it determines the locations of the amplification primers, if they are specified and were sequenced."
- [results] quality threshold sensitivity: "We assembled the same single-template data with a quality threshold of 0.6 and this increased the number of sequences assembled by 9%, yielding 652 249 sequences."
- [methods] read pair discard rule: "If no suitable overlap is found, then the read pair is discarded."
- [readme] README usage: "The short version is:

	pandaseq -f forward.fastq -r reverse.fastq"
