---
name: amplicon-assembly-validation
description: Use when you have overlapping paired-end Illumina FASTQ/SAM reads (e.g.
  16S rRNA amplicons) and must merge them while probabilistically correcting uncalled
  or miscalled bases, then filter assemblies by quality score.
when_to_use_negative:
- Reads are not expected to overlap (e.g. non-overlapping paired-end libraries) —
  SeqPrep's README warns to save overlapping for libraries with known overlap; naive
  assembly or no merge is more appropriate
- Input is already an assembled contig or a post-assembly feature/OTU table — no overlap-assembly
  step remains to apply
- The platform is not Illumina or reads have high gap probability — PANDAseq exploits
  Illumina-specific properties including low probability of gap-inclusion
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: PANDAseq
  role: Probabilistic paired-end overlap assembler with error correction; primary
    tool for assembling FASTQ reads and applying quality/length/uncalled-base filters
  repo: https://github.com/neufeld/pandaseq
- name: PANDAseq-sam
  role: PANDAseq extension that runs the same assembly on SAM/BAM formatted input
  repo: https://github.com/neufeld/pandaseq-sam
- name: SeqPrep
  role: Alternative adapter-trimming/merging assembler using Needleman-Wunsch-style
    alignment scoring; comparison and fallback for adapter-present libraries
  repo: https://github.com/jstjohn/SeqPrep
- name: stitch
  role: Alternative multi-threaded FASTQ paired-end assembler; unmaintained, README
    recommends PANDAseq instead
  repo: https://github.com/audy/stitch
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:d523a85349a65a0186cb2c180e88b39ef09b5c7c04b23be22ea603337ca49b2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - PANDAseq
  - PANDAseq-sam
  - SeqPrep
  - stitch
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# amplicon-assembly-validation

## Summary

Probabilistically assemble paired-end Illumina 16S rRNA amplicon reads with PANDAseq, which determines the optimal overlap from quality-score-derived match probabilities, corrects mismatches and uncalled bases in the overlap, and rejects assemblies failing quality/length/uncalled-base criteria.

## When to use

Use when you have overlapping paired-end Illumina FASTQ/SAM reads (e.g. 16S rRNA amplicons) and must merge them while probabilistically correcting uncalled or miscalled bases, then filter assemblies by quality score.

## When NOT to use

- Reads are not expected to overlap (e.g. non-overlapping paired-end libraries) — SeqPrep's README warns to save overlapping for libraries with known overlap; naive assembly or no merge is more appropriate
- Input is already an assembled contig or a post-assembly feature/OTU table — no overlap-assembly step remains to apply
- The platform is not Illumina or reads have high gap probability — PANDAseq exploits Illumina-specific properties including low probability of gap-inclusion

## Inputs

- Paired-end Illumina FASTQ files (forward and reverse reads; CASAVA ASCII quality encoding, pre-1.8 A0 or 1.8 A1)
- SAM/BAM formatted paired-end reads (for pandaseq-sam)
- Optional primer sequences embedded in reads
- Quality-score threshold (e.g. 0.9 or 0.6), length bounds, and uncalled-base filter settings

## Outputs

- Assembled (joined) consensus sequences for each read pair with per-pair assembly quality scores
- Discarded/singletons output for read pairs with no suitable overlap or failing filters
- Assembly statistics (e.g. number of assembled sequences at a given quality threshold)

## How to apply

Run PANDAseq on forward and reverse FASTQ files (or SAM/BAM with pandaseq-sam); it first locates amplification primers if sequenced, then scores every candidate overlap c by computing Pr[F,R|c] from CASAVA quality values converted to error probabilities ε = 10^(-A/10), with match probability (1-εX)(1-εY) + εXεY/3, mismatch probability (1/3)(1-εX)εY + (1/3)(1-εY)εX + (2/9)εXεY, and 1/4 when a base is N. Select the overlap maximizing Pr[F,R|c], reconstruct the joined sequence keeping the higher-quality base at disagreements (N treated as always matching), and calculate an overall assembly quality score. Discard read pairs with no suitable overlap, and reject assemblies based on user-specified quality-score, length, or uncalled-base thresholds — the paper benchmarks with a 0.9 quality threshold (598,775 of 673,845 M. capsulatus reads assembled) and shows a 0.6 threshold recovers 9% more sequences (652,249). Validate by comparing reconstructed sequences against expected amplicons and confirming assembly quality scores meet the threshold.

## Related tools

- **PANDAseq** (Probabilistic paired-end overlap assembler with error correction; primary tool for assembling FASTQ reads and applying quality/length/uncalled-base filters) — https://github.com/neufeld/pandaseq
- **PANDAseq-sam** (PANDAseq extension that runs the same assembly on SAM/BAM formatted input) — https://github.com/neufeld/pandaseq-sam
- **SeqPrep** (Alternative adapter-trimming/merging assembler using Needleman-Wunsch-style alignment scoring; comparison and fallback for adapter-present libraries) — https://github.com/jstjohn/SeqPrep
- **stitch** (Alternative multi-threaded FASTQ paired-end assembler; unmaintained, README recommends PANDAseq instead) — https://github.com/audy/stitch

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq -t 0.9
```

## Evaluation signals

- Selected overlap reproduces the expected joined amplicon sequence for validation pairs
- Assembled sequences meet the paper's quality criterion: quality score >= 0.9 (all 1,350,602 simulated pairs and 598,775 of 673,845 M. capsulatus reads)
- Errors introduced by assembly stay in the 0.02-0.08% range reported for PANDAseq output
- Read pairs with no overlap exceeding the threshold are discarded rather than force-merged
- Number of assembled sequences at a threshold of 0.6 should exceed the 0.9-threshold count by roughly the paper's 9% margin for comparable data

## Limitations

- Uncertain error corrections come from reads with many low-quality bases identified by upstream processing
- Incorrect assemblies with mismatches in quality-masked regions of both reads can occur but carry low quality scores and are discarded at stringent thresholds
- A lower quality threshold (e.g. 0.6) increases recovery but admits more sequences with potential errors; threshold choice trades yield for accuracy
- PANDAseq assumes low probability of gap-inclusion, an Illumina-specific property; it is not a general-purpose gapped assembler

## Evidence

- [intro] PANDAseq uses paired-end Illumina reads to determine the proper amount of overlap and reconstructs the entire sequence by correcting errors in the overlapping region.: "determines the proper amount of overlap and reconstructs the entire sequence by correcting errors in the overlapping region"
- [results] Of 673,845 M. capsulatus reads, 598,775 assembled at quality >= 0.9.: "Of these, 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9."
- [methods] PANDAseq rejects sequences by quality score, length, or uncalled bases, and discards pairs with no suitable overlap.: "PANDAseq enables users to reject sequences based on low quality score, lengths that are too short or too long, or the presence of uncalled bases."
- [results] Only 0.02-0.08% of output reads contained errors introduced by assembly.: "only about 0.02-0.08% of output read contained errors introduced by the PANDAseq assembly process"
- [intro] PANDAseq exploits Illumina-specific properties including low gap-inclusion probability.: "PANDAseq makes use of Illumina-specific properties, including the low probability of gap-inclusion."
- [readme] Basic CLI usage is pandaseq -f forward.fastq -r reverse.fastq.: "pandaseq -f forward.fastq -r reverse.fastq"
