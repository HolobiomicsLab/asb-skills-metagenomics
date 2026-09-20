---
name: assembly-contig-alignment-metrics
description: Use when benchmarking metagenomic assemblers against a known mock community
  with reference replicons, and you need a per-replicon contiguity fraction (LSA)
  from contig-vs-reference alignments.
when_to_use_negative:
- No reference replicons are available (LSA is a reference-based metric requiring
  defined mock community ground truth).
- Evaluating non-prokaryotic or eukaryotic assembly targets — LMAS was designed for
  short-read prokaryotic replicon assembly.
- Input is raw reads or unassembled contigs without an assembler comparison goal —
  LSA only meaningfully ranks assemblies, not raw data quality.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: LMAS
  role: Workflow that filters contigs, orchestrates minimap2 alignment of FS contigs
    to reference replicons, computes LSA, and writes lsa_metrics.csv
  repo: https://github.com/B-UMMI/LMAS
- name: minimap2
  role: Aligner producing the PAF records of FS contigs against reference replicon
    sequences from which the longest single alignment is extracted
- name: LMAS.js
  role: Renders the interactive HTML report consuming the aggregated LSA metrics
  repo: https://github.com/B-UMMI/LMAS.js
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:a785994a58ff686fa002e8e059769af396829422c1eb3a6a0a5a76ac6d2cf546
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - LMAS
  - LMAS.js
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# assembly-contig-alignment-metrics

## Summary

Computes reference-based contiguity metrics (notably LSA, the longest single alignment fraction) for metagenomic de novo assemblies by aligning filtered-size contigs against reference replicons with minimap2 and normalizing alignment lengths by replicon length.

## When to use

Use when benchmarking metagenomic assemblers against a known mock community with reference replicons, and you need a per-replicon contiguity fraction (LSA) from contig-vs-reference alignments.

## When NOT to use

- No reference replicons are available (LSA is a reference-based metric requiring defined mock community ground truth).
- Evaluating non-prokaryotic or eukaryotic assembly targets — LMAS was designed for short-read prokaryotic replicon assembly.
- Input is raw reads or unassembled contigs without an assembler comparison goal — LSA only meaningfully ranks assemblies, not raw data quality.

## Inputs

- minimap2 alignment records (PAF) of FS (filtered-size, contigs >= --minLength, default 1000 bp) contigs against reference replicon sequences
- Reference replicon sequences (FASTA) defining mock community ground truth
- Per-assembler assemblies (contig FASTA) after minimum contig size filtering

## Outputs

- lsa_metrics.csv with one LSA entry per assembler-reference pair, values in (0, 1]
- Per-replicon LSA values feeding the interactive HTML report's global and reference-specific performance metrics

## How to apply

Load the minimap2 pairwise alignment records (PAF) produced by aligning the filtered set (FS) contigs — contigs passing the --minLength threshold, default 1000 bp — against the reference replicon sequences. For each assembler and each reference replicon, extract all single contiguous alignments between FS contigs and that replicon and identify the longest one. Normalize the longest alignment length by the reference replicon length to obtain the LSA fraction, bounded between 0 and 1; a value of 1 means a single contig alignment spans the entire replicon. Aggregate LSA values per assembler per reference into lsa_metrics.csv, which feeds the global and reference replicon-specific metrics in the interactive HTML report. Rationale: LMAS integrates assembly and evaluation in a single pipeline so all assemblers are compared under identical conditions using the same filtered contig set and the same alignments.

## Related tools

- **LMAS** (Workflow that filters contigs, orchestrates minimap2 alignment of FS contigs to reference replicons, computes LSA, and writes lsa_metrics.csv) — https://github.com/B-UMMI/LMAS
- **minimap2** (Aligner producing the PAF records of FS contigs against reference replicon sequences from which the longest single alignment is extracted)
- **LMAS.js** (Renders the interactive HTML report consuming the aggregated LSA metrics) — https://github.com/B-UMMI/LMAS.js

## Evaluation signals

- lsa_metrics.csv exists and contains exactly one LSA entry per assembler-reference pair
- Every LSA value falls within (0, 1], consistent with its definition as the longest single alignment length divided by the reference replicon length
- An LSA of 1 is only achieved when a single contig alignment spans the full replicon; fragmented assemblies should show lower LSA values
- LSA values derive only from contigs that passed the --minLength filter (default 1000 bp), so very short contig alignments must be absent from the calculation

## Limitations

- Computationally intensive because it requires de novo assembly of the metagenomic samples prior to metric computation.
- Results were only assessed with LMAS default parameters; non-default parameter tuning was not explored, so LSA behavior under tuned assembler settings is unknown.
- Strain-level (SNP-level) discrimination is not achievable from assembler-generated contigs alone, so LSA does not capture sub-species resolution.
- No single assembler is universally best, so LSA per reference must be read in context — results may not generalize across community compositions.

## Evidence

- [methods] minLength parameter: "The **X** in the **Contigs > X** section corresponds to the minimum contig size as defined in the ``--minLength`` parameter"
- [discussion] single pipeline: "LMAS is the only software-integrating assembly and its evaluation into a single pipeline, guaranteeing the same conditions are met for all tools."
- [results] run with defaults: "the LMAS workflow was run 3 times for all samples using default parameters"
- [readme] LMAS overview: [evidence span withheld — anchor status: not_found]
- [results] filtering effects: "the differences between them tended to be attenuated after filtering for contigs <1 kbp"
