---
name: phred-quality-probability-conversion
description: Use when FASTQ or SAM input from Illumina CASAVA pipelines contains ASCII
  quality values that must be turned into per-base error probabilities for probabilistic
  scoring, overlap alignment, or assembly quality filtering.
when_to_use_negative:
- Input is a feature table, amplicon sequence variant table, or already-assembled
  contigs with no per-base PHRED quality strings
- Reads carry non-CASAVA quality encoding without verification — SeqPrep README notes
  ASCII < 33 characters appear when phred+64 data is misread, so check the encoding
  offset before applying the conversion
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: PANDAseq
  role: consumes CASAVA-encoded quality values, converts them to error probabilities
    for overlap scoring and assembly quality calculation
  repo: https://github.com/neufeld/pandaseq
- name: PANDAseq-sam
  role: same quality-to-probability conversion applied to SAM/BAM formatted paired-end
    input
  repo: https://github.com/neufeld/pandaseq-sam
- name: SeqPrep
  role: alternative paired-end merger whose -q cutoff (quality score for counting
    mismatches in overlap, default 13) reflects the same PHRED quality interpretation
  repo: https://github.com/jstjohn/SeqPrep
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-13-31
    title: 'PANDAseq: paired-end assembler for illumina sequences'
  source_content_hash: sha256:358824a97c023598609cadb98e6f50014b5384b57aeaace2d29c6ae5a7c3add5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neufeld/pandaseq
  tools:
  - PANDAseq
  - PANDAseq-sam
  - SeqPrep
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# phred-quality-probability-conversion

## Summary

Convert CASAVA-encoded ASCII PHRED quality characters in Illumina FASTQ reads into per-base error probabilities using ε = 10^(-A/10), which is the basis for PANDAseq's probabilistic overlap scoring, error correction, and assembly quality thresholds.

## When to use

Use when FASTQ or SAM input from Illumina CASAVA pipelines contains ASCII quality values that must be turned into per-base error probabilities for probabilistic scoring, overlap alignment, or assembly quality filtering.

## When NOT to use

- Input is a feature table, amplicon sequence variant table, or already-assembled contigs with no per-base PHRED quality strings
- Reads carry non-CASAVA quality encoding without verification — SeqPrep README notes ASCII < 33 characters appear when phred+64 data is misread, so check the encoding offset before applying the conversion

## Inputs

- Illumina paired-end reads in FASTQ format (forward.fastq, reverse.fastq) with CASAVA-encoded PHRED quality strings
- SAM/BAM formatted paired-end files (for PANDAseq-sam variant)

## Outputs

- Per-base error probabilities ε = 10^(-A/10) used in overlap-scoring and quality computations
- Assembly quality score per reconstructed sequence (geometric mean of per-base probabilities), compared against thresholds p = 0.9 and p = 0.6

## How to apply

For each base, take the CASAVA-encoded ASCII quality value A (pre-1.8 or CASAVA 1.8 encodings) and compute the base-error probability ε = 10^(-A/10). Respect the CASAVA convention that end-of-read quality scores may be masked to the lowest score; bases in these quality-masked tail regions are treated specially downstream (1/4 probability for arbitrary calls, and doubly-masked mismatches drive low assembly scores). Uncalled bases (N) are handled with a flat 1/4 probability that the true base matches. The resulting per-base probabilities feed PANDAseq's match/mismatch probability formulas, e.g. Pr[X̂=Ŷ|X=Y] = (1-εX)(1-εY) + εXεY/3, and the geometric-mean assembly quality score thresholded at p = 0.9 or p = 0.6.

## Related tools

- **PANDAseq** (consumes CASAVA-encoded quality values, converts them to error probabilities for overlap scoring and assembly quality calculation) — https://github.com/neufeld/pandaseq
- **PANDAseq-sam** (same quality-to-probability conversion applied to SAM/BAM formatted paired-end input) — https://github.com/neufeld/pandaseq-sam
- **SeqPrep** (alternative paired-end merger whose -q cutoff (quality score for counting mismatches in overlap, default 13) reflects the same PHRED quality interpretation) — https://github.com/jstjohn/SeqPrep

## Examples

```
pandaseq -f forward.fastq -r reverse.fastq
```

## Evaluation signals

- Every converted ε must lie in (0, 1]; assembly quality scores computed from these ε values must also lie in (0, 1]
- Validate against the simulated benchmark: all 1,350,602 synthesized paired-end reads built with real Illumina quality masks should assemble with quality scores > 0.9
- On the M. capsulatus control run (673,845 pairs, SRA024100), 598,775 sequences should pass the 0.9 threshold; lowering to 0.6 yields 652,249 (a 9% increase)
- Mismatch-dominated overlaps confined to doubly quality-masked regions should produce low overall quality scores and be discarded — a sanity check that the ε conversion preserves CASAVA masking semantics
- Verify the accept flag matches the threshold comparison exactly: score >= 0.9 and score >= 0.6 flags in the quality_scores table

## Limitations

- CASAVA masks end-of-read quality scores to the lowest score, so ε values in read tails are upper-bound estimates and doubly-masked mismatches can unfairly depress assembly quality scores
- Uncalled (N) bases use a flat 1/4 match probability, giving no discrimination among true bases
- No software version number is provided for PANDAseq in the article, so exact behavior of the conversion across versions is unverified

## Evidence

- [methods] convert CASAVA ASCII quality: [evidence span withheld — anchor status: not_found]
- [methods] match probability formula: [evidence span withheld — anchor status: not_found]
- [discussion] CASAVA masking caveat: "incorrect assemblies have low quality scores, as these mismatches occur in quality-masked regions of both reads, and are discarded"
- [results] benchmark validation result: "all 1 350 602 synthesized paired-end sequences assembled with quality scores greater than 0.9"
- [results] 0.6 threshold recovery gain: "We assembled the same single-template data with a quality threshold of 0.6 and this increased the number of sequences assembled by 9%"
- [readme] SeqPrep encoding caveat: [evidence span withheld — anchor status: external_doc]
