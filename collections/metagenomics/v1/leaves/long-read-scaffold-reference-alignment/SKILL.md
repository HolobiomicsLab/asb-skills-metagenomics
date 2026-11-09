---
name: long-read-scaffold-reference-alignment
description: Use when you must confirm whether a single scaffold from a hybrid metagenome
  assembly corresponds to a known reference genome and judge its structural completeness
  versus fragmented competitor assemblies.
when_to_use_negative:
- No complete reference genome exists for the target species — the study notes it
  focuses on de novo methods and 'reference information has not been well examined'
- Input is raw short- or long-read data rather than assembled scaffolds — this skill
  validates assembly structure, it does not assemble
- Evaluating short-read-only contig assemblies without long-range links, where near-complete
  single-species scaffolds are not expected
edam_operation: http://edamontology.org/operation_3194
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3697
tools:
- name: Minimap2
  role: Aligns extracted scaffolds/contigs to the B. longum 51A reference using the
    asm5 assembly-to-reference preset to assess structural correctness and edge error
    rates
  repo: https://github.com/lh3/minimap2
- name: MetaPlatanus
  role: Source of the candidate near-complete scaffold (scaffold2094) whose structure
    is being validated against the reference
  repo: https://github.com/rkajitani/MetaPlatanus
- name: CheckM
  role: Estimates genome completeness and contamination of the extracted scaffold
    sets per assembler
- name: metaFlye
  role: Competitor long-read assembler whose B. longum scaffolds are extracted for
    the fragmentation comparison
- name: MEGAHIT
  role: Competitor short-read assembler (also used by MetaPlatanus for initial contigs)
    included in the fragmentation comparison
  repo: https://github.com/voutcn/megahit
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:99f46363ed84a2d20b784bc312a620016e534e584e7634c53b225700117d1e4d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/rkajitani/MetaPlatanus
  repo_url: https://github.com/lh3/minimap2
  tools:
  - CheckM
  - MEGAHIT
  - MetaPlatanus
  - Minimap2
  - metaFlye
  edam_operation: http://edamontology.org/operation_3194
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# long-read-scaffold-reference-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that a metagenome assembler produced a near-complete, structurally correct scaffold for a target species by aligning the extracted scaffold (e.g. scaffold2094 of MetaPlatanus sample V06-T-0501-S07) to a complete reference genome (B. longum strain 51A) with Minimap2 and comparing fragmentation against competitor assemblies.

## When to use

Use when you must confirm whether a single scaffold from a hybrid metagenome assembly corresponds to a known reference genome and judge its structural completeness versus fragmented competitor assemblies.

## When NOT to use

- No complete reference genome exists for the target species — the study notes it focuses on de novo methods and 'reference information has not been well examined'
- Input is raw short- or long-read data rather than assembled scaffolds — this skill validates assembly structure, it does not assemble
- Evaluating short-read-only contig assemblies without long-range links, where near-complete single-species scaffolds are not expected

## Inputs

- MetaPlatanus assembly FASTA for human gut sample V06-T-0501-S07 (contains scaffold2094), from FigShare doi:10.6084/m9.figshare.15090603.v1
- Competitor assembly FASTA/contig sets (metaFlye, OPERA-MS, MEGAHIT) from the same FigShare deposit
- Complete reference genome FASTA of Bifidobacterium longum strain 51A

## Outputs

- Minimap2 PAF/SAM alignments of extracted scaffolds vs B. longum 51A reference
- Per-assembler minor 1k-mer edge rates computed within 1 kb of scaffold termini
- CheckM completeness and contamination estimates for each extracted scaffold set
- Comparison table blongum_51A_scaffold_comparison.tsv with scaffold counts, CheckM values, and edge error rates per assembler

## How to apply

Retrieve the assemblies from the FigShare deposit (doi:10.6084/m9.figshare.15090603.v1) and the complete B. longum strain 51A reference genome. Extract the candidate scaffold (scaffold2094) from the MetaPlatanus V06-T-0501-S07 assembly and the scaffolds/contigs covering the same genome from each competitor assembly (metaFlye, OPERA-MS, MEGAHIT). Align each extracted sequence set to the 51A reference with Minimap2 using the assembly-to-reference preset (asm5 / map-ont-style settings). Quantify residual error at scaffold termini by computing the minor 1k-mer rate within 1 kb of each scaffold edge, and run CheckM on the extracted sets for completeness and contamination. Compile scaffold counts, CheckM completeness/contamination, and edge minor 1k-mer rates into a TSV (blongum_51A_scaffold_comparison.tsv) with one row per assembler; the reproduction succeeds if only MetaPlatanus yields a single near-complete scaffold while competitors show more than seven sequences with interspecies misassembly signs at their edges (Figure 3 finding).

## Related tools

- **Minimap2** (Aligns extracted scaffolds/contigs to the B. longum 51A reference using the asm5 assembly-to-reference preset to assess structural correctness and edge error rates) — https://github.com/lh3/minimap2
- **MetaPlatanus** (Source of the candidate near-complete scaffold (scaffold2094) whose structure is being validated against the reference) — https://github.com/rkajitani/MetaPlatanus
- **CheckM** (Estimates genome completeness and contamination of the extracted scaffold sets per assembler)
- **metaFlye** (Competitor long-read assembler whose B. longum scaffolds are extracted for the fragmentation comparison)
- **MEGAHIT** (Competitor short-read assembler (also used by MetaPlatanus for initial contigs) included in the fragmentation comparison) — https://github.com/voutcn/megahit

## Examples

```
minimap2 -cx asm5 scaffold2094.fasta B_longum_51A.fasta > scaffold2094_vs_51A.paf
```

## Evaluation signals

- blongum_51A_scaffold_comparison.tsv exists in TSV format with one row for every assembler (MetaPlatanus, metaFlye, OPERA-MS, MEGAHIT)
- Only the MetaPlatanus scaffold2094 set aligns as a single near-complete scaffold consistent with the B. longum 51A genome structure
- Competitor assemblies covering B. longum 51A contain more than seven sequences (fragmented) per the Figure 3 finding
- Minor 1k-mer rates within 1 kb of MetaPlatanus scaffold edges are low, while competitor assemblies show interspecies misassembly signs at their edges
- CheckM reports high completeness and low contamination for the MetaPlatanus scaffold relative to competitor scaffold sets

## Limitations

- metaFlye requires additional short-read sequencing and polishing for rigorous downstream analysis
- Reference-based comparison is outside the study's de novo focus — reference information was not well examined, so this skill is a validation step, not the assembly method
- No tool versions were reported for compared assemblers (metaFlye, OPERA-MS, MEGAHIT), complicating exact reproduction
- Key quantitative evidence (Supplementary Tables S17, S24, S25) is deferred to supplementary materials

## Evidence

- [readme] ./minimap2 -cx asm5 asm1.fa asm2.fa > aln.paf             # intra-species asm-to-asm alignment: [evidence span withheld — anchor status: external_doc]
