---
name: reference-based-contiguity-assessment
description: Use when benchmarking metagenomic de novo assemblies against a known
  mock community and you need a per-replicon measure of contig contiguity (single-alignment
  coverage of a reference, not NGA50/total coverage).
when_to_use_negative:
- No reference replicon sequences are available (reference-free metrics such as contig-length
  distributions or N50 should be used instead).
- You need total reference coverage combining multiple disjoint alignments — LSA only
  measures the single longest contiguous alignment, so it will underestimate multi-contig
  coverage.
- Evaluating assembly accuracy (misassembly classes such as chimera, inversion, translocation)
  rather than contiguity — those are separate LMAS metrics.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: minimap2
  role: Aligns filtered-set contigs to reference replicon sequences, producing the
    PAF records from which the longest single alignment is extracted
- name: LMAS
  role: Automated Nextflow workflow that runs the assemblers, filters contigs by --minLength,
    computes the LSA metric, and writes lsa_metrics.csv
  repo: https://github.com/B-UMMI/LMAS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:7373d38d90d3b9238c9457582943018dca548b55fb0c1e62345f4da077e1c44a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - LMAS
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# reference-based-contiguity-assessment

## Summary

Computes the LSA (longest single alignment) contiguity metric per reference replicon: the length of the single longest contig-to-reference alignment (from minimap2 PAF records of filtered-set contigs) divided by the reference replicon length. This captures how much of each replicon a single contig spans, without merging disjoint alignments.

## When to use

Use when benchmarking metagenomic de novo assemblies against a known mock community and you need a per-replicon measure of contig contiguity (single-alignment coverage of a reference, not NGA50/total coverage).

## When NOT to use

- No reference replicon sequences are available (reference-free metrics such as contig-length distributions or N50 should be used instead).
- You need total reference coverage combining multiple disjoint alignments — LSA only measures the single longest contiguous alignment, so it will underestimate multi-contig coverage.
- Evaluating assembly accuracy (misassembly classes such as chimera, inversion, translocation) rather than contiguity — those are separate LMAS metrics.

## Inputs

- minimap2 PAF alignment file of FS (filtered-size) contigs against reference replicon sequences
- Contig FASTA files per assembler, already filtered by the --minLength threshold (default 1000 bp)
- Reference replicon FASTA sequences for the mock community (with known replicon lengths)

## Outputs

- lsa_metrics.csv with one LSA fraction per assembler-reference replicon pair, each value in (0, 1]
- Per-replicon LSA values consumed by the LMAS interactive HTML report

## How to apply

Load the minimap2 alignment records (PAF) of the filtered set (FS) contigs — those with length >= the --minLength parameter, default 1000 bp — aligned against each reference replicon sequence of the mock community. For each assembler and each reference replicon, extract all single contiguous contig-to-replicon alignments and identify the longest one; do not chain or merge alignments, since LSA is deliberately a single-alignment measure. Normalize the longest alignment length by the reference replicon length to obtain the LSA fraction, which is bounded between 0 and 1. Aggregate the LSA values per assembler per reference into a table and write it to lsa_metrics.csv. The metric complements QUAST/MetaQUAST-style reference-based evaluation, which LMAS performs by aligning contigs to a reference.

## Related tools

- **minimap2** (Aligns filtered-set contigs to reference replicon sequences, producing the PAF records from which the longest single alignment is extracted)
- **LMAS** (Automated Nextflow workflow that runs the assemblers, filters contigs by --minLength, computes the LSA metric, and writes lsa_metrics.csv) — https://github.com/B-UMMI/LMAS

## Evaluation signals

- lsa_metrics.csv exists and contains exactly one LSA entry per assembler-reference replicon pair.
- Every LSA value is bounded in (0, 1], consistent with its definition as a fraction of the reference replicon length.
- LSA approaches 1 only when a single contig's alignment spans (nearly) the whole replicon; contigs split across a replicon yield lower single-alignment values.
- The contig set used matches the filtered set: all contigs below the --minLength default of 1000 bp are excluded before alignment.
- LSA values are computed from minimap2 PAF records and reflect single contiguous alignments only (no chaining/merging of alignment blocks).

## Limitations

- LSA measures only the longest single contiguous alignment, so fragmented but complete assemblies are penalized relative to single-contig recovery of a replicon.
- Results depend on the contig length filter: LMAS default parameters use a minimum contig size (--minLength), and the paper notes assembler differences were attenuated after filtering for contigs <1 kbp.
- LMAS results reflect default assembler parameters; the paper notes 'their usability may be limited, at least with the default parameters we used'.

## Evidence

- [intro] MetaQUAST [11], a modification of QUAST, extends the original software by performing assembly evaluation based on aligning contigs to a reference: "MetaQUAST [11], a modification of QUAST, extends the original software by performing assembly evaluation based on aligning contigs to a reference"
- [methods] minimum contig size filter is defined by --minLength: "The **X** in the **Contigs > X** section corresponds to the minimum contig size as defined in the ``--minLength`` parameter"
- [readme] LMAS workflow description: [evidence span withheld — anchor status: not_found]
- [discussion] single-pipeline integration rationale: "LMAS is the only software-integrating assembly and its evaluation into a single pipeline, guaranteeing the same conditions are met for all tools."
