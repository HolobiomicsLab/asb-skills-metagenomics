---
name: smith-waterman-sequence-scoring
description: Use when reconstructed transferred genome segments from an HGT local
  strain must be scored against reference segment sequences to quantify similarity
  and compute reconstruction accuracy (RA).
when_to_use_negative:
- The input is raw paired-end shotgun reads or BAM files rather than reconstructed
  segment sequences — use BWA/LEMON preprocessing first
- A global (end-to-end) alignment score is required — Smith-Waterman is a local alignment
  algorithm and will only report the best-scoring local region
- No reference (true) segment sequences or breakpoint positions exist to compare against,
  so RA cannot be computed
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: LEMON
  role: Pipeline whose transferred-segment identification stage applies Smith-Waterman
    to compare segments of Hi against hi and compute RA
  repo: https://github.com/lichen2018/LEMON
- name: ssw-py
  role: Python Smith-Waterman library listed as a LEMON dependency for sequence alignment
    scoring
  repo: https://github.com/lichen2018/LEMON
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:02b68004aa6b3019548448a8014910392f66a30cae4845b027ff6f73544da853
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - BWA
  - LEMON
  - LUMPY
  - Smith-Waterman algorithm
  - WGSIM
  - ssw-py
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Smith-Waterman sequence scoring

## Summary

Applies the Smith-Waterman local alignment algorithm to measure the similarity between candidate transferred segment sequences of a reconstructed local strain (Hi) and the corresponding reference segment sequences (hi), yielding scores that feed LEMON's reconstruction accuracy (RA) metric.

## When to use

Use when reconstructed transferred genome segments from an HGT local strain must be scored against reference segment sequences to quantify similarity and compute reconstruction accuracy (RA).

## When NOT to use

- The input is raw paired-end shotgun reads or BAM files rather than reconstructed segment sequences — use BWA/LEMON preprocessing first
- A global (end-to-end) alignment score is required — Smith-Waterman is a local alignment algorithm and will only report the best-scoring local region
- No reference (true) segment sequences or breakpoint positions exist to compare against, so RA cannot be computed

## Inputs

- Reconstructed transferred segment sequences (segments of Hi) from LEMON's accurate HGT breakpoints file (sample.acc.txt)
- Corresponding donor/receptor segment sequences from the metagenomic reference FASTA
- True (reference) breakpoint pair positions for match flagging

## Outputs

- Per-segment Smith-Waterman alignment scores SW(hi,Hi)
- Match flags per segment pair (breakpoints within 20 bp tolerance)
- Reconstruction accuracy values RAi and a compiled reconstruction_accuracy_comparison.csv

## How to apply

Load the reconstructed transferred segments derived from LEMON's accurate HGT breakpoints file (sample.acc.txt) and the corresponding donor/receptor sequences from the metagenomic reference FASTA. Align each reconstructed segment sequence against each true/reference segment sequence using the Smith-Waterman local alignment algorithm with match_score = 1 and mismatch_score = -1 (the ssw-py package is LEMON's Python dependency for this). Record the alignment score SW(hi,Hi) and compute RAi = SW(hi,Hi)/(mi*match_score), where mi is the number of segments in the true strain, setting RAi = 0 if the reconstruction is empty. Flag a segment pair as matched when both breakpoint positions of the reconstructed segment lie within 20 bp of the true breakpoint pair. These scores are then compiled into reconstruction-accuracy comparisons (e.g., LEMON vs LUMPY at 5X and 30X coverage).

## Related tools

- **LEMON** (Pipeline whose transferred-segment identification stage applies Smith-Waterman to compare segments of Hi against hi and compute RA) — https://github.com/lichen2018/LEMON
- **ssw-py** (Python Smith-Waterman library listed as a LEMON dependency for sequence alignment scoring) — https://github.com/lichen2018/LEMON

## Evaluation signals

- Each RAi value falls in the range [0, 1]
- RAi = 1 is achieved when all transferred segments are matched, i.e., SW equals mi*match_score
- Every segment row contains a finite Smith-Waterman alignment score
- Matched segments have both breakpoint positions within 20 bp of the true breakpoint pair
- Reproduced benchmark values match the reported figures: LEMON RA of 0.6087 at 5X and 1.0 at 30X, LUMPY RA of 0.5238 at 5X and 0.9565 at 30X

## Limitations

- An empty reconstruction yields RAi = 0 by definition, which conflates 'no reconstruction' with 'poor reconstruction'
- The match flag depends on the 20 bp breakpoint tolerance; scoring outcomes are sensitive to this threshold
- Scoring requires a metagenomic reference (16,093 species, 1,246,881 scaffolds in the article's reference set S) against which segments are aligned

## Evidence

- [methods] SW scoring of candidate vs reference segments: "We apply Smith-Waterman algorithm to measure the similarity between segment sequences of Hi and hi."
- [readme] LEMON context: transferred segments and local strain reconstruction: "It is a software takes use of existing shotgun NGS datasets to detect HGT breakpoints, identify the transferred genome segments, and reconstructs the inserted local strain."
- [abstract] Simulation accuracy motivating the metric: "The simulated results indicate that the method can identify transferred segments accurately on reference sequences of the microbiome."
