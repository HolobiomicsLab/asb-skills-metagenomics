---
name: reference-replicon-alignment-analysis
description: 'Use when evaluating de novo metagenomic assemblies against defined mock
  communities: you have assembled contigs (FASTA) and reference replicons and need
  per-contig alignment identity plus a Phred-like Pls quality score per assembler.'
when_to_use_negative:
- The sample has no known reference replicons (LMAS evaluation relies on aligning
  contigs to defined mock-community references)
- You only need assembly statistics intrinsic to the contigs (e.g. N50, QUAST-style
  feature checks) rather than reference-based per-contig quality
- Evaluating assemblies of eukaryotic or non-mock-community samples for which LMAS
  benchmarking against defined communities does not apply
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: LMAS
  role: Automated Nextflow workflow that aligns contigs to reference replicons and
    computes per-contig Pls quality scores for benchmarking assemblers
  repo: https://github.com/B-UMMI/LMAS
- name: QUAST
  role: Contrasting reference-free contig feature-based evaluation, which LMAS complements
    with reference-replicon alignment metrics
- name: MetaQUAST
  role: Reference-based assembly evaluation by aligning contigs to a reference; conceptually
    related precursor to the LMAS approach
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:5529885b980ef267932169ca0c3fcfd4c6db85963f51635f17dd7d3db4a1a6cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - LMAS
  - MetaQUAST
  - QUAST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# reference-replicon-alignment-analysis

## Summary

Aligns assembled contigs back to known reference replicons within LMAS and converts per-alignment identity into a Phred-like per-contig quality score (Pls), quantifying how faithfully each contig was assembled against the mock-community reference.

## When to use

Use when evaluating de novo metagenomic assemblies against defined mock communities: you have assembled contigs (FASTA) and reference replicons and need per-contig alignment identity plus a Phred-like Pls quality score per assembler.

## When NOT to use

- The sample has no known reference replicons (LMAS evaluation relies on aligning contigs to defined mock-community references)
- You only need assembly statistics intrinsic to the contigs (e.g. N50, QUAST-style feature checks) rather than reference-based per-contig quality
- Evaluating assemblies of eukaryotic or non-mock-community samples for which LMAS benchmarking against defined communities does not apply

## Inputs

- Assembled contig FASTA files, one per assembler (contigs above the minimum contig size threshold, 1000 bp via --minLength)
- Reference replicon sequences for the defined mock community (e.g. ZymoBIOMICS Microbial Community Standards, BMock12, Gut-Mix-RR)
- Contig-to-reference alignment results per assembler (per-alignment identity values)

## Outputs

- pls_scores.csv with per-contig PLS values tagged by assembler and reference replicon
- Reference replicon-specific performance metrics surfaced in the interactive HTML report (Contigs > X section, X = --minLength)

## How to apply

Run the LMAS workflow, which aligns contigs produced by each assembler (above the --minLength threshold, 1000 bp by default) to each reference replicon of the mock community. For every aligned contig, compute the error rate E = 1 - Identity, where Identity is the fraction of the alignment matching the reference replicon. Convert E to the Phred-like Pls via Equation 1, -10 * log10(E), capping the score at 60 when E = 0. Tag each per-contig Pls value with the assembler and reference and write the results to pls_scores.csv. This leverages the fact that LMAS integrates assembly and evaluation into a single pipeline, so all assemblers are scored under identical conditions.

## Related tools

- **LMAS** (Automated Nextflow workflow that aligns contigs to reference replicons and computes per-contig Pls quality scores for benchmarking assemblers) — https://github.com/B-UMMI/LMAS
- **QUAST** (Contrasting reference-free contig feature-based evaluation, which LMAS complements with reference-replicon alignment metrics)
- **MetaQUAST** (Reference-based assembly evaluation by aligning contigs to a reference; conceptually related precursor to the LMAS approach)

## Evaluation signals

- pls_scores.csv exists and contains a PLS value for every aligned contig (contigs above the --minLength threshold that map to a reference replicon)
- No per-contig Pls value exceeds the cap of 60
- Each Pls equals -10 * log10(1 - Identity) for the corresponding contig alignment, with 60 substituted when Identity = 1 (E = 0)
- Pls values can be grouped by assembler and reference replicon without missing or dangling tags
- Contigs below the minimum size threshold (1000 bp default) are absent from the score table

## Limitations

- Only contigs above the --minLength parameter threshold (1000 bp default) are scored; short contigs are excluded
- Assemblies with no contigs mapping to a reference replicon yield no Pls values for that reference
- Strain-level (SNP-level) discrimination was not achieved even by the best assemblers tested, so Pls should not be read as strain-resolution evidence

## Evidence

- [methods] min contig size filter: "The **X** in the **Contigs > X** section corresponds to the minimum contig size as defined in the ``--minLength`` parameter"
- [discussion] integrated evaluation: "LMAS is the only software-integrating assembly and its evaluation into a single pipeline, guaranteeing the same conditions are met for all tools."
- [readme] LMAS purpose: [evidence span withheld — anchor status: not_found]
- [intro] report metrics: "The results of LMAS are presented in an interactive HTML report where selected global and reference replicon-specific performance metrics can be explored."
