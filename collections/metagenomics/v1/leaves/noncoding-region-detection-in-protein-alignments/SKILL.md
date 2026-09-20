---
name: noncoding-region-detection-in-protein-alignments
description: Use when stitched (hit-stitched) query-to-reference protein alignments
  contain putative noncoding stretches — many indels with occasional matches — that
  must be trimmed before concatenating phylogenetic markers.
when_to_use_negative:
- The input is raw reads or unassembled contigs before the DIAMOND similarity search
  and hit-stitching phase — this skill operates on stitched alignments only.
- Reference sequences are so divergent (not near-USCOs) that most of the alignment
  is low-identity; Patchwork's authors recommend limiting references to near-USCOs
  to avoid chimeric sequences.
- A post-hoc check for artifactual noncoding stretches in already-built alignments
  is needed — PREQUAL is recommended instead for detecting and removing such regions.
edam_operation: http://edamontology.org/operation_2009
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0080
tools:
- name: Patchwork
  role: Implements the sliding window-based alignment trimming that removes noncoding
    regions from stitched sequences
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: Produces the query-to-reference BLASTX alignments (BLOSUM62 scoring) that
    serve as input to the trimming step
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:e2fc729fbe5894acecdc422c5a6378a5f146ca0efa2f96e79b4ac54f6a4ec00e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - DIAMOND
  - Julia
  - Patchwork
  edam_operation: http://edamontology.org/operation_2009
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# noncoding-region-detection-in-protein-alignments

## Summary

Patchwork's final processing stage: a sliding window algorithm scans stitched DIAMOND query-to-reference alignments and cuts regions whose average gap-excluded query-reference distance exceeds a threshold, removing noncoding stretches from the stitched marker sequences.

## When to use

Use when stitched (hit-stitched) query-to-reference protein alignments contain putative noncoding stretches — many indels with occasional matches — that must be trimmed before concatenating phylogenetic markers.

## When NOT to use

- The input is raw reads or unassembled contigs before the DIAMOND similarity search and hit-stitching phase — this skill operates on stitched alignments only.
- Reference sequences are so divergent (not near-USCOs) that most of the alignment is low-identity; Patchwork's authors recommend limiting references to near-USCOs to avoid chimeric sequences.
- A post-hoc check for artifactual noncoding stretches in already-built alignments is needed — PREQUAL is recommended instead for detecting and removing such regions.

## Inputs

- Stitched query-to-reference alignment produced by DIAMOND BLASTX (after hit stitching and alignment masking)
- User-specified window size (optional; Patchwork default used if absent)
- User-specified distance threshold (optional; Patchwork default used if absent)

## Outputs

- trimmed_sequence.fasta — FASTA file of the trimmed coding sequence per marker

## How to apply

Load the stitched alignment produced by DIAMOND BLASTX (default scoring: BLOSUM62 substitution matrix, gap open 11, gap extension 1) into Patchwork's trimming stage. Read the user-provided window size and distance threshold, falling back to Patchwork's built-in defaults when not supplied. Scan the alignment left to right in windows, computing the average gap-excluded, identity-based distance between query and reference residues per window. Cut every region whose window-average distance exceeds the threshold; because the criterion is evaluated over the window average rather than single positions, isolated bad-but-correct matches are preserved. Write the trimmed coding sequence to trimmed_sequence.fasta. The rationale is that noncoding stretches enter the stitched sequence as indel-rich regions with occasional matches, which sliding-window averaging distinguishes from genuine coding sequence.

## Related tools

- **Patchwork** (Implements the sliding window-based alignment trimming that removes noncoding regions from stitched sequences) — https://github.com/fethalen/Patchwork
- **DIAMOND** (Produces the query-to-reference BLASTX alignments (BLOSUM62 scoring) that serve as input to the trimming step) — https://github.com/bbuchfink/diamond

## Evaluation signals

- Regions removed are exactly those whose average window distance (gap-excluded, identity-based) is above the distance threshold — no more, no less.
- Isolated bad-but-correct matches are preserved because trimming is triggered by the window average, not single positions.
- Trimmed same-species markers attain ~95.9% identical aligned positions and ~92.2% mean query coverage, as reported in the Patchwork benchmark on 815 D. gyrociliatus USCOs.
- Output file trimmed_sequence.fasta exists, is valid FASTA, and contains a continuous (concatenated and realigned) coding sequence per reference.
- Most markers remain well above the 90% aligned-position trimming threshold after trimming.

## Limitations

- For distantly related references (e.g. the H. robusta cross-species benchmark), average identity drops (~89% for retrieved markers), making threshold choice harder — choosing the correct cutoff becomes easier as identity approaches 100%.
- The trimming operates on translated alignments, so residual artifactual noncoding stretches may survive; PREQUAL is suggested as a downstream detector of such regions.
- Window size and distance threshold must be supplied or Patchwork's defaults are used; no task-specific cutoff values are stated in the sources.

## Evidence

- [abstract] sliding window trims noncoding regions: "The novel sliding window algorithm trims away any noncoding regions from the resulting sequence"
- [methods] sliding window trimming implementation: "we have implemented a sliding window–based alignment trimming approach to rid the alignments from these unwanted regions"
- [readme] stitching context before trimming: "Stitch overlapping or gappy sequences together based on a reference"
- [discussion] cutoff threshold caveat: "choosing the correct cutoff threshold becomes increasingly easier as the level of identity approaches 100%"
- [results] benchmark quality of trimmed markers: "On average, 95.9% of all aligned positions were identical matches, with a mean query coverage of 92.2%"
