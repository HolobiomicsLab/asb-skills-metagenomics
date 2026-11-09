---
name: illumina-quality-probability-scoring
description: Decision support for setting PANDAseq assembly quality-score thresholds
  when recovering more overlapping Illumina paired-end reads matters more than maximum
  stringency, e.g. 16S rRNA amplicon libraries with many low-quality tail bases.
when_to_use_negative:
- Read pairs have no sufficient overlap — PANDAseq discards these pairs regardless
  of threshold, so the skill cannot recover them
- Sequences contain uncalled bases that must be excluded outright when the downstream
  pipeline forbids N calls
- Assembly of non-overlapping paired-end reads (e.g. fragment libraries without overlap)
  where quality-probability scoring in the overlap region is not applicable
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0632
tools:
- name: PANDAseq
  role: Probabilistic paired-end assembler that computes quality-score-based match
    probabilities and filters assemblies by the quality threshold
  repo: https://github.com/neufeld/pandaseq
- name: SeqPrep
  role: Alternative overlapping paired-end merger used for comparison; uses alignment
    with mismatch/overlap fraction parameters rather than probabilistic quality scoring
  repo: https://github.com/jstjohn/SeqPrep/
- name: stitch
  role: Alternative paired-end overlap assembler (now unmaintained) used as a benchmark
    against PANDAseq
  repo: https://github.com/audy/stitch
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:ee666cd61977b513c659cae62806ef42d925714762a9a809cb389dfa81086a07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - PANDAseq
  - SeqPrep
  - stitch
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# illumina-quality-probability-scoring

## Summary

Uses Illumina per-base quality scores within the paired-end overlap to compute the probability that true nucleotides match, so mismatches and uncalled bases are corrected probabilistically instead of discarding read pairs. Lowering the assembly quality-score threshold (e.g. 0.9 to 0.6) trades stringency for higher sequence recovery.

## When to use

Decision support for setting PANDAseq assembly quality-score thresholds when recovering more overlapping Illumina paired-end reads matters more than maximum stringency, e.g. 16S rRNA amplicon libraries with many low-quality tail bases.

## When NOT to use

- Read pairs have no sufficient overlap — PANDAseq discards these pairs regardless of threshold, so the skill cannot recover them
- Sequences contain uncalled bases that must be excluded outright when the downstream pipeline forbids N calls
- Assembly of non-overlapping paired-end reads (e.g. fragment libraries without overlap) where quality-probability scoring in the overlap region is not applicable

## Inputs

- Forward paired-end Illumina FASTQ file (e.g. 108-base reads from the M. capsulatus 16S rRNA amplicon library, SRA024100-derived sample data)
- Reverse paired-end Illumina FASTQ file matched by flow-cell template position
- Assembly quality-score threshold (e.g. 0.9 or 0.6)

## Outputs

- Assembled overlapping consensus sequences with overall assembly quality scores above the threshold (e.g. 598,775 sequences at q=0.9; 652,249 at q=0.6)
- Count comparison of assembled sequences across thresholds (assembled_sequence_counts.csv in the reproduction task)

## How to apply

Run PANDAseq (pandaseq -f forward.fastq -r reverse.fastq) on the paired-end FASTQ files; the tool determines primer locations, identifies the optimal overlap, calculates the probability that true nucleotides X and Y are the same given their observed quality scores, then constructs the output sequence with an overall assembly quality score. Validate each assembled sequence against the user-specified quality threshold and other constraints such as length and uncalled bases; read pairs with no suitable overlap are discarded. To recover more sequences from a single-template library, re-run the identical input with a lower quality threshold: on the 673,845 paired-end 108-base M. capsulatus reads, threshold 0.9 yielded 598,775 sequences while threshold 0.6 yielded 652,249 — a 9% increase. Lowering the threshold increases error incorporation only modestly (0.02–0.08% of outputs), since incorrect assemblies with mismatches in quality-masked regions of both reads naturally receive low quality scores and are discarded.

## Related tools

- **PANDAseq** (Probabilistic paired-end assembler that computes quality-score-based match probabilities and filters assemblies by the quality threshold) — https://github.com/neufeld/pandaseq
- **SeqPrep** (Alternative overlapping paired-end merger used for comparison; uses alignment with mismatch/overlap fraction parameters rather than probabilistic quality scoring) — https://github.com/jstjohn/SeqPrep/
- **stitch** (Alternative paired-end overlap assembler (now unmaintained) used as a benchmark against PANDAseq) — https://github.com/audy/stitch

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq -q 0.6 > assembled.fa
```

## Evaluation signals

- At threshold 0.9 the count of assembled sequences from the 673,845 M. capsulatus read pairs should be 598,775
- Lowering the threshold to 0.6 on the same input should increase assembled sequences by ~9% to 652,249
- Simulated data check: all synthesized paired-end sequences assemble with quality scores above 0.9
- Error audit: introduced errors should remain within ~0.02–0.08% of output reads, far below the 5–8% of sequences with corrected errors
- Length and uncalled-base constraints must be satisfied for every retained assembly

## Limitations

- Uncertain error corrections arise from reads with many low-quality bases, so very low thresholds admit more questionable corrections
- Approximately 40% of naive perfect-match assemblies are discarded due to uncalled or miscalled bases — motivates, but also bounds, probabilistic correction
- No software version number is provided for PANDAseq in the article, complicating exact reproduction

## Evidence

- [results] Of these, 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9.: "Of these, 598 775 sequences were assembled with an assembly quality score greater than or equal to 0.9."
- [results] 9% increase at threshold 0.6: "We assembled the same single-template data with a quality threshold of 0.6 and this increased the number of sequences assembled by 9%, yielding 652 249 sequences."
- [methods] Probability scoring given quality values: [evidence span withheld — anchor status: not_found]
- [methods] Quality-score-based filtering options: "PANDAseq enables users to reject sequences based on low quality score, lengths that are too short or too long, or the presence of uncalled bases."
- [readme] Usage invocation from README: "The short version is:

	pandaseq -f forward.fastq -r reverse.fastq"
- [results] Introduced-error rate: "only about 0.02-0.08% of output read contained errors introduced by the PANDAseq assembly process"
