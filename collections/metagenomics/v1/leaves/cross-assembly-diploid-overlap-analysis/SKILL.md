---
name: cross-assembly-diploid-overlap-analysis
description: Use when multiple Supernova2 assemblies exist for the same sample and
  you need per-assembly diploid/hemizygous/uncovered fractions plus cross-assembly
  diploid overlap percentages.
when_to_use_negative:
- Input is simulated Linked-Read data rather than real assemblies evaluated against
  hg38 (use the LRTK-SIM gold-standard path instead)
- Only a single assembly is available, so cross-assembly diploid overlap cannot be
  computed
- Assemblies are not from Supernova2 phase blocks (e.g. PacBio/ONT haplotig-style
  assemblies), since megabubble haplotype scaffold extraction is Supernova2-specific
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Supernova2
  role: Assembler whose phase blocks are exported via mkoutput as megabubble haplotype
    scaffold pairs for window-based coverage classification
- name: Evaluate_diploid_assembly
  role: Jupyter notebook (Evaluate_assembly_performance.ipynb) implementing the diploid
    assembly evaluation and comparison workflow
  repo: https://github.com/zhanglu295/Evaluate_diploid_assembly
- name: hg38 human reference sequence
  role: Reference to which haplotype scaffolds are aligned to assign genomic coordinates
    for 500-bp window classification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:2e900e319eebad06415bb9ca50a3401ae970b33218c3569156f6b3210cf4e879
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhanglu295/Evaluate_diploid_assembly
  tools:
  - Evaluate_diploid_assembly
  - Supernova2
  - hg38 human reference sequence
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-assembly-diploid-overlap-analysis

## Summary

Quantifies how much of the human genome is recovered in a diploid state by 10x Linked-Read Supernova2 assemblies and measures diploid-region overlap across independently generated assemblies of the same cell line (NA12878, NA24385).

## When to use

Use when multiple Supernova2 assemblies exist for the same sample and you need per-assembly diploid/hemizygous/uncovered fractions plus cross-assembly diploid overlap percentages.

## When NOT to use

- Input is simulated Linked-Read data rather than real assemblies evaluated against hg38 (use the LRTK-SIM gold-standard path instead)
- Only a single assembly is available, so cross-assembly diploid overlap cannot be computed
- Assemblies are not from Supernova2 phase blocks (e.g. PacBio/ONT haplotig-style assemblies), since megabubble haplotype scaffold extraction is Supernova2-specific

## Inputs

- Supernova2 assemblies from six real 10x Linked-Read libraries (three NA12878, three NA24385)
- hg38 human reference sequence
- Supernova2 mkoutput-exported megabubble haplotype scaffold pairs from phase blocks

## Outputs

- Per-window 500-bp classifications (diploid / hemizygous / uncovered) per assembly
- Overall genome coverage and per-assembly diploid fraction
- Cross-assembly diploid overlap fractions per cell line
- diploid_assembly_stats.tsv

## How to apply

Run Supernova2 mkoutput on each real-data assembly to export megabubble haplotype scaffold pairs from phase blocks, then align the extracted haplotype scaffolds to the hg38 human reference to obtain genomic coordinates. Slide non-overlapping 500-bp windows across the genome and classify each window per assembly as diploid (covered by both homologous contigs), hemizygous, or uncovered. Compute overall genome coverage (~91% expected) and per-assembly diploid fraction (70-80% for optimal libraries, 58.9% for R6); verify hemizygous X regions are spanned by a single contig in 79.9-87.6% of windows. Finally, intersect diploid-region coordinates across the three NA12878 and three NA24385 assemblies separately to obtain cross-assembly overlap fractions of 50.24% (NA12878) and 67.27% (NA24385), and write results to diploid_assembly_stats.tsv. The evaluation code is provided in the Evaluate_diploid_assembly repository's Evaluate_assembly_performance.ipynb notebook.

## Related tools

- **Supernova2** (Assembler whose phase blocks are exported via mkoutput as megabubble haplotype scaffold pairs for window-based coverage classification)
- **Evaluate_diploid_assembly** (Jupyter notebook (Evaluate_assembly_performance.ipynb) implementing the diploid assembly evaluation and comparison workflow) — https://github.com/zhanglu295/Evaluate_diploid_assembly
- **hg38 human reference sequence** (Reference to which haplotype scaffolds are aligned to assign genomic coordinates for 500-bp window classification)

## Examples

```
jupyter notebook Evaluate_assembly_performance.ipynb
```

## Evaluation signals

- Overall genome coverage across the six real-data assemblies reaches ~91%
- Per-assembly diploid fraction falls in the 70-80% band for optimal libraries, with the R6 library at 58.9%
- Hemizygous X regions are spanned by a single contig in 79.9-87.6% of windows
- Cross-assembly diploid overlap reproduces 50.24% for NA12878 and 67.27% for NA24385 within tolerance
- Output file diploid_assembly_stats.tsv is written with all per-assembly and overlap statistics

## Limitations

- NA12878 and NA24385 cross-assembly overlap fractions differ substantially (50.24% vs 67.27%), so overlap is library-dependent
- Only ~91% of the genome is covered at all; uncovered and hemizygous regions are excluded from diploid fraction
- No alternative assembler to Supernova2 was available for comparison, limiting generalization of the diploid-recovery conclusions

## Evidence

- [abstract] When broadly optimal parameters were used for library preparation and sequencing, ∼80% of the genome was assembled in a diploid state.: "When broadly optimal parameters were used for library preparation and sequencing, ∼80% of the genome was assembled in a diploid state."
- [discussion] Evaluation code repository citation: "the codes for comparison are currently available at http://mendel.stanford.edu/supplementarydata/zhang_SN2_2019 and https://github.com/zhanglu295/Evaluate_diploid_assembly"
- [readme] README purpose statement: [evidence span withheld — anchor status: external_doc]
