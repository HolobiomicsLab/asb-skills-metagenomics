---
name: metagenomic-reference-sequence-comparison
description: Use when candidate transferred segments reconstructed from shotgun metagenomics
  data must be scored against donor/receptor reference segment sequences to measure
  local alignment similarity and reconstruction accuracy.
when_to_use_negative:
- When no reference donor/receptor sequences are available, since the scoring is defined
  only against corresponding reference segment sequences (hi)
- When a global (end-to-end) alignment is required rather than local alignment, since
  Smith-Waterman scores only the best local match and may understate divergence outside
  the aligned region
- When the goal is HGT breakpoint detection itself rather than scoring already-reconstructed
  segments — use LEMON's raw/accurate breakpoint detection steps instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3941
tools:
- name: Smith-Waterman algorithm
  role: Local alignment algorithm used to measure similarity between segment sequences
    of Hi and hi with match_score=1 and mismatch_score=-1
- name: ssw-py
  role: Python binding of the SSW Smith-Waterman implementation listed as a required
    LEMON package for computing segment alignment scores
  repo: https://github.com/lichen2018/LEMON
- name: LEMON
  role: Pipeline whose reconstructed HGT strain segments and breakpoint outputs feed
    the alignment-based similarity scoring
  repo: https://github.com/lichen2018/LEMON
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:4affc6c029cea7d621ccbc039bceff7219107a566d4c299269742cc2431d5d75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - LEMON
  - Smith-Waterman algorithm
  - ssw-py
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# metagenomic-reference-sequence-comparison

## Summary

Quantifies how well reconstructed transferred segments (Hi) match their reference counterparts (hi) by performing Smith-Waterman local sequence alignment, as used in LEMON's transferred-segment identification stage to compute per-strain reconstruction accuracy (RAi).

## When to use

Use when candidate transferred segments reconstructed from shotgun metagenomics data must be scored against donor/receptor reference segment sequences to measure local alignment similarity and reconstruction accuracy.

## When NOT to use

- When no reference donor/receptor sequences are available, since the scoring is defined only against corresponding reference segment sequences (hi)
- When a global (end-to-end) alignment is required rather than local alignment, since Smith-Waterman scores only the best local match and may understate divergence outside the aligned region
- When the goal is HGT breakpoint detection itself rather than scoring already-reconstructed segments — use LEMON's raw/accurate breakpoint detection steps instead

## Inputs

- Candidate transferred segment sequences reconstructed from local strains (derived from the accurate HGT breakpoints file, e.g. sample.acc.txt)
- Corresponding donor/receptor reference segment sequences from the metagenomic reference FASTA
- True/reference breakpoint pair positions for match flagging

## Outputs

- segment_similarity_scores.csv containing per-segment Smith-Waterman alignment scores, breakpoint match flags, and RAi reconstruction accuracy values

## How to apply

Load the candidate transferred segments derived from the reconstructed local strains (using the accurate HGT breakpoints file, e.g. sample.acc.txt) and the corresponding donor/receptor sequences from the metagenomic reference FASTA. Align each reconstructed segment sequence against each true/reference segment sequence with the Smith-Waterman local alignment algorithm, using match_score=1 and mismatch_score=-1, as implemented via the ssw-py Python binding required by LEMON. Record the alignment score SW(hi,Hi) for each strain and compute the reconstruction accuracy RAi = SW(hi,Hi)/(mi*match_score), where mi is the number of segments in the true strain (RAi=0 for an empty reconstruction). Additionally flag a segment pair as matched when both reconstructed breakpoint positions lie within 20 bp of the true breakpoint pair positions. Write per-segment alignment scores, match flags, and RA values to segment_similarity_scores.csv.

## Related tools

- **Smith-Waterman algorithm** (Local alignment algorithm used to measure similarity between segment sequences of Hi and hi with match_score=1 and mismatch_score=-1)
- **ssw-py** (Python binding of the SSW Smith-Waterman implementation listed as a required LEMON package for computing segment alignment scores) — https://github.com/lichen2018/LEMON
- **LEMON** (Pipeline whose reconstructed HGT strain segments and breakpoint outputs feed the alignment-based similarity scoring) — https://github.com/lichen2018/LEMON

## Evaluation signals

- Every RAi value falls within the range [0, 1]
- RAi equals 1 exactly when all transferred segments are matched, i.e. SW equals mi*match_score
- Every segment row in segment_similarity_scores.csv contains a finite Smith-Waterman score
- Segment match flags agree with the 20 bp breakpoint tolerance rule: both reconstructed breakpoint positions lie within 20 bp of the true positions
- Empty reconstructions yield RAi = 0 rather than undefined/null values

## Limitations

- RAi depends on the number of true segments (mi); a strain with no reconstructed segments scores 0, which conflates failure to reconstruct with poor sequence similarity
- Scoring quality depends on the accuracy of the upstream breakpoint detection (raw and accurate HGT breakpoints) and the reconstruction stage
- The match criterion is positional (breakpoints within 20 bp) in addition to the sequence alignment score, so high SW scores alone do not guarantee a match flag

## Evidence

- [methods] Smith-Waterman measures similarity between reconstructed and reference segments: "We apply Smith-Waterman algorithm to measure the similarity between segment sequences of Hi and hi."
- [readme] ssw-py is the Smith-Waterman implementation dependency of LEMON: "- Python packages(3.5+)
  - pysam
  - sklearn
  - numpy
  - scipy
  - lmfit
  - ssw-py"
- [readme] Accurate breakpoint file provides the reconstructed segments used in comparison: "-a        FILE  accurate reakpoints file"
