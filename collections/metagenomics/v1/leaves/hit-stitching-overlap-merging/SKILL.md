---
name: hit-stitching-overlap-merging
description: 'Use when reconstructing Patchwork''s hit-stitching phase: DIAMOND tabular
  BLASTX hits against protein references must be merged into a single continuous sequence
  per reference for marker retrieval.'
when_to_use_negative:
- The input is raw DIAMOND hits that are already nonoverlapping and ordered per reference
  — stitching has nothing to merge
- The goal is trimming noncoding regions from an already-stitched sequence — that
  is Patchwork's separate sliding-window alignment trimming step
- References are not single-copy or near-USCO sequences — the article warns against
  chimeric sequences when references are not limited to near-USCOs
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0084
tools:
- name: Patchwork
  role: Implements the hit-stitching algorithm (stack-based overlap detection, HSP
    retention, concatenation) in Julia
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: Produces the tabular BLASTX alignments between query nucleotide sequences
    and the amino acid reference database that serve as stitching input
  repo: https://github.com/bbuchfink/diamond
- name: Julia
  role: Programming language in which Patchwork's stitching logic is written
  repo: https://github.com/fethalen/Patchwork
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:40b2bed21243413f8e503f6c834c164d5c348ad2f813fcd4af9af0e4b12c4309
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
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# hit-stitching-overlap-merging

## Summary

Patchwork's hit-stitching step merges adjacent or overlapping DIAMOND BLASTX hits into one continuous translated amino acid sequence per reference protein, retaining only the best-scoring HSP for each overlapping region.

## When to use

Use when reconstructing Patchwork's hit-stitching phase: DIAMOND tabular BLASTX hits against protein references must be merged into a single continuous sequence per reference for marker retrieval.

## When NOT to use

- The input is raw DIAMOND hits that are already nonoverlapping and ordered per reference — stitching has nothing to merge
- The goal is trimming noncoding regions from an already-stitched sequence — that is Patchwork's separate sliding-window alignment trimming step
- References are not single-copy or near-USCO sequences — the article warns against chimeric sequences when references are not limited to near-USCOs

## Inputs

- DIAMOND tabular BLASTX output of query contigs vs. amino acid reference protein database
- Query DNA contig (or read-derived) nucleotide sequences in FASTA
- Reference amino acid sequences (e.g. near-USCO / USCO protein references)

## Outputs

- stitched_hits.fasta containing one continuous, nonoverlapping translated amino acid sequence per reference

## How to apply

Parse the DIAMOND tabular BLASTX output (produced in tabular format against the reference protein database) together with the query contig sequences. For each reference, sort the query hit regions by their alignment coordinates on the target and push them onto a stack. Compare consecutive stack regions pairwise; when two regions overlap, record the overlap, realign the overlapping interval between the two regions, and retain only the best-scoring segment pair (HSP) for the overlap, preserving the nonoverlapping flanking parts as up to three new stack entries and iterating until no overlaps remain. Finally, the resulting set of ordered, nonoverlapping regions is concatenated into one continuous sequence per reference and written to stitched_hits.fasta. The rationale is that translated query nucleotide sequences for a given region are stitched against the reference protein, guided by alignment scores rather than raw contig order.

## Related tools

- **Patchwork** (Implements the hit-stitching algorithm (stack-based overlap detection, HSP retention, concatenation) in Julia) — https://github.com/fethalen/Patchwork
- **DIAMOND** (Produces the tabular BLASTX alignments between query nucleotide sequences and the amino acid reference database that serve as stitching input) — https://github.com/bbuchfink/diamond
- **Julia** (Programming language in which Patchwork's stitching logic is written) — https://github.com/fethalen/Patchwork

## Examples

```
diamond blastx -q contigs.fasta -d references.dmnd -o hits.tsv --masking 0 --tabular-style tabular  # hits.tsv is the stitching input parsed by Patchwork
```

## Evaluation signals

- stitched_hits.fasta must be a valid FASTA with exactly one continuous sequence per reference protein
- No two retained regions within a stitched sequence overlap — iterative realignment and HSP retention must have removed all overlaps
- The stitched result must match the output of the deposited Patchwork implementation (github.com/fethalen/Patchwork) on the same DIAMOND hit input
- Each overlapping interval contributes only the best-scoring (HSP) sequence, with nonoverlapping flanking parts of both regions preserved

## Limitations

- Stitching quality depends on the initial DIAMOND alignments; e.g. disabling tantan masking (--masking 0) improves query coverage, especially for read data, affecting which regions are available to stitch
- The article recommends limiting references to near-USCOs, as non-single-copy references can yield chimeric stitched sequences
- Iterative pairwise overlap resolution on the stack means many heavily overlapping hits may require multiple realignment rounds per reference

## Evidence

- [methods] HSP retention during merge: "all alignments made between the query region and the target sequence are merged in a way such that only the highest-scoring segment pair (HSP) for each region is retained"
- [methods] concatenation of ordered nonoverlapping regions: "the resulting set of ordered, nonoverlapping sequence regions are concatenated into 1 continuous sequence"
- [abstract] hit-stitching definition: "followed by a "hit stitching" phase, in which adjacent or overlapping regions are merged into a single unit"
- [readme] README purpose statement: "Multiple, overlapping hits are merged to derive a single, continuous sequence for each reference sequence."
