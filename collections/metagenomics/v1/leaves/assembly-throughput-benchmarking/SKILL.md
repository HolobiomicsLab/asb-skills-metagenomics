---
name: assembly-throughput-benchmarking
description: Use when you need to measure how fast a paired-end assembler processes
  a large Illumina FASTQ dataset and what fraction of input read pairs it recovers,
  e.g. to compare PANDAseq against alternative assemblers.
when_to_use_negative:
- Input reads are not overlapping Illumina paired-end reads (e.g. non-overlapping
  or single-end libraries), where overlap-based assembly and its coverage metric do
  not apply
- The goal is assembly accuracy/error profiling rather than throughput and recovery
  benchmarking (accuracy requires the 0.02–0.08% introduced-error analysis, not timing)
- Working with SAM/BAM formatted input, which requires the pandaseq-sam variant rather
  than the FASTQ short form
edam_operation: http://edamontology.org/operation_3230
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: PANDAseq
  role: Paired-end assembler under benchmark; run in short form on forward/reverse
    FASTQ with quality threshold 0.9, timed with a shell timer and output parsed from
    STAT log lines
  repo: https://github.com/neufeld/pandaseq
- name: SeqPrep
  role: Alternative overlapping paired-end merger against which assembler recovery/throughput
    is compared
  repo: https://github.com/jstjohn/SeqPrep
- name: stitch
  role: Alternative multi-threaded paired-end assembler used as a comparison point
    for recovery fraction
  repo: https://github.com/audy/stitch
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:4d9e25e4eabf6e049498559ec32b434f4ee315e56c41b8664ee55c4a37dc5416
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - PANDAseq
  - SeqPrep
  - stitch
  edam_operation: http://edamontology.org/operation_3230
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# assembly-throughput-benchmarking

## Summary

Benchmark the wall-clock speed and read-recovery (coverage) fraction of an Illumina paired-end assembler, using PANDAseq on the Methylococcus capsulatus 16S rRNA single-template library (673,845 paired-end 108-base reads) as the reference workload.

## When to use

Use when you need to measure how fast a paired-end assembler processes a large Illumina FASTQ dataset and what fraction of input read pairs it recovers, e.g. to compare PANDAseq against alternative assemblers.

## When NOT to use

- Input reads are not overlapping Illumina paired-end reads (e.g. non-overlapping or single-end libraries), where overlap-based assembly and its coverage metric do not apply
- The goal is assembly accuracy/error profiling rather than throughput and recovery benchmarking (accuracy requires the 0.02–0.08% introduced-error analysis, not timing)
- Working with SAM/BAM formatted input, which requires the pandaseq-sam variant rather than the FASTQ short form

## Inputs

- Forward and reverse FASTQ files of overlapping Illumina paired-end reads (M. capsulatus ATCC 33009 16S rRNA library, 673,845 paired-end 108-base reads)
- Assembly quality threshold (0.9 for the reported benchmark; 0.6 sensitivity variant)

## Outputs

- Wall-clock execution time of the assembler run (reference: 2 minutes 25 seconds)
- Count of assembled output sequences parsed from STAT log lines (reference: 598,775 at threshold 0.9)
- Coverage fraction = assembled output sequences / input read pairs (reference: 95.5%)

## How to apply

Download and unpack the M. capsulatus paired-end sample data (673,845 paired-end 108-base 16S rRNA reads from pandaseq_sampledata.tar, NCBI SRA accession SRA024100) to obtain forward and reverse FASTQ files. Run PANDAseq in its short form ('pandaseq -f forward.fastq -r reverse.fastq') with an assembly quality threshold of 0.9, timing total wall-clock execution with a shell timer; the reported runtime is 2 minutes 25 seconds. Parse the assembler's log 'STAT' lines (e.g. 'STAT READS' for input read pairs and per-thread STAT filtering counts) to count assembled output sequences. Compute coverage as output divided by input pairs — the reported expectation is 598,775 of 673,845 pairs assembled at threshold 0.9, i.e. 95.5% of the dataset (the smallest fraction among the assemblers compared). Lowering the threshold to 0.6 increases recovery by 9% to 652,249 sequences, so threshold choice is a key decision point in the coverage benchmark.

## Related tools

- **PANDAseq** (Paired-end assembler under benchmark; run in short form on forward/reverse FASTQ with quality threshold 0.9, timed with a shell timer and output parsed from STAT log lines) — https://github.com/neufeld/pandaseq
- **SeqPrep** (Alternative overlapping paired-end merger against which assembler recovery/throughput is compared) — https://github.com/jstjohn/SeqPrep
- **stitch** (Alternative multi-threaded paired-end assembler used as a comparison point for recovery fraction) — https://github.com/audy/stitch

## Examples

```
time pandaseq -f forward.fastq -r reverse.fastq -t 0.9 2> pandaseq.log; grep 'STAT' pandaseq.log
```

## Evaluation signals

- Wall-clock runtime is of the same order of magnitude as the reported 2 minutes 25 seconds on the M. capsulatus library
- STAT log 'READS' count reports 673,845 input read pairs
- Computed coverage fraction (assembled / input pairs) is within a few percent of 95.5% at threshold 0.9
- Assembled output count is 598,775 at threshold 0.9; re-running at threshold 0.6 should recover ~652,249 sequences (a ~9% increase)
- Sequences discarded at threshold 0.9 are largely explained by uncalled bases and low-quality assemblies, consistent with ~40% naive discard rates cited in the article

## Limitations

- PANDAseq's higher stringency yields the smallest assembled fraction among the compared assemblers (95.5% at threshold 0.9); coverage is sensitive to the quality threshold (0.6 recovers 9% more)
- No PANDAseq software version number is provided in the article, complicating exact reproduction of the reported runtime
- Uncertain error corrections come from reads with many low-quality bases identified by upstream processing, so coverage/quality trade-offs depend on input read quality

## Evidence

- [results] Sequencing produced 673 845 paired-end 108-base reads, available at http://neufeldserver.uwaterloo.ca/~apmasell/pandaseq_sampledata.tar.: "Sequencing produced 673 845 paired-end 108-base reads, available at http://neufeldserver.uwaterloo.ca/~apmasell/pandaseq_sampledata.tar."
- [results] Of these, 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9.: "Of these, 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9."
- [results] We assembled the same single-template data with a quality threshold of 0.6 and this increased the number of sequences assembled by 9%, yielding 652 249 sequences.: "We assembled the same single-template data with a quality threshold of 0.6 and this increased the number of sequences assembled by 9%, yielding 652 249 sequences."
- [readme] The short version is:

	pandaseq -f forward.fastq -r reverse.fastq: "The short version is:

	pandaseq -f forward.fastq -r reverse.fastq"
- [intro] Assembly is extremely fast and millions of paired-end reads can be rapidly assembled on a desktop computer.: "Assembly is extremely fast and millions of paired-end reads can be rapidly assembled on a desktop computer."
