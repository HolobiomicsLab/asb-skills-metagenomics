---
name: blosum62-substitution-matrix-scoring
description: Use when trimming stitched DIAMOND BLASTX alignments of noncoding stretches,
  or when computing gap-excluded identity-based query-reference distances per window
  for coding-sequence cleanup in phylogenetic marker retrieval.
when_to_use_negative:
- The query is raw unaligned reads or contigs that have not yet been stitched against
  the amino acid reference — masking/stitching must precede trimming
- Downstream alignment cleanup where a dedicated noncoding-region detector such as
  PREQUAL is preferred by the user
- The reference sequences are not near-universal single-copy orthologs (near-USCOs),
  where trimming thresholds become hard to choose and chimeric sequences are likely
edam_operation: http://edamontology.org/operation_0297
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0194
tools:
- name: Patchwork
  role: Implements the sliding window-based alignment trimming that applies BLOSUM62-based
    scoring to remove noncoding regions
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: Produces the BLASTX alignment scored under BLOSUM62 (gap open 11, gap extension
    1) that feeds the trimming step
  repo: https://github.com/bbuchfink/diamond
- name: Julia
  role: Programming language in which Patchwork's scoring and trimming algorithms
    are written
  repo: https://github.com/fethalen/Patchwork
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:e7ca42f272181e4feb05651cc6df23a2a4425af365042fb5147b698caf095687
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - DIAMOND
  - Julia
  - Patchwork
  edam_operation: http://edamontology.org/operation_0297
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# blosum62-substitution-matrix-scoring

## Summary

Scores query-to-reference alignments with the BLOSUM62 substitution matrix (gap open 11, gap extension 1) inside Patchwork's sliding window trimming, so that poorly aligned, putatively noncoding regions stitched from genome-skimming assemblies can be detected and removed.

## When to use

Use when trimming stitched DIAMOND BLASTX alignments of noncoding stretches, or when computing gap-excluded identity-based query-reference distances per window for coding-sequence cleanup in phylogenetic marker retrieval.

## When NOT to use

- The query is raw unaligned reads or contigs that have not yet been stitched against the amino acid reference — masking/stitching must precede trimming
- Downstream alignment cleanup where a dedicated noncoding-region detector such as PREQUAL is preferred by the user
- The reference sequences are not near-universal single-copy orthologs (near-USCOs), where trimming thresholds become hard to choose and chimeric sequences are likely

## Inputs

- Stitched query-to-reference alignment from DIAMOND BLASTX (stitched_sequence output)
- User-provided window size and distance threshold parameters

## Outputs

- Trimmed coding sequence in trimmed_sequence.fasta (FASTA format)

## How to apply

Load the stitched query-to-reference alignment produced by DIAMOND BLASTX, which Patchwork scores with the BLOSUM62 substitution matrix (gap open penalty 11, gap extension penalty 1). Scan the alignment left to right in user-defined windows (falling back to built-in defaults), computing the average gap-excluded, identity-based distance between query and reference residues per window. Cut regions where the average window distance exceeds the user-supplied distance threshold, thereby removing noncoding stretches characterized by many indels with occasional matches; requiring the criterion to fail on the window average rather than single positions preserves isolated bad-but-correct matches. Write the trimmed coding sequence to trimmed_sequence.fasta and verify that removed regions are exactly those above the threshold and that aligned-position identity is high (e.g., ~95.9% for same-species references).

## Related tools

- **Patchwork** (Implements the sliding window-based alignment trimming that applies BLOSUM62-based scoring to remove noncoding regions) — https://github.com/fethalen/Patchwork
- **DIAMOND** (Produces the BLASTX alignment scored under BLOSUM62 (gap open 11, gap extension 1) that feeds the trimming step) — https://github.com/bbuchfink/diamond
- **Julia** (Programming language in which Patchwork's scoring and trimming algorithms are written) — https://github.com/fethalen/Patchwork

## Evaluation signals

- Regions removed are exactly those whose average window distance exceeds the distance threshold (window-average criterion, not single positions)
- Trimmed sequences show high aligned-position identity, comparable to Patchwork's reported ~95.9% identical aligned positions for same-species references
- Output file trimmed_sequence.fasta is valid FASTA and contains one continuous sequence per reference after concatenation/realignment
- Gap-excluded identity distances are computed with gaps excluded, matching Patchwork's identity-based distance definition

## Limitations

- Choosing the correct cutoff threshold becomes increasingly easier only as the level of identity approaches 100%; no specific default cutoff values are stated in the discussion
- Poorly chosen reference sequences (not near-USCOs) risk chimeric sequences and unreliable distance estimates
- Tantan masking effects: disabling DIAMOND masking (--masking 0) improves query coverage especially for read data, which changes what the trimming step sees

## Evidence

- [methods] sliding window trimming rationale: "we have implemented a sliding window–based alignment trimming approach to rid the alignments from these unwanted regions"
- [abstract] novel algorithm description: "The novel sliding window algorithm trims away any noncoding regions from the resulting sequence"
- [intro] alignment source tool: "Patchwork utilizes the sequence aligner DIAMOND (Buchfink et al. 2021)"
- [results] trimming threshold benchmark: "most markers were well above the 90% aligned position trimming threshold"
- [results] masking caveat: "query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument)"
