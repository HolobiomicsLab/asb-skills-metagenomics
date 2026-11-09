---
name: per-replicon-metric-normalization
description: Use when benchmarking metagenomic de novo assemblers against a defined
  mock community and you need a per-reference-replicon completeness fraction (LSA)
  from contig-to-reference alignments.
when_to_use_negative:
- The goal is a global assembly metric (e.g. N50, total length) rather than a per-replicon
  completeness fraction
- Input assemblies have not been filtered by the minimum contig size (--minLength,
  default 1000 bp), so contig lengths are not comparable across assemblers
- There is no reference replicon set available for the mock community (LSA is reference-dependent)
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: LMAS
  role: Automated Nextflow workflow that assembles the mock community reads with multiple
    assemblers, filters contigs, and computes/aggregates the per-replicon LSA metric
    into lsa_metrics.csv
  repo: https://github.com/B-UMMI/LMAS
- name: minimap2
  role: Aligns the filtered-size (FS) contigs against the reference replicon sequences,
    producing the PAF alignment records from which the longest single alignment per
    replicon is extracted
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:59d267e8f304d90b99a492df8f5762f48924bb4d314ef075e4bd8bb37f4bce03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - LMAS
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# per-replicon-metric-normalization

## Summary

Computes the LMAS LSA (longest single alignment) metric per reference replicon: the length of the longest single contiguous contig-to-reference alignment (from minimap2 PAF records of filtered-size contigs) normalized by the reference replicon length. This yields a per-assembler, per-replicon completeness fraction bounded in (0, 1] rather than a global assembly statistic.

## When to use

Use when benchmarking metagenomic de novo assemblers against a defined mock community and you need a per-reference-replicon completeness fraction (LSA) from contig-to-reference alignments.

## When NOT to use

- The goal is a global assembly metric (e.g. N50, total length) rather than a per-replicon completeness fraction
- Input assemblies have not been filtered by the minimum contig size (--minLength, default 1000 bp), so contig lengths are not comparable across assemblers
- There is no reference replicon set available for the mock community (LSA is reference-dependent)

## Inputs

- minimap2 PAF alignment records of FS contigs against reference replicon sequences
- Filtered-size (FS) contig FASTA files, i.e. contigs >= --minLength (default 1000 bp)
- Reference replicon sequences (per-replicon FASTA) for the mock community

## Outputs

- lsa_metrics.csv with one LSA fraction per assembler-reference pair
- Per-assembler, per-reference LSA values bounded in (0, 1]

## How to apply

Load the minimap2 alignment records (PAF) of the FS (filtered-size) contigs — contigs >= the --minLength default of 1000 bp — aligned against the reference replicon sequences. For each assembler and each reference replicon, extract all single contiguous alignments between contigs and that replicon and identify the longest one. Normalize the longest alignment length by the reference replicon length to obtain the LSA fraction, bounded between 0 and 1. Aggregate the LSA values per assembler per reference into a table written to lsa_metrics.csv; the normalization by replicon length (rather than total reference size) is what makes the metric interpretable per replicon in a multi-replicon mock community.

## Related tools

- **LMAS** (Automated Nextflow workflow that assembles the mock community reads with multiple assemblers, filters contigs, and computes/aggregates the per-replicon LSA metric into lsa_metrics.csv) — https://github.com/B-UMMI/LMAS
- **minimap2** (Aligns the filtered-size (FS) contigs against the reference replicon sequences, producing the PAF alignment records from which the longest single alignment per replicon is extracted)

## Examples

```
minimap2 reference_replicons.fasta fs_contigs.fasta > fs_vs_reference.paf
```

## Evaluation signals

- lsa_metrics.csv exists after the workflow run
- The file contains exactly one LSA entry per assembler-reference pair
- Every LSA value falls within (0, 1], consistent with its definition as a fraction of the reference replicon length
- The alignments used derive from the filtered set (FS) of contigs >= --minLength (default 1000 bp), not the raw assemblies
- A replicon fully covered by a single contig alignment yields LSA == 1

## Limitations

- LSA captures only the longest single contiguous alignment, so a replicon reconstructed as several shorter contigs scores low despite high total coverage
- The metric depends on the --minLength filter (default 1000 bp); differences between assemblers tend to be attenuated after filtering for contigs <1 kbp
- Computed per replicon only for defined mock communities with known reference sequences; not applicable without a reference

## Evidence

- [methods] LMAS computes the LSA metric per reference as the fraction of the longest single alignment between a contig and the reference, relative to the reference replicon length, using minimap2 alignments of the filtered set (FS) contigs.: [evidence span withheld — anchor status: not_found]
- [methods] min contig length: "The **X** in the **Contigs > X** section corresponds to the minimum contig size as defined in the ``--minLength`` parameter"
- [results] filtering attenuates differences: "the differences between them tended to be attenuated after filtering for contigs <1 kbp"
- [discussion] single pipeline guarantee: "LMAS is the only software-integrating assembly and its evaluation into a single pipeline, guaranteeing the same conditions are met for all tools."
- [abstract] LMAS platform purpose: "LMAS (Last Metagenomic Assembler Standing) was developed as a flexible platform allowing users to evaluate assembler performance given known standard communities."
