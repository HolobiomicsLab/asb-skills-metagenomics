---
name: 16s-rrna-amplicon-count-comparison
description: Use when assembling overlapping paired-end Illumina 16S rRNA amplicon
  reads (e.g. 108-base M. capsulatus pairs) and deciding whether lowering the PANDAseq
  quality threshold from 0.9 to a looser cutoff (e.g. 0.6) recovers more sequences.
when_to_use_negative:
- Input reads have already been assembled into a single contig per pair — re-merging
  and re-thresholding would change the underlying sequence set, not test the threshold
  effect.
- The library is not expected to contain overlapping paired-end reads (PANDAseq discards
  pairs with no suitable overlap), so count comparisons between thresholds would not
  be meaningful.
- Goal is de novo assembly of long reads (e.g. whole-genome data) rather than assembling
  overlapping short paired-end Illumina amplicon reads.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: PANDAseq
  role: Primary assembler used to assemble overlapping paired-end reads at quality-score
    thresholds 0.9 and 0.6 for the count comparison
  repo: https://github.com/neufeld/pandaseq
- name: SeqPrep
  role: Alternative overlapping paired-end merger cited for comparison against PANDAseq's
    assembly counts
  repo: https://github.com/jstjohn/SeqPrep
- name: stitch
  role: Alternative paired-end assembler (now unmaintained) referenced in the comparison
    of assembled sequence yields
  repo: https://github.com/audy/stitch
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:f7b31d0937b4770a88d9fb34c260aedd45263e6fc3de4d5ec3bafd84e8f30338
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - PANDAseq
  - SeqPrep
  - stitch
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# 16s-rrna-amplicon-count-comparison

## Summary

Quantifies how the PANDAseq assembly quality-score threshold changes sequence recovery for a paired-end Illumina 16S rRNA amplicon library, using the M. capsulatus single-template benchmark (598,775 reads at threshold 0.9 vs 652,249 at 0.6).

## When to use

Use when assembling overlapping paired-end Illumina 16S rRNA amplicon reads (e.g. 108-base M. capsulatus pairs) and deciding whether lowering the PANDAseq quality threshold from 0.9 to a looser cutoff (e.g. 0.6) recovers more sequences.

## When NOT to use

- Input reads have already been assembled into a single contig per pair — re-merging and re-thresholding would change the underlying sequence set, not test the threshold effect.
- The library is not expected to contain overlapping paired-end reads (PANDAseq discards pairs with no suitable overlap), so count comparisons between thresholds would not be meaningful.
- Goal is de novo assembly of long reads (e.g. whole-genome data) rather than assembling overlapping short paired-end Illumina amplicon reads.

## Inputs

- Paired-end FASTQ files (forward.fastq, reverse.fastq) from the M. capsulatus 16S rRNA amplicon library (673,845 paired-end 108-base reads; NCBI SRA accession SRA024100 / pandaseq_sampledata.tar)
- PANDAseq installation built from the neufeld/pandaseq repository (needs zlib and libbz2 development packages)

## Outputs

- assembled_sequence_counts.csv with one row per quality threshold (0.9 -> 598,775; 0.6 -> 652,249)
- Percent increase in recovered sequences between thresholds (expected ~9%)

## How to apply

Download the PANDAseq sample-data archive containing the 673,845 paired-end 108-base Methylococcus capsulatus reads and assemble them with pandaseq using the default quality threshold of 0.9, recording the count of assembled sequences (expected 598,775). Re-run the identical command with the threshold lowered to 0.6 and record the count again (expected 652,249, a 9% increase). The rationale is that PANDAseq's probabilistic error correction lets marginal read pairs assemble, so the 0.6 threshold recovers ~9% more sequences than the 0.9 cutoff, which discards roughly 40% of reads due to uncalled or miscalled bases at stricter settings. Write both counts to assembled_sequence_counts.csv and compare each value against the published figure, treating a count above the 0.9-threshold value as the expected direction of the effect. Judge success by reproducing both counts and by confirming the percent increase (652,249 / 598,775 ≈ 9%).

## Related tools

- **PANDAseq** (Primary assembler used to assemble overlapping paired-end reads at quality-score thresholds 0.9 and 0.6 for the count comparison) — https://github.com/neufeld/pandaseq
- **SeqPrep** (Alternative overlapping paired-end merger cited for comparison against PANDAseq's assembly counts) — https://github.com/jstjohn/SeqPrep
- **stitch** (Alternative paired-end assembler (now unmaintained) referenced in the comparison of assembled sequence yields) — https://github.com/audy/stitch

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq -A 598775_assembled.fa > assembled_0.9_counts.txt && pandaseq -f forward.fastq -r reverse.fastq -T 0.6 -A 652249_assembled.fa > assembled_0.6_counts.txt
```

## Evaluation signals

- Count at quality threshold 0.9 must match the published 598,775 assembled sequences from the 673,845 read pairs.
- Count at quality threshold 0.6 must match the published 652,249 assembled sequences.
- Percent increase between the two runs should be ~9% (652,249 / 598,775).
- The count at the lower threshold must exceed the count at the stricter threshold (monotonic recovery with looser quality cutoffs).
- Re-running the same FASTQ inputs with the same threshold must reproduce identical counts (determinism check).

## Limitations

- No PANDAseq version number is recorded in the article or repository changelog, so exact reproduction across versions is not guaranteed.
- Lowering the threshold increases yield but admits lower-quality assemblies; the article notes uncertain error corrections come from reads with many low-quality bases, and ~0.02–0.08% of output reads contained errors introduced by the assembly process.
- Read pairs for which no suitable overlap is found are discarded regardless of threshold, capping the maximum recoverable count below the input 673,845 pairs.

## Evidence

- [results] Baseline assembled count at quality threshold 0.9: "Of these, 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9."
- [results] Assembled count at threshold 0.6 (9% increase): "We assembled the same single-template data with a quality threshold of 0.6 and this increased the number of sequences assembled by 9%, yielding 652 249 sequences."
- [results] Dataset size and source for the paired-end 16S rRNA library: "Sequencing produced 673 845 paired-end 108-base reads, available at http://neufeldserver.uwaterloo.ca/~apmasell/pandaseq_sampledata.tar."
- [intro] Motivation: naive assembly discards ~40% of sequences: "approximately 40% of the sequences were discarded due to uncalled or miscalled bases."
- [readme] Concrete PANDAseq invocation for assembling the FASTQ pairs: "The short version is:

	pandaseq -f forward.fastq -r reverse.fastq"
