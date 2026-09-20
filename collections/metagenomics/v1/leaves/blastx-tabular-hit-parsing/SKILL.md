---
name: blastx-tabular-hit-parsing
description: Use when DNA query contigs (or reads) have been searched against protein
  reference sequences with DIAMOND and tabular BLASTX hits must be grouped per reference
  and ordered for merging into continuous marker sequences.
when_to_use_negative:
- The query is protein rather than DNA, or the goal is direct tree inference from
  unassembled reads (e.g. Read2Tree-style workflows) rather than marker retrieval
  and concatenation
- The downstream marker set is not restricted to near-USCO reference sequences, where
  Patchwork itself warns chimeric sequences may result
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: DIAMOND
  role: Performs the translated DNA-vs-protein similarity search whose tabular BLASTX
    output is parsed for hit regions
  repo: https://github.com/bbuchfink/diamond
- name: Patchwork
  role: Consumes the parsed DIAMOND tabular hits and performs hit stitching, alignment
    masking, sliding-window trimming, and concatenation
  repo: https://github.com/fethalen/Patchwork
- name: Julia
  role: Implementation language of Patchwork's parsing and stitching routines (BioSequences/BioAlignments
    ecosystem)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:5ab0a72a91cbd4e4e4bc488df84df77a6c06bd4cee8a3e268090b5772b14db60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bbuchfink/diamond
  tools:
  - DIAMOND
  - Julia
  - Patchwork
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# blastx-tabular-hit-parsing

## Summary

Parse DIAMOND BLASTX tabular output (query contigs searched against amino acid reference sequences) to extract per-reference hit regions, supplying the coordinates and HSPs that Patchwork's hit-stitching phase needs to merge adjacent or overlapping homologous regions into one continuous sequence per reference.

## When to use

Use when DNA query contigs (or reads) have been searched against protein reference sequences with DIAMOND and tabular BLASTX hits must be grouped per reference and ordered for merging into continuous marker sequences.

## When NOT to use

- The query is protein rather than DNA, or the goal is direct tree inference from unassembled reads (e.g. Read2Tree-style workflows) rather than marker retrieval and concatenation
- The downstream marker set is not restricted to near-USCO reference sequences, where Patchwork itself warns chimeric sequences may result

## Inputs

- DIAMOND BLASTX output in tabular (BLAST tabular) format
- Query DNA contig sequences (assembled contigs or raw reads) in FASTA
- Amino acid reference sequences (e.g. USCOs from OrthoDB or near-USCOs from an annotated genome) with a DIAMOND-formatted database

## Outputs

- Per-reference ordered stack of query hit regions with alignment coordinates and best-scoring HSPs
- stitched_hits.fasta with one continuous amino acid sequence per reference

## How to apply

Run DIAMOND blastx against the amino acid reference database (e.g. USCO/near-USCO references) with tabular-style output, and disable tantan masking (--masking 0) when working with read data since it reduces query coverage. Parse the tabular output together with the query contig sequences, and for each reference, sort query hit regions by their alignment coordinates on the target from first to last and push them onto a stack. Compare consecutive regions pairwise; when two regions overlap, record the overlap and retain only the highest-scoring segment pair (HSP) for each region. The parsed, ordered hit regions then feed Patchwork's stitching, which realigns overlaps, keeps the best-scoring HSP sequence per overlap, preserves nonoverlapping flanks, and concatenates the remaining ordered nonoverlapping regions into one continuous sequence per reference (stitched_hits.fasta).

## Related tools

- **DIAMOND** (Performs the translated DNA-vs-protein similarity search whose tabular BLASTX output is parsed for hit regions) — https://github.com/bbuchfink/diamond
- **Patchwork** (Consumes the parsed DIAMOND tabular hits and performs hit stitching, alignment masking, sliding-window trimming, and concatenation) — https://github.com/fethalen/Patchwork
- **Julia** (Implementation language of Patchwork's parsing and stitching routines (BioSequences/BioAlignments ecosystem))

## Examples

```
diamond blastx -d usco_refs.dmnd -q contigs.fasta -o diamond_hits.tsv --outfmt 6 --masking 0
```

## Evaluation signals

- stitched_hits.fasta is valid FASTA containing exactly one continuous sequence per reference
- Stitched output matches the deposited Patchwork implementation (github.com/fethalen/Patchwork) on the same DIAMOND hit input
- Each reference's hit regions are sorted by alignment coordinates on the target and the final sequence contains no overlapping intervals
- For read-based input, query coverage is consistent with tantan masking being disabled via --masking 0
- Aligned markers show sensible identity/coverage ranges (e.g. ~95.9% identical aligned positions and 92.2% mean query coverage in the same-species benchmark)

## Limitations

- Reference sequences should be limited to near-USCOs to avoid chimeric sequences
- Disabling DIAMOND tantan masking improves query coverage especially for read data, so masked results may under-report hits
- Unaligned residues, ambiguous amino acid characters, and stop codons must still be removed in the subsequent alignment masking step; parsing alone does not clean the sequence

## Evidence

- [methods] Hit Stitching: "all alignments made between the query region and the target sequence are merged in a way such that only the highest-scoring segment pair (HSP) for each region is retained"
- [abstract] Abstract workflow: "followed by a "hit stitching" phase, in which adjacent or overlapping regions are merged into a single unit"
- [results] Disabling tantan masking: "query coverage improves especially for read data when tantan masking in DIAMOND is disabled"
- [readme] Patchwork README: "Multiple, overlapping hits are merged to derive a single, continuous sequence for each reference sequence."
