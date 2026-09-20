---
name: sliding-window-alignment-trimming
description: Use when a stitched marker sequence from fragmented genome-skimming (LC-WGS)
  assemblies contains noncoding stretches — many indels with occasional matches —
  that must be excised before concatenation into a phylogenetic marker.
when_to_use_negative:
- The input has not yet passed hit stitching and alignment masking — trimming is the
  final processing stage and expects a masked, stitched alignment
- The goal is to trim artifact regions from an existing multi-sequence alignment post
  hoc; the article recommends PREQUAL for detecting and removing such noncoding stretches
  downstream
- The query is not a translated nucleotide-to-protein alignment against near-USCO
  reference sequences; the authors recommend limiting references to near-USCOs to
  avoid chimeric sequences
edam_operation: http://edamontology.org/operation_0259
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Patchwork
  role: Implements the sliding window-based alignment trimming of the stitched DIAMOND
    BLASTX alignment
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: Produces the query-to-reference BLASTX alignment (BLOSUM62, gap open 11, gap
    extend 1) that is input to trimming
  repo: https://github.com/bbuchfink/diamond
- name: Julia
  role: Programming language in which Patchwork and its trimming algorithm are written
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:4d6eaff1efdc69a98240debb50258fbbf6098081851a90e8a13d893bfd32af65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - DIAMOND
  - Julia
  - Patchwork
  edam_operation: http://edamontology.org/operation_0259
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# sliding-window-alignment-trimming

## Summary

Patchwork's sliding window-based alignment trimming removes poorly aligned residues, such as those from putative noncoding regions, from stitched query-to-reference DIAMOND BLASTX alignments as the final processing stage after hit stitching and masking.

## When to use

Use when a stitched marker sequence from fragmented genome-skimming (LC-WGS) assemblies contains noncoding stretches — many indels with occasional matches — that must be excised before concatenation into a phylogenetic marker.

## When NOT to use

- The input has not yet passed hit stitching and alignment masking — trimming is the final processing stage and expects a masked, stitched alignment
- The goal is to trim artifact regions from an existing multi-sequence alignment post hoc; the article recommends PREQUAL for detecting and removing such noncoding stretches downstream
- The query is not a translated nucleotide-to-protein alignment against near-USCO reference sequences; the authors recommend limiting references to near-USCOs to avoid chimeric sequences

## Inputs

- Stitched query-to-reference alignment from DIAMOND BLASTX (after hit stitching and alignment masking)
- User-specified window size (optional; built-in default used if absent)
- User-specified distance threshold (optional; built-in default used if absent)

## Outputs

- Trimmed coding sequence in FASTA format (trimmed_sequence.fasta)

## How to apply

Load the stitched query-to-reference alignment produced by DIAMOND BLASTX (default BLOSUM62 substitution matrix, gap open penalty 11, gap extension penalty 1) after hit stitching and alignment masking have removed unaligned residues, ambiguous amino acid characters, and stop codons. Read the user-provided window size and distance threshold, falling back to Patchwork's built-in defaults if not supplied. Scan the alignment left to right in windows, computing the average gap-excluded, identity-based distance between query and reference residues per window. Cut every region where the average query-reference distance exceeds the distance threshold, which removes noncoding stretches characterized by many indels with occasional matches. The window-average criterion (rather than single positions) preserves isolated bad-but-correct matches; write the trimmed coding sequence to trimmed_sequence.fasta.

## Related tools

- **Patchwork** (Implements the sliding window-based alignment trimming of the stitched DIAMOND BLASTX alignment) — https://github.com/fethalen/Patchwork
- **DIAMOND** (Produces the query-to-reference BLASTX alignment (BLOSUM62, gap open 11, gap extend 1) that is input to trimming) — https://github.com/bbuchfink/diamond
- **Julia** (Programming language in which Patchwork and its trimming algorithm are written)

## Evaluation signals

- Regions removed from the alignment are exactly those whose average window distance (gap-excluded, identity-based) exceeds the user's distance threshold
- Isolated bad-but-correct matches are preserved because cutting depends on the window-average distance failing, not single positions
- For same-species references (e.g., 815 D. gyrociliatus USCOs), trimmed sequences attain ~95.9% identical aligned positions and ~92.2% mean query coverage
- Most markers finish well above the 90% aligned-position trimming threshold used in the benchmark
- Output file trimmed_sequence.fasta is valid FASTA containing the trimmed coding sequence

## Limitations

- Choosing the correct cutoff distance threshold becomes increasingly difficult as reference-organism identity departs from 100%; no specific default cutoff values are stated in the article
- Distant references (e.g., H. robusta vs. D. gyrociliatus) yield lower identity (~89%) and coverage (74.5%), so trimming may remove more legitimate sequence
- Noncoding stretches that survive trimming can be handled downstream with PREQUAL, but trimming alone does not guarantee all artifactual regions are removed

## Evidence

- [abstract] Sliding window algorithm as final stage: "The novel sliding window algorithm trims away any noncoding regions from the resulting sequence"
- [methods] Rationale for trimming: "we have implemented a sliding window–based alignment trimming approach to rid the alignments from these unwanted regions"
- [methods] Masking precedes trimming: "unaligned residues, ambiguous amino acid characters, and stop codons (also known as “termination codons”) are all removed from the resulting query sequence."
- [methods] Hit stitching precedes trimming: "all alignments made between the query region and the target sequence are merged in a way such that only the highest-scoring segment pair (HSP) for each region is retained"
- [results] Benchmark identity expectation: "On average, 95.9% of all aligned positions were identical matches, with a mean query coverage of 92.2%"
- [discussion] Threshold difficulty caveat: "choosing the correct cutoff threshold becomes increasingly easier as the level of identity approaches 100%"
- [discussion] PREQUAL alternative downstream: "the tool PREQUAL could be used to detect such and remove such regions"
