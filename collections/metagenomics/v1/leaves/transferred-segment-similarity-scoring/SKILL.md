---
name: transferred-segment-similarity-scoring
description: Use when reconstructed local HGT strain segments (from a sample.acc.txt
  breakpoints file) must be quantitatively compared to true/reference segments to
  measure reconstruction accuracy.
when_to_use_negative:
- The goal is HGT breakpoint detection or raw/accurate breakpoint calling from BAM
  files rather than scoring already-reconstructed segments
- Input sequences are full genomes or unassembled reads rather than bounded transferred-segment
  pairs (hi, Hi); Smith-Waterman local alignment here is intended for segment-vs-segment
  comparison
- Global (end-to-end) sequence identity is required instead of local alignment; Smith-Waterman
  finds optimal local alignment and may understate similarity for full-length comparison
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: LEMON
  role: Pipeline that reconstructs the transferred segments and whose evaluation stage
    applies the Smith-Waterman scoring
  repo: https://github.com/lichen2018/LEMON
- name: ssw-py
  role: Python binding providing the Smith-Waterman local alignment used to score
    segment pairs
- name: BWA
  role: Maps shotgun reads against references upstream, producing the alignments from
    which breakpoints and segments are derived
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:12f6cb74c55907c6b5b05755ad0820d4e728e8a2f438d75587e0f65e7af47e9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - BWA
  - LEMON
  - ssw-py
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# transferred-segment-similarity-scoring

## Summary

Scores the similarity of candidate transferred genome segments reconstructed by LEMON against their reference segment sequences using the Smith-Waterman local alignment algorithm, yielding per-segment alignment scores SW(hi,Hi) and reconstruction accuracy RAi = SW(hi,Hi)/(mi*match_score).

## When to use

Use when reconstructed local HGT strain segments (from a sample.acc.txt breakpoints file) must be quantitatively compared to true/reference segments to measure reconstruction accuracy.

## When NOT to use

- The goal is HGT breakpoint detection or raw/accurate breakpoint calling from BAM files rather than scoring already-reconstructed segments
- Input sequences are full genomes or unassembled reads rather than bounded transferred-segment pairs (hi, Hi); Smith-Waterman local alignment here is intended for segment-vs-segment comparison
- Global (end-to-end) sequence identity is required instead of local alignment; Smith-Waterman finds optimal local alignment and may understate similarity for full-length comparison

## Inputs

- Candidate transferred segment sequences from the reconstructed local strain (derived from the accurate HGT breakpoints file, e.g. sample.acc.txt)
- Reference donor/receptor segment sequences from the metagenomic reference FASTA
- True/reference segment list with true breakpoint pair positions and segment counts mi

## Outputs

- Per-segment Smith-Waterman alignment scores SW(hi,Hi)
- Segment match flags (reconstructed breakpoints within 20 bp of true breakpoints)
- Per-strain reconstruction accuracy values RAi in [0, 1] (e.g. segment_similarity_scores.csv)

## How to apply

Load the candidate transferred segments derived from reconstructed local strains (via the accurate HGT breakpoints file, e.g. sample.acc.txt) and the corresponding donor/receptor sequences from the metagenomic reference FASTA. Align each reconstructed segment sequence Hi against each true/reference segment sequence hi with the Smith-Waterman local alignment algorithm using match_score = 1 and mismatch_score = -1 (the Python ssw-py package provides the alignment implementation). Record the alignment score SW(hi,Hi) for each pair and compute RAi = SW(hi,Hi)/(mi*match_score), where mi is the number of segments in the true strain, setting RAi = 0 if the reconstruction is empty. Flag a segment pair as matched when both reconstructed breakpoint positions lie within 20 bp of the true breakpoint pair positions. Write per-segment scores, match flags, and RA values (e.g. to segment_similarity_scores.csv).

## Related tools

- **LEMON** (Pipeline that reconstructs the transferred segments and whose evaluation stage applies the Smith-Waterman scoring) — https://github.com/lichen2018/LEMON
- **ssw-py** (Python binding providing the Smith-Waterman local alignment used to score segment pairs)
- **BWA** (Maps shotgun reads against references upstream, producing the alignments from which breakpoints and segments are derived)

## Examples

```
python LEMON/Scripts/reconstruct_HGT_strain.py -r ref.fasta -c coverage.txt -s sample -a sample.acc.txt -o out/
```

## Evaluation signals

- Every RAi falls in [0, 1]; RAi = 1 exactly when all transferred segments are matched (SW equals mi*match_score)
- Every segment row carries a finite Smith-Waterman score (no NaN/inf)
- Empty reconstructions are explicitly assigned RAi = 0 rather than being dropped or erroring
- Match flags agree with the 20 bp breakpoint tolerance: both reconstructed breakpoint positions within 20 bp of the true pair positions
- RAi denominator equals mi*match_score with match_score = 1, so scores scale with the true strain's segment count

## Limitations

- Scoring depends on the reconstruction stage: an empty or wrong reconstruction yields RAi = 0 regardless of alignment quality
- The 20 bp breakpoint match tolerance is a fixed threshold; breakpoints outside it are unmatched even if sequences align well
- Smith-Waterman scores are normalized by the number of true segments mi, so accuracy is defined per true strain and is not directly comparable across strains with very different mi

## Evidence

- [methods] Smith-Waterman scoring basis: "We apply Smith-Waterman algorithm to measure the similarity between segment sequences of Hi and hi."
- [readme] Python alignment dependency: "- Python packages(3.5+)
  - pysam
  - sklearn
  - numpy
  - scipy
  - lmfit
  - ssw-py"
- [readme] Pipeline context for segment inputs: "It is a software takes use of existing shotgun NGS datasets to detect HGT breakpoints, identify the transferred genome segments, and reconstructs the inserted local strain."
