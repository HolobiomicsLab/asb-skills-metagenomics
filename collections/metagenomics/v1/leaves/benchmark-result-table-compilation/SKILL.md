---
name: benchmark-result-table-compilation
description: Use when several metagenome assemblers have been run on the same sample
  (e.g. MetaPlatanus vs metaFlye, OPERA-MS, MEGAHIT) and the goal is to tabulate scaffold
  counts, CheckM completeness/contamination, and reference-alignment edge error rates
  per assembler.
when_to_use_negative:
- You have only a single assembler's output and no competitor assemblies to compare
  — there is no comparison table to compile.
- You lack a reference genome for the target species, so alignment-based edge-error
  (minor 1k-mer) metrics cannot be computed.
- Your question concerns raw read QC or short-read preprocessing rather than assembly
  benchmarking.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: MetaPlatanus
  role: Assembler under evaluation; its output scaffolds (e.g. scaffold2094 of V06-T-0501-S07)
    form the MetaPlatanus row of the comparison table
  repo: https://github.com/rkajitani/MetaPlatanus
- name: Minimap2
  role: Aligns extracted scaffolds to the B. longum 51A reference using the assembly-to-reference
    preset (asm5) to compute minor 1k-mer edge rates
  repo: https://github.com/lh3/minimap2
- name: CheckM
  role: Estimates genome completeness and contamination for each assembler's extracted
    scaffold set
- name: metaFlye
  role: Competitor assembler whose B. longum scaffolds/contigs are tabulated alongside
    MetaPlatanus
- name: OPERA-MS
  role: Competitor assembler included in the comparison table
- name: MEGAHIT
  role: Competitor short-read assembler included in the comparison table; also used
    by MetaPlatanus for initial contigs
  repo: https://github.com/voutcn/megahit
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:642eb0720659ac755ab9cb32a628f9b19f2aa74a96555605acf8b17c9bbf84d5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/rkajitani/MetaPlatanus
  repo_url: https://github.com/rkajitani/MetaPlatanus
  tools:
  - CheckM
  - MEGAHIT
  - MetaPlatanus
  - Minimap2
  - OPERA-MS
  - metaFlye
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# benchmark-result-table-compilation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compiles assembler benchmark results into a per-assembler comparison table (TSV) that quantifies contiguity, completeness, contamination, and edge-level misassembly for a target genome. In the MetaPlatanus paper this is instantiated as the B. longum 51A scaffold comparison (e.g. scaffold2094 of sample V06-T-0501-S07) to show only MetaPlatanus yields a single near-complete scaffold while competitors fragment into >7 sequences.

## When to use

Use when several metagenome assemblers have been run on the same sample (e.g. MetaPlatanus vs metaFlye, OPERA-MS, MEGAHIT) and the goal is to tabulate scaffold counts, CheckM completeness/contamination, and reference-alignment edge error rates per assembler.

## When NOT to use

- You have only a single assembler's output and no competitor assemblies to compare — there is no comparison table to compile.
- You lack a reference genome for the target species, so alignment-based edge-error (minor 1k-mer) metrics cannot be computed.
- Your question concerns raw read QC or short-read preprocessing rather than assembly benchmarking.

## Inputs

- Assembly FASTA files for MetaPlatanus and competitor assemblers (metaFlye, OPERA-MS, MEGAHIT) from the FigShare deposit doi:10.6084/m9.figshare.15090603.v1
- Complete B. longum strain 51A reference genome FASTA
- Target scaffold/contig FASTA subsets (e.g. scaffold2094 from sample V06-T-0501-S07 and competitor equivalents)
- Minimap2 asm5 alignment outputs (PAF/SAM) of extracted scaffolds vs the 51A reference

## Outputs

- blongum_51A_scaffold_comparison.tsv — one row per assembler with scaffold counts, CheckM completeness, CheckM contamination, and minor 1k-mer edge rates
- Per-assembler alignment files against the B. longum 51A reference
- CheckM completeness/contamination estimates for each extracted scaffold set

## How to apply

Retrieve all assemblies under comparison from the study's FigShare deposit (doi:10.6084/m9.figshare.15090603.v1) and the reference genome (B. longum strain 51A). Extract the target sequences: e.g. scaffold2094 from the MetaPlatanus V06-T-0501-S07 assembly and the corresponding B. longum scaffolds/contigs from each competitor assembly. Align each extracted set to the reference with Minimap2 using the assembly-to-reference preset (asm5), then compute the minor 1k-mer rate within 1 kb of each scaffold edge to quantify residual error at termini. Run CheckM on each extracted set to obtain completeness and contamination estimates. Compile scaffold counts, CheckM completeness/contamination, and minor 1k-mer edge rates into one row per assembler and save as a TSV (blongum_51A_scaffold_comparison.tsv); the validation criterion is that the table exists in TSV format with a row for every assembler and reproduces the Figure 3 finding that only MetaPlatanus yields a single near-complete B. longum 51A scaffold.

## Related tools

- **MetaPlatanus** (Assembler under evaluation; its output scaffolds (e.g. scaffold2094 of V06-T-0501-S07) form the MetaPlatanus row of the comparison table) — https://github.com/rkajitani/MetaPlatanus
- **Minimap2** (Aligns extracted scaffolds to the B. longum 51A reference using the assembly-to-reference preset (asm5) to compute minor 1k-mer edge rates) — https://github.com/lh3/minimap2
- **CheckM** (Estimates genome completeness and contamination for each assembler's extracted scaffold set)
- **metaFlye** (Competitor assembler whose B. longum scaffolds/contigs are tabulated alongside MetaPlatanus)
- **OPERA-MS** (Competitor assembler included in the comparison table)
- **MEGAHIT** (Competitor short-read assembler included in the comparison table; also used by MetaPlatanus for initial contigs) — https://github.com/voutcn/megahit

## Examples

```
minimap2 -cx asm5 blongum_51A_ref.fa scaffold2094.fa > scaffold2094_vs_51A.paf
```

## Evaluation signals

- Output file blongum_51A_scaffold_comparison.tsv exists and parses as valid TSV with a row for every assembler evaluated (MetaPlatanus, metaFlye, OPERA-MS, MEGAHIT).
- MetaPlatanus row shows a single scaffold covering the B. longum 51A genome, while competitor rows show >7 sequences, reproducing the Figure 3 finding.
- CheckM completeness for the MetaPlatanus scaffold is consistent with a near-complete genome (low contamination) versus fragmented competitor sets.
- Minor 1k-mer rates within 1 kb of scaffold edges are low for the MetaPlatanus scaffold and elevated at competitor scaffold edges showing interspecies misassembly signs.
- Minimap2 alignments of scaffold2094 to the 51A reference confirm the MetaPlatanus scaffold structure is consistent with the complete reference genome.

## Limitations

- The article does not report tool versions for the compared assemblers (metaFlye, OPERA-MS, MEGAHIT, etc.), so exact reproduction may vary.
- The study focuses on de novo methods; reference information has not been well examined, so this comparison is not a reference-based evaluation of assembly algorithms.
- Key benchmark evidence is deferred to supplementary materials (Supplementary Tables S17, S24, S25; Figure 6), which may not be directly available.

## Evidence

- [supplementary] Assemblies generate in this study are available at FigShare (https://doi.org/10.6084/m9.figshare.15090603.v1).: "Assemblies generate in this study are available at FigShare (https://doi.org/10.6084/m9.figshare.15090603.v1)."
- [readme] minimap2 asm5 preset: [evidence span withheld — anchor status: external_doc]
- [abstract] fragmented competitor assemblies finding: "MetaPlatanus can circumvent the limitations of highly fragmented assemblies and frequent interspecies misassembles obtained by the other tools."
- [discussion] metaFlye benchmark context: "Although the metaFlye assembly exhibited high performance with large-scale accuracy and contiguity"
- [abstract] mock benchmark result motivating the table: "Benchmarking using mock datasets consisting of known bacteria with long reads or mate pairs revealed the high contiguity MetaPlatanus with a few interspecies misassemblies."
