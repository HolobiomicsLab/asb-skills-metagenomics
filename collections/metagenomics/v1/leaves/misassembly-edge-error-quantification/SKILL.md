---
name: misassembly-edge-error-quantification
description: Use when validating that a metagenomic assembler's scaffold corresponds
  to a specific reference genome (e.g. B. longum 51A scaffold2094 in gut sample V06-T-0501-S07)
  and you need to quantify edge misassembly and fragmentation per assembler.
when_to_use_negative:
- Input is raw sequencing reads rather than assembled scaffolds — use a read-mapping
  preset (map-ont) not the asm-to-asm workflow
- Reference-based evaluation is out of scope and only de novo assembly quality assessment
  is needed (the article notes reference information 'has not been well examined'
  in its own benchmarks)
- Aligning sequences between divergent species (>~15% divergence), where asm5 intra-species
  assumptions no longer hold per the Minimap2 README
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: Minimap2
  role: Aligns each extracted scaffold set to the B. longum 51A reference using the
    assembly-to-reference preset (asm5) so edge minor 1k-mer rates can be computed
  repo: https://github.com/lh3/minimap2
- name: CheckM
  role: Estimates genome completeness and contamination of the extracted scaffold
    sets for each assembler
- name: MetaPlatanus
  role: Source of the near-complete scaffold2094 assembly being validated against
    the reference
  repo: https://github.com/rkajitani/MetaPlatanus
- name: MEGAHIT
  role: Competitor short-read assembler whose fragmented contigs are included in the
    comparison
  repo: https://github.com/voutcn/megahit
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:dc47f564ee97728bb85ff77a5743a873c6b0fb76f4212f59795f6ab8fd70d4c6
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# misassembly-edge-error-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies residual assembly error at scaffold termini by aligning extracted scaffolds to a complete reference genome with Minimap2 (asm5 preset) and computing the minor 1k-mer rate within 1 kb of each scaffold edge, complemented by CheckM completeness/contamination estimates, to confirm whether an assembler produced a single near-complete, structurally correct genome scaffold versus fragmented, misjoined assemblies.

## When to use

Use when validating that a metagenomic assembler's scaffold corresponds to a specific reference genome (e.g. B. longum 51A scaffold2094 in gut sample V06-T-0501-S07) and you need to quantify edge misassembly and fragmentation per assembler.

## When NOT to use

- Input is raw sequencing reads rather than assembled scaffolds — use a read-mapping preset (map-ont) not the asm-to-asm workflow
- Reference-based evaluation is out of scope and only de novo assembly quality assessment is needed (the article notes reference information 'has not been well examined' in its own benchmarks)
- Aligning sequences between divergent species (>~15% divergence), where asm5 intra-species assumptions no longer hold per the Minimap2 README

## Inputs

- Metagenome assembly FASTA files per assembler from FigShare doi:10.6084/m9.figshare.15090603.v1 (MetaPlatanus, metaFlye, OPERA-MS, MEGAHIT)
- Complete reference genome FASTA (Bifidobacterium longum strain 51A)
- Extracted scaffold/contig FASTA subsets covering the reference species
- Minimap2 PAF/SAM alignments of scaffold sets to the reference

## Outputs

- Minimap2 alignment files (PAF/SAM) of each assembler's scaffold set vs the B. longum 51A reference
- Minor 1k-mer rate per scaffold computed within 1 kb of each edge
- CheckM completeness and contamination estimates per extracted scaffold set
- Comparison table blongum_51A_scaffold_comparison.tsv with scaffold counts, CheckM values, and edge error rates per assembler

## How to apply

Retrieve all assembler outputs (MetaPlatanus, metaFlye, OPERA-MS, MEGAHIT) from the FigShare deposit (doi:10.6084/m9.figshare.15090603.v1) plus the complete reference genome (B. longum strain 51A). Extract the target scaffold (e.g. scaffold2094) or the competitor scaffolds/contigs covering the reference. Align each extracted sequence set to the reference with Minimap2 using the assembly-to-reference preset (-x asm5/map-ont style settings), then compute the minor 1k-mer rate within 1 kb of each scaffold edge from the alignments to quantify residual error at scaffold termini. Run CheckM on each extracted scaffold set to obtain completeness and contamination. Compile scaffold counts, CheckM values, and edge minor 1k-mer rates into one comparison table (TSV); the reproduction succeeds only if the target assembler yields a single near-complete scaffold consistent with the reference while competitors show >7 sequences with interspecies misassembly signs at their edges.

## Related tools

- **Minimap2** (Aligns each extracted scaffold set to the B. longum 51A reference using the assembly-to-reference preset (asm5) so edge minor 1k-mer rates can be computed) — https://github.com/lh3/minimap2
- **CheckM** (Estimates genome completeness and contamination of the extracted scaffold sets for each assembler)
- **MetaPlatanus** (Source of the near-complete scaffold2094 assembly being validated against the reference) — https://github.com/rkajitani/MetaPlatanus
- **MEGAHIT** (Competitor short-read assembler whose fragmented contigs are included in the comparison) — https://github.com/voutcn/megahit

## Examples

```
minimap2 -cx asm5 B_longum_51A.fa scaffold2094.fa > scaffold2094_vs_51A.paf
```

## Evaluation signals

- Output file blongum_51A_scaffold_comparison.tsv exists and is valid TSV with one row per assembler
- MetaPlatanus scaffold2094 aligns end-to-end to the B. longum 51A reference as a single near-complete scaffold consistent with the complete genome
- Competitor assemblies cover the B. longum genome with more than seven fragmented sequences showing misassembly signs at their edges
- Minor 1k-mer rates within 1 kb of scaffold edges are computed for every assembler and are near zero for the MetaPlatanus scaffold
- CheckM reports high completeness with low contamination for the MetaPlatanus scaffold set

## Limitations

- metaFlye requires additional short-read sequencing and polishing for rigorous downstream analysis
- Competitor assemblers show interspecies misassemblies at scaffold edges, so edge-based error rates can be inflated by false interspecies links
- No tool versions were reported for the compared assemblers (metaFlye, OPERA-MS, MEGAHIT), limiting exact reproducibility

## Evidence

- [readme] Minimap2 asm-to-asm alignment preset: [evidence span withheld — anchor status: external_doc]
