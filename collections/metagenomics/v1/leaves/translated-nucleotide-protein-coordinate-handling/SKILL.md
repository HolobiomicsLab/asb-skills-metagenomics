---
name: translated-nucleotide-protein-coordinate-handling
description: Use when reconstructing how adjacent or overlapping DIAMOND BLASTX hits
  against protein references are stitched into a single continuous amino acid sequence
  per reference, as in Patchwork.
when_to_use_negative:
- The input has not yet been through a DIAMOND sequence similarity search — stitching
  operates on existing homologous region alignments, not raw reads or contigs alone
- You need trimmed, noncoding-free sequences — the stitching phase precedes Patchwork's
  sliding window alignment trimming, which removes noncoding regions from the stitched
  result
- Reference sequences are not near-universal single-copy orthologs — the authors warn
  that references beyond near-USCOs risk producing chimeric stitched sequences
edam_operation: http://edamontology.org/operation_0495
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: Patchwork
  role: Implements the hit-stitching algorithm that merges adjacent or overlapping
    DIAMOND hits into a single unit per reference protein
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: Produces the translated BLASTX alignments (tabular output) between query DNA
    contigs and reference protein sequences that feed the stitching step
  repo: https://github.com/bbuchfink/diamond
- name: Julia
  role: Programming language in which Patchwork (and its stitching logic) is written,
    using BioJulia packages for sequence handling
  repo: https://github.com/fethalen/Patchwork
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:faa0015ad145d453c29cd9f68ab54717dab4fd6452487fb4493e6d7f49ffe183
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - DIAMOND
  - Julia
  - Patchwork
  edam_operation: http://edamontology.org/operation_0495
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# translated-nucleotide-protein-coordinate-handling

## Summary

Handles the coordinate mapping between translated DNA query contigs and amino acid reference sequences during Patchwork's hit-stitching phase, where DIAMOND BLASTX hits on the same reference are ordered by alignment coordinates and overlapping regions are merged into one continuous sequence per reference.

## When to use

Use when reconstructing how adjacent or overlapping DIAMOND BLASTX hits against protein references are stitched into a single continuous amino acid sequence per reference, as in Patchwork.

## When NOT to use

- The input has not yet been through a DIAMOND sequence similarity search — stitching operates on existing homologous region alignments, not raw reads or contigs alone
- You need trimmed, noncoding-free sequences — the stitching phase precedes Patchwork's sliding window alignment trimming, which removes noncoding regions from the stitched result
- Reference sequences are not near-universal single-copy orthologs — the authors warn that references beyond near-USCOs risk producing chimeric stitched sequences

## Inputs

- DIAMOND BLASTX tabular output of query nucleotide contigs aligned against amino acid reference protein sequences
- Query contig (DNA nucleotide) sequences in FASTA format
- Reference protein sequences in FASTA format

## Outputs

- stitched_hits.fasta: valid FASTA containing one continuous stitched amino acid sequence per reference sequence
- Per-reference merged regions with overlaps resolved by best-scoring HSP retention

## How to apply

Parse the DIAMOND tabular BLASTX output (produced with tabular-style output against the reference protein database) together with the query contig sequences. For each reference, sort the query hit regions by their alignment coordinates on the target, first to last, and push them onto a stack; compare consecutive regions pairwise and record overlaps. For each overlapping interval, realign the two regions, retain only the best-scoring HSP sequence for the overlap, and preserve the nonoverlapping flanking parts as up to three new stack entries, iterating until no overlaps remain. Finally, concatenate the ordered, nonoverlapping regions into one continuous translated nucleotide (amino acid) sequence per reference and write them to stitched_hits.fasta. This works because reference protein sequences guide stitching: the best-scoring translated query nucleotide sequences for any given region are merged into continuous stretches.

## Related tools

- **Patchwork** (Implements the hit-stitching algorithm that merges adjacent or overlapping DIAMOND hits into a single unit per reference protein) — https://github.com/fethalen/Patchwork
- **DIAMOND** (Produces the translated BLASTX alignments (tabular output) between query DNA contigs and reference protein sequences that feed the stitching step) — https://github.com/bbuchfink/diamond
- **Julia** (Programming language in which Patchwork (and its stitching logic) is written, using BioJulia packages for sequence handling) — https://github.com/fethalen/Patchwork

## Examples

```
diamond blastx -d refs.dmnd -q contigs.fasta -o hits.tsv --outfmt 6 --masking 0
```

## Evaluation signals

- stitched_hits.fasta is valid FASTA with exactly one continuous sequence per reference protein
- No overlapping intervals remain among the merged regions for any reference — pairwise coordinate comparison of stacked regions yields zero overlaps
- Each overlap was resolved by retaining the highest-scoring HSP sequence for the overlapping interval, preserving flanking nonoverlapping parts of both regions
- Stitched output matches the deposited Patchwork implementation (github.com/fethalen/Patchwork) on the same DIAMOND hit input
- Stitched regions follow target-coordinate order (first to last) as sorted before stacking

## Limitations

- Stitching merges whatever DIAMOND reports; chimeric sequences can result if reference sequences are not limited to near-USCOs (authors recommend restricting references to near-USCOs)
- The stitching phase occurs before noncoding-region trimming, so the stitched sequence may still contain noncoding, ambiguous, or stop-codon residues until later masking/trimming steps run
- Disabling tantan masking in DIAMOND (--masking 0) changes the hits available for stitching — query coverage improves especially for read data, affecting which regions get stitched

## Evidence

- [abstract] hit stitching merges overlapping hits: "followed by a "hit stitching" phase, in which adjacent or overlapping regions are merged into a single unit"
- [methods] HSP retention in stitching: "all alignments made between the query region and the target sequence are merged in a way such that only the highest-scoring segment pair (HSP) for each region is retained"
- [methods] concatenation of remaining regions: "the resulting set of ordered, nonoverlapping sequence regions are concatenated into 1 continuous sequence"
- [readme] README statement of merging behavior: "Multiple, overlapping hits are merged to derive a single, continuous sequence for each reference sequence."
- [discussion] chimera warning: "we recommend that the user limits the use of reference sequences to near-USCOs"
