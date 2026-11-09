---
name: linked-read-megabubble-haplotype-extraction
description: Use when you have Supernova2 assemblies from 10x Linked-Read data and
  need to quantify the diploid fraction (windows covered by both homologous contigs)
  against a reference genome.
when_to_use_negative:
- Input data are simulated Linked-Reads evaluated only by contiguity/accuracy metrics
  against a simulated gold standard (LRTK-SIM workflow), not real assemblies to be
  assessed for diploid coverage
- Assemblies were produced by assemblers other than Supernova2, whose phase-block/megabubble
  output structure mkoutput depends on
- The goal is long-read (PacBio/ONT) assembly evaluation, where haplotype-resolved
  contigs are not organized into 10x megabubble phase blocks
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Supernova2
  role: De novo assembler of the 10x Linked-Read data whose mkoutput step exports
    the megabubble haplotype scaffold pairs from phase blocks for diploid-fraction
    evaluation
- name: hg38 human reference sequence
  role: Reference to which haplotype scaffolds are aligned to assign genomic coordinates
    for 500-bp window classification
- name: Evaluate_diploid_assembly
  role: Repository with the evaluation notebook (Evaluate_assembly_performance.ipynb)
    describing how to evaluate and compare diploid assemblies
  repo: https://github.com/zhanglu295/Evaluate_diploid_assembly
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:12829d930d44d3dac5f09755a0bf74bc399b026f2e252ce112ae7ccd338cde2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhanglu295/Evaluate_diploid_assembly
  tools:
  - 10x Linked-Read sequencing
  - Evaluate_diploid_assembly
  - Supernova2
  - hg38 human reference sequence
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# linked-read-megabubble-haplotype-extraction

## Summary

Extracts megabubble haplotype scaffold pairs from Supernova2 phase blocks (via Supernova2 mkoutput) and aligns them to hg38 to quantify what fraction of the human genome is recovered in a diploid state. This is the core evaluation method of the 10x Linked-Read diploid assembly assessment workflow.

## When to use

Use when you have Supernova2 assemblies from 10x Linked-Read data and need to quantify the diploid fraction (windows covered by both homologous contigs) against a reference genome.

## When NOT to use

- Input data are simulated Linked-Reads evaluated only by contiguity/accuracy metrics against a simulated gold standard (LRTK-SIM workflow), not real assemblies to be assessed for diploid coverage
- Assemblies were produced by assemblers other than Supernova2, whose phase-block/megabubble output structure mkoutput depends on
- The goal is long-read (PacBio/ONT) assembly evaluation, where haplotype-resolved contigs are not organized into 10x megabubble phase blocks

## Inputs

- Supernova2 de novo assemblies of 10x Linked-Read data (e.g. from NA12878 and NA24385 libraries)
- hg38 human reference sequence
- Haplotype scaffolds exported from phase blocks via Supernova2 mkoutput

## Outputs

- Per-window classification of the genome as diploid, hemizygous, or uncovered per assembly
- Per-assembly diploid fraction and overall genome coverage statistics
- Cross-assembly diploid overlap fractions for same-sample replicate assemblies
- diploid_assembly_stats.tsv

## How to apply

Run Supernova2 mkoutput on each assembly to export the megabubble haplotype scaffold pairs from phase blocks. Align the extracted haplotype scaffolds to the hg38 human reference to obtain genomic coordinates. Slide non-overlapping 500-bp windows across the genome and classify each window as diploid (covered by both haplotypes), hemizygous (single haplotype), or uncovered based on contig alignment coverage. Compute overall genome coverage and per-assembly diploid fraction; expect ~91% overall coverage and diploid fractions of 70-80% for broadly optimal libraries (e.g. 58.9% for the R6 library). As a secondary check, verify hemizygous X regions are spanned by a single contig in 79.9-87.6% of windows, and intersect diploid-region coordinates across replicate assemblies from the same cell line to compute cross-assembly overlap (50.24% for NA12878, 67.27% for NA24385).

## Related tools

- **Supernova2** (De novo assembler of the 10x Linked-Read data whose mkoutput step exports the megabubble haplotype scaffold pairs from phase blocks for diploid-fraction evaluation)
- **hg38 human reference sequence** (Reference to which haplotype scaffolds are aligned to assign genomic coordinates for 500-bp window classification)
- **Evaluate_diploid_assembly** (Repository with the evaluation notebook (Evaluate_assembly_performance.ipynb) describing how to evaluate and compare diploid assemblies) — https://github.com/zhanglu295/Evaluate_diploid_assembly

## Examples

```
supernova mkoutput --asm=asm --style=haplotypes --out=hap_scaffolds
```

## Evaluation signals

- Overall genome coverage of the real-data assemblies reaches ~91%
- Diploid fraction falls in the 70-80% band for broadly optimal libraries (R6 library expected at 58.9%)
- Hemizygous X regions are spanned by a single contig in 79.9-87.6% of windows
- Cross-assembly diploid overlap fractions reproduce 50.24% (NA12878) and 67.27% (NA24385) within tolerance
- Output table diploid_assembly_stats.tsv is produced with per-assembly diploid fractions and coverage values

## Limitations

- One library (R6) falls below the optimal 70-80% diploid band at 58.9%, so library parameters strongly affect the result
- Cross-assembly diploid overlap is far from complete (50.24% for NA12878, 67.27% for NA24385), so diploid-region identity is assembly-dependent
- The fragment-length effect on assembly quality could not be reasoned about because no alternative assembler comparable to Supernova2 was available

## Evidence

- [abstract] for most assemblies 70%–80% of the genome was covered by 2 homologous contigs, defining the recovered diploid fraction: [evidence span withheld — anchor status: not_found]
- [methods] Run Supernova2 mkoutput to export megabubble haplotype scaffolds: [evidence span withheld — anchor status: not_found]
- [methods] 500-bp windows classified as diploid, hemizygous, or uncovered: [evidence span withheld — anchor status: not_found]
- [discussion] Codes for diploid assembly evaluation are publicly available: "the codes for comparison are currently available at http://mendel.stanford.edu/supplementarydata/zhang_SN2_2019 and https://github.com/zhanglu295/Evaluate_diploid_assembly"
- [readme] README points to the evaluation notebook: [evidence span withheld — anchor status: external_doc]
