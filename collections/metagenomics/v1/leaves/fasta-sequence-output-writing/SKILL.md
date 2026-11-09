---
name: fasta-sequence-output-writing
description: Use when merging overlapping DIAMOND BLASTX hits into continuous per-reference
  sequences or trimming noncoding regions, and the resulting marker sequences must
  be emitted as FASTA for phylogenetic use.
when_to_use_negative:
- The data are raw DIAMOND tabular BLASTX hits that have not yet been stitched or
  masked — FASTA emission should happen after overlap resolution, not on raw HSP tables
- The input is unassembled short reads (FASTQ); Patchwork can search raw reads, but
  this FASTA-writing step applies to the resulting stitched/trimmed marker sequences,
  not the read files
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0092
tools:
- name: Patchwork
  role: Pipeline whose stitching and trimming stages produce and write the stitched_hits.fasta
    and trimmed_sequence.fasta outputs
  repo: https://github.com/fethalen/Patchwork
- name: BioSequences.jl
  role: Julia package providing DNA/amino-acid sequence types used to hold and emit
    the stitched and trimmed marker sequences in FASTA
  repo: https://github.com/BioJulia/BioSequences.jl
- name: DIAMOND
  role: Produces the tabular BLASTX hits whose merged, trimmed residues are the content
    written to the FASTA outputs
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:8a617aa19a73d40fe881cd82487fc97511135322f8d65105fb4b24c69dee7168
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - BioSequences.jl
  - DIAMOND
  - Julia
  - Patchwork
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# fasta-sequence-output-writing

## Summary

Serialize the nucleotide/translated sequences produced by Patchwork's hit-stitching and sliding-window alignment-trimming stages into FASTA files (stitched_hits.fasta, trimmed_sequence.fasta), one continuous sequence per reference. This is the terminal output step that makes retrieved phylogenetic markers usable downstream.

## When to use

Use when merging overlapping DIAMOND BLASTX hits into continuous per-reference sequences or trimming noncoding regions, and the resulting marker sequences must be emitted as FASTA for phylogenetic use.

## When NOT to use

- The data are raw DIAMOND tabular BLASTX hits that have not yet been stitched or masked — FASTA emission should happen after overlap resolution, not on raw HSP tables
- The input is unassembled short reads (FASTQ); Patchwork can search raw reads, but this FASTA-writing step applies to the resulting stitched/trimmed marker sequences, not the read files

## Inputs

- Ordered, nonoverlapping stitched hit regions (one set per reference protein) derived from DIAMOND tabular BLASTX output and query contigs
- Trimmed coding sequence resulting from sliding window–based alignment trimming of the stitched query-to-reference alignment

## Outputs

- stitched_hits.fasta — FASTA with one continuous sequence per reference
- trimmed_sequence.fasta — FASTA of the trimmed coding sequence per reference

## How to apply

After Patchwork concatenates the ordered, nonoverlapping DIAMOND hit regions into one continuous sequence per reference, write all stitched sequences to stitched_hits.fasta in FASTA format (one header and one continuous sequence per reference protein). After the final sliding window–based alignment trimming stage — which cuts regions whose average gap-excluded query-reference distance exceeds the user-provided distance threshold — write the trimmed coding sequence to trimmed_sequence.fasta. Use Julia sequence types (e.g. BioSequences.jl DNASequence) so headers and residues are emitted without altering the residue content established by the preceding masking and stitching steps. Validate that the written FASTA parses cleanly and that the per-reference stitched sequences reproduce the behavior of the deposited Patchwork implementation (github.com/fethalen/Patchwork) on the same DIAMOND tabular input.

## Related tools

- **Patchwork** (Pipeline whose stitching and trimming stages produce and write the stitched_hits.fasta and trimmed_sequence.fasta outputs) — https://github.com/fethalen/Patchwork
- **BioSequences.jl** (Julia package providing DNA/amino-acid sequence types used to hold and emit the stitched and trimmed marker sequences in FASTA) — https://github.com/BioJulia/BioSequences.jl
- **DIAMOND** (Produces the tabular BLASTX hits whose merged, trimmed residues are the content written to the FASTA outputs) — https://github.com/bbuchfink/diamond

## Examples

```
using BioSequences; seq = DNASequence("ACGTACGT"); println(seq)
```

## Evaluation signals

- stitched_hits.fasta parses as valid FASTA and contains exactly one continuous sequence per reference protein
- Per-reference stitched output matches the result of the deposited Patchwork implementation (github.com/fethalen/Patchwork) on the same DIAMOND hit input
- trimmed_sequence.fasta contains exactly the regions whose average window distance was at or below the threshold; removed regions are exactly those above it
- Trimmed sequences attain high aligned-position identity comparable to Patchwork's benchmarks (e.g. ~95.9% identical aligned positions for same-species references)

## Limitations

- Choosing the correct distance-cutoff threshold for trimming becomes increasingly easier only as identity approaches 100%, so output quality is threshold-sensitive for distant references
- Query coverage of the written sequences can drop for read data when DIAMOND's tantan masking is active (disable with --masking 0); low-coverage inputs (around 1x) may yield poorer retrieved markers than assembly inputs
- Reference sequences should be limited to near-USCOs to avoid chimeric output sequences

## Evidence

- [methods] concatenation of stitched regions: "the resulting set of ordered, nonoverlapping sequence regions are concatenated into 1 continuous sequence"
- [abstract] sliding window trimming: "The novel sliding window algorithm trims away any noncoding regions from the resulting sequence"
- [readme] README feature statement: "Multiple, overlapping hits are merged to derive a single, continuous
sequence for each reference sequence."
- [results] benchmark quality of retrieved markers: "On average, 95.9% of all aligned positions were identical matches, with a mean query coverage of 92.2%"
