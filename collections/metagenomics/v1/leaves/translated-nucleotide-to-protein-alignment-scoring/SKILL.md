---
name: translated-nucleotide-to-protein-alignment-scoring
description: Use when nucleotide query contigs or raw reads from genome-skimming (LC-WGS)
  assemblies must be aligned against protein reference markers to obtain homologous,
  scored regions for downstream stitching and trimming.
when_to_use_negative:
- The reference sequences are nucleotide rather than amino acid — DIAMOND BLASTX in
  Patchwork searches DNA queries against protein references
- The goal is de novo gene prediction or annotation rather than homology retrieval
  of known single-copy markers against curated references
- The input is already a stitched or trimmed marker alignment — downstream stages
  (masking, sliding-window trimming) apply instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: DIAMOND
  role: Performs the translated nucleotide-to-protein sequence similarity search producing
    scored HSP alignments for Patchwork
  repo: https://github.com/bbuchfink/diamond
- name: Patchwork
  role: Wraps DIAMOND BLASTX scoring and applies hit stitching, masking, and sliding-window
    trimming to the alignment
  repo: https://github.com/fethalen/Patchwork
- name: BioAlignments.jl
  role: Julia library providing sequence alignment algorithms and data structures
    used within Patchwork
  repo: https://github.com/BioJulia/BioAlignments.jl
- name: BioSequences.jl
  role: Julia library providing DNA and amino acid sequence data types for handling
    query and reference sequences
  repo: https://github.com/BioJulia/BioSequences.jl
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:83ad10814411dd5fc0fa7d3989c4071e4262b9b50de7934e73739560f9b9967f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bbuchfink/diamond
  tools:
  - BioAlignments.jl
  - BioSequences.jl
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
yes_rate: 1.0
n_claims: 10
---

# translated-nucleotide-to-protein-alignment-scoring

## Summary

Score alignments between translated DNA query contigs and amino acid reference sequences using DIAMOND BLASTX with BLOSUM62 and defined gap penalties, providing the scored alignment substrate that Patchwork's stitching, masking, and sliding-window trimming stages operate on.

## When to use

Use when nucleotide query contigs or raw reads from genome-skimming (LC-WGS) assemblies must be aligned against protein reference markers to obtain homologous, scored regions for downstream stitching and trimming.

## When NOT to use

- The reference sequences are nucleotide rather than amino acid — DIAMOND BLASTX in Patchwork searches DNA queries against protein references
- The goal is de novo gene prediction or annotation rather than homology retrieval of known single-copy markers against curated references
- The input is already a stitched or trimmed marker alignment — downstream stages (masking, sliding-window trimming) apply instead

## Inputs

- DNA query contigs from fragmented WGS assemblies or raw short reads (e.g., Illumina, FASTA/FASTQ)
- Amino acid reference marker sequences (e.g., 815 USCO references from D. gyrociliatus, GenBank GCA_904063045.1)
- DIAMOND database of protein reference sequences

## Outputs

- Scored query-to-reference alignments (DIAMOND BLASTX HSPs) consumed by Patchwork's hit-stitching stage
- Stitched marker sequences in FASTA format (trimmed_sequence.fasta after trimming)

## How to apply

Run DIAMOND BLASTX to align the provided DNA query (assembled contigs or reads) against amino acid reference sequences, using the BLOSUM62 substitution matrix with a gap open penalty of 11 and gap extension penalty of 1 as Patchwork's default scoring. The resulting query-to-reference alignment is loaded by Patchwork, which then merges adjacent or overlapping hits by retaining only the highest-scoring segment pair (HSP) per region. Note that disabling tantan masking in DIAMOND (--masking 0) improves query coverage, especially for read data. The scored alignment feeds Patchwork's subsequent alignment masking (removal of unaligned residues, ambiguous amino acids, and stop codons) and sliding-window-based alignment trimming of noncoding regions.

## Related tools

- **DIAMOND** (Performs the translated nucleotide-to-protein sequence similarity search producing scored HSP alignments for Patchwork) — https://github.com/bbuchfink/diamond
- **Patchwork** (Wraps DIAMOND BLASTX scoring and applies hit stitching, masking, and sliding-window trimming to the alignment) — https://github.com/fethalen/Patchwork
- **BioAlignments.jl** (Julia library providing sequence alignment algorithms and data structures used within Patchwork) — https://github.com/BioJulia/BioAlignments.jl
- **BioSequences.jl** (Julia library providing DNA and amino acid sequence data types for handling query and reference sequences) — https://github.com/BioJulia/BioSequences.jl

## Examples

```
patchwork --query contigs.fasta --reference uscos.fa --diamond-args "--masking 0"
```

## Evaluation signals

- With a high-quality query assembly and 815 same-species USCO references, all 815 markers should be retrieved with ~95.9% average identical aligned positions and 92.2% mean query coverage
- Against distant H. robusta references, retrieval of 943/957 sequences with ~89% average percent identity and 74.5% mean query coverage is expected
- Every query region retains only the highest-scoring segment pair (HSP) after hit stitching — overlapping/adjacent alignments must be merged into a single unit
- Query coverage should measurably increase when tantan masking is disabled via --masking 0, especially for read-based input
- Trimmed output (trimmed_sequence.fasta) must be valid FASTA, with regions removed exactly those whose average window distance exceeds the user-specified threshold

## Limitations

- Disabling tantan masking (--masking 0) trades sensitivity/coverage against potential spurious hits in low-complexity regions
- Distantly related references yield lower identity and coverage (e.g., only 769/943 H. robusta retrievals aligned back uniquely to D. gyrociliatus USCOs)
- At very low coverage (1x), read-based data may behave differently from assembly data; Patchwork with assemblies reached only up to 62% total percent identity for data with at least 10x coverage
- Recommending reference sequences be limited to near-USCOs to avoid chimeric sequences

## Evidence

- [abstract] Patchwork is an alignment-based tool that utilizes the sequence aligner DIAMOND: "Patchwork is an alignment-based tool that utilizes the sequence aligner DIAMOND"
- [abstract] Homologous regions are obtained via a sequence similarity search: "Homologous regions are obtained via a sequence similarity search"
- [methods] all alignments made between the query region and the target sequence are merged in a way such that only the highest-scoring segment pair (HSP) for each region is retained: "all alignments made between the query region and the target sequence are merged in a way such that only the highest-scoring segment pair (HSP) for each region is retained"
- [results] query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument): "query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument)"
- [readme] Patchwork searches the provided DNA query contigs against one or more amino acid reference sequences: "The program
searches the provided DNA query contigs against one or more amino acid reference
sequences."
