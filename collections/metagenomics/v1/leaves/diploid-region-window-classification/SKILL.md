---
name: diploid-region-window-classification
description: Use when you need to measure what fraction of the genome a 10x Linked-Read
  de novo assembly (Supernova2) recovers in a diploid (both-homologs-covered) state,
  rather than a haploid contiguity metric such as N50.
when_to_use_negative:
- When the goal is haploid contiguity or accuracy assessment (e.g. N50, NA50, GIAB
  variant comparison) rather than diploid-state recovery.
- When the assembly was not produced with Supernova2 phase blocks / megabubble haplotypes
  — the classification depends on per-haplotype scaffold pairs from mkoutput.
- When no reference genome with good coordinates exists to which haplotype scaffolds
  can be aligned (window coordinates are defined on hg38).
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Supernova2
  role: Assembler whose mkoutput module exports the megabubble haplotype scaffold
    pairs from phase blocks that are then classified per 500-bp window
- name: Evaluate_diploid_assembly
  role: Reference implementation of the diploid-assembly evaluation and comparison
    code (see Evaluate_assembly_performance.ipynb)
  repo: https://github.com/zhanglu295/Evaluate_diploid_assembly
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:4737cc1ad124a3c542d26ca635c2a9d8841115ecb677c3ab68fb2cb783c2f68c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhanglu295/Evaluate_diploid_assembly
  tools:
  - Evaluate_diploid_assembly
  - Supernova2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# diploid-region-window-classification

## Summary

A window-based classification method that quantifies the diploid fraction of Supernova2 assemblies built from 10x Linked-Read data by sliding 500-bp windows over the hg38 reference and labeling each window as diploid, hemizygous, or uncovered based on haplotype-scaffold alignment coverage.

## When to use

Use when you need to measure what fraction of the genome a 10x Linked-Read de novo assembly (Supernova2) recovers in a diploid (both-homologs-covered) state, rather than a haploid contiguity metric such as N50.

## When NOT to use

- When the goal is haploid contiguity or accuracy assessment (e.g. N50, NA50, GIAB variant comparison) rather than diploid-state recovery.
- When the assembly was not produced with Supernova2 phase blocks / megabubble haplotypes — the classification depends on per-haplotype scaffold pairs from mkoutput.
- When no reference genome with good coordinates exists to which haplotype scaffolds can be aligned (window coordinates are defined on hg38).

## Inputs

- Supernova2 megabubble haplotype scaffold pairs exported via mkoutput from phase blocks (FASTA per assembly)
- hg38 human reference sequence (FASTA)
- Scaffold-to-reference alignments with genomic coordinates for each of the six real-data assemblies (NA12878 and NA24385)

## Outputs

- Per-window classification table (500-bp windows labeled diploid / hemizygous / uncovered per assembly)
- Per-assembly diploid fraction and overall genome coverage statistics
- Cross-assembly diploid overlap fractions per sample group (NA12878, NA24385)
- diploid_assembly_stats.tsv

## How to apply

First export the megabubble haplotype scaffold pairs from each Supernova2 assembly's phase blocks using `Supernova2 mkoutput`, then align the extracted haplotype scaffolds to the hg38 human reference sequence to obtain genomic coordinates. Slide non-overlapping 500-bp windows across the genome and classify each window as diploid (covered by both haplotypes), hemizygous, or uncovered according to contig alignment coverage. Compute overall genome coverage (~91% expected for the real-data assemblies) and per-assembly diploid fraction, expecting 70%–80% for optimally prepared libraries (~58.9% for the R6 library). Separately verify that hemizygous X regions are spanned by a single contig in 79.9%–87.6% of windows, and intersect diploid-region coordinates across the three NA12878 and three NA24385 assemblies to compute cross-assembly diploid overlaps (50.24% for NA12878, 67.27% for NA24385) before writing diploid_assembly_stats.tsv. The article's evaluation code is available in the Evaluate_diploid_assembly repository, with details in Evaluate_assembly_performance.ipynb.

## Related tools

- **Supernova2** (Assembler whose mkoutput module exports the megabubble haplotype scaffold pairs from phase blocks that are then classified per 500-bp window)
- **Evaluate_diploid_assembly** (Reference implementation of the diploid-assembly evaluation and comparison code (see Evaluate_assembly_performance.ipynb)) — https://github.com/zhanglu295/Evaluate_diploid_assembly

## Evaluation signals

- Overall genome coverage of the six real-data assemblies should be approximately 91%.
- Diploid fraction for most assemblies should fall in the 70%–80% band, with the R6 library at ~58.9%.
- Hemizygous X regions should be spanned by a single contig in 79.9%–87.6% of windows.
- Cross-assembly diploid overlap should reproduce 50.24% for NA12878 and 67.27% for NA24385 within tolerance.
- Windows must be non-overlapping 500-bp bins and every window must be assigned exactly one of the three states (diploid, hemizygous, uncovered).

## Limitations

- The diploid fraction is sensitive to library preparation parameters; only ~80% of the genome is assembled in a diploid state under broadly optimal parameters (CF 335×–823×, no larger than 1,000×, WμFL ~50–150 kb).
- Assembly quality varies with sequencing and physical coverage, so fractions below the 70%–80% band may reflect suboptimal libraries (e.g. R6 at 58.9%) rather than a pipeline error.
- NA50-based quality assessment and this diploid fraction rely on the existence of a high-quality human reference genome; they are not generalizable to non-model organisms.

## Evidence

- [other] Overall genome coverage of the 6 real-data assemblies was ∼91%, and for most assemblies 70%–80% of the genome was covered by 2 homologous contigs: [evidence span withheld — anchor status: not_found]
- [abstract] When broadly optimal parameters were used for library preparation and sequencing, ~80% of the genome was assembled in a diploid state.: "When broadly optimal parameters were used for library preparation and sequencing, ∼80% of the genome was assembled in a diploid state."
- [readme] This is a description of how to evaluate and compare diploid assemblies of different parameters.: [evidence span withheld — anchor status: external_doc]
- [discussion] the codes for comparison are currently available at http://mendel.stanford.edu/supplementarydata/zhang_SN2_2019 and https://github.com/zhanglu295/Evaluate_diploid_assembly: "the codes for comparison are currently available at http://mendel.stanford.edu/supplementarydata/zhang_SN2_2019 and https://github.com/zhanglu295/Evaluate_diploid_assembly"
