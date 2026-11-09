---
name: highest-scoring-segment-pair-selection
description: Use when multiple overlapping or adjacent DIAMOND (tabular BLASTX) hits
  map to the same reference protein and must be merged into a single continuous sequence
  while resolving overlaps by HSP score.
when_to_use_negative:
- Only a single nonoverlapping hit exists per reference — no stitching or HSP comparison
  is needed
- Hits come from different references and should not be merged into one sequence
- The query is unaligned raw reads rather than translated contig hits from DIAMOND
  — retrieval then relies on the read-based mode of Patchwork, not hit stitching of
  BLASTX tabular output
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Patchwork
  role: 'Implements the hit-stitching algorithm: stacks, overlap detection, realignment
    and HSP retention, and concatenation of merged regions per reference'
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: Produces the tabular BLASTX alignments of query contigs against reference
    proteins that are the input to HSP selection and stitching
  repo: https://github.com/bbuchfink/diamond
- name: Julia
  role: Language in which Patchwork's stitching and HSP-retention logic is written
  repo: https://github.com/fethalen/Patchwork
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:5c1b2f8cb04ee756c98c6183ec8692c2ad35a28fdde7f189d18de45e5060272c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - DIAMOND
  - Julia
  - Patchwork
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# highest-scoring-segment-pair-selection

## Summary

During Patchwork's hit-stitching phase, overlapping or adjacent DIAMOND BLASTX hit regions against a reference protein are merged so that only the highest-scoring segment pair (HSP) is retained for each overlapping interval, producing one continuous stitched sequence per reference.

## When to use

Use when multiple overlapping or adjacent DIAMOND (tabular BLASTX) hits map to the same reference protein and must be merged into a single continuous sequence while resolving overlaps by HSP score.

## When NOT to use

- Only a single nonoverlapping hit exists per reference — no stitching or HSP comparison is needed
- Hits come from different references and should not be merged into one sequence
- The query is unaligned raw reads rather than translated contig hits from DIAMOND — retrieval then relies on the read-based mode of Patchwork, not hit stitching of BLASTX tabular output

## Inputs

- DIAMOND tabular BLASTX output (query contig hits against amino acid reference protein database)
- Query nucleotide contig sequences (FASTA)
- Reference protein sequences (one or more amino acid FASTA entries)

## Outputs

- stitched_hits.fasta — one continuous merged amino acid sequence per reference protein
- Per-reference set of ordered, nonoverlapping HSP-retained regions

## How to apply

Parse the DIAMOND tabular output (produced with the tabular style against the amino acid reference database) together with the query contig sequences. For each reference, sort the query hit regions by their alignment coordinates on the target and push them onto a stack, then compare consecutive regions pairwise to detect overlaps. For each detected overlap, realign the overlapping interval, retain only the highest-scoring segment pair (HSP) sequence for that overlap, and preserve the nonoverlapping flanking parts of both regions as up to three new stack entries, iterating until no overlaps remain. Finally, concatenate the ordered, nonoverlapping regions into one continuous amino acid sequence per reference and write them to stitched_hits.fasta. This is done in Patchwork (Julia) so that merged sequences faithfully represent the best-supported translated query regions for each reference.

## Related tools

- **Patchwork** (Implements the hit-stitching algorithm: stacks, overlap detection, realignment and HSP retention, and concatenation of merged regions per reference) — https://github.com/fethalen/Patchwork
- **DIAMOND** (Produces the tabular BLASTX alignments of query contigs against reference proteins that are the input to HSP selection and stitching) — https://github.com/bbuchfink/diamond
- **Julia** (Language in which Patchwork's stitching and HSP-retention logic is written) — https://github.com/fethalen/Patchwork

## Evaluation signals

- stitched_hits.fasta is valid FASTA containing exactly one continuous sequence per reference protein
- For every merged overlap, the retained sequence corresponds to the highest-scoring segment pair (HSP) from the realignment of the overlapping interval
- No remaining stack entries overlap in alignment coordinates — the algorithm iterated until all overlaps were resolved
- Stitched output matches the deposited Patchwork implementation (github.com/fethalen/Patchwork) run on the same DIAMOND hit input
- Merged region order follows ascending alignment coordinates on the target reference

## Limitations

- Stitching merges regions guided by the reference protein; using references that are not near-universal single-copy orthologs risks chimeric stitched sequences
- Only the highest-scoring HSP is kept per overlap, so lower-scoring but potentially informative overlapping sequence is discarded
- Results depend on DIAMOND hit quality; settings such as tantan masking (--masking 0) affect which hits are available for stitching

## Evidence

- [methods] HSP retention during stitching: "all alignments made between the query region and the target sequence are merged in a way such that only the highest-scoring segment pair (HSP) for each region is retained"
- [abstract] hit stitching phase definition: "followed by a "hit stitching" phase, in which adjacent or overlapping regions are merged into a single unit"
- [readme] README description of merging: "Multiple, overlapping hits are merged to derive a single, continuous sequence for each reference sequence."
- [methods] concatenation of remaining regions: "the resulting set of ordered, nonoverlapping sequence regions are concatenated into 1 continuous sequence"
