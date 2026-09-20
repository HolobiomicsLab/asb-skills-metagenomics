---
name: bin-completeness-and-strain-heterogeneity-assessment
description: Use when you have a set of metagenomic bins clustered from assembled
  contigs and need per-bin completeness and strain heterogeneity estimates to judge
  which bins represent near-complete organisms.
when_to_use_negative:
- Input is raw reads and no assembly/binning has been performed yet — the skill requires
  pre-existing contigs and MetaBAT2 bins
- Input is isolate (single-organism) genome data with no strain mixture — strain heterogeneity
  assessment is designed for metagenomic, intra-species-diverse communities
- You need functional annotation of bins rather than completeness/redundancy quality
  estimates
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: CheckM
  role: Assesses each metagenomic bin for completeness and strain heterogeneity, producing
    the bin-quality metrics
- name: MetaBAT2
  role: Clusters the filtered contigs into metagenomic bins that are then assessed
    by CheckM
- name: Bowtie2
  role: Maps trimmed reads back to the filtered contigs to provide coverage information
    for binning
- name: MetaCortex
  role: Upstream assembler whose sequence-graph contigs feed the binning and assessment
    pipeline
  repo: https://github.com/SR-Martin/metacortex
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:a0e5bb5a0b9fc931c049bab5b088f88de9353a23dc428689595b7ae4fb77fc7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex
  tools:
  - Bowtie2
  - CheckM
  - Conda
  - GCC
  - MetaBAT2
  - MetaCortex
  - SRA toolkit
  - trim-galore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# bin-completeness-and-strain-heterogeneity-assessment

## Summary

Assess the quality of metagenomic bins generated from a MetaCortex SW assembly by evaluating each bin with CheckM for completeness and strain heterogeneity, compiling the results into a bin-quality table (e.g. 155 bins, 23 bins >50% complete, 11 bins with >50% strain heterogeneity).

## When to use

Use when you have a set of metagenomic bins clustered from assembled contigs and need per-bin completeness and strain heterogeneity estimates to judge which bins represent near-complete organisms.

## When NOT to use

- Input is raw reads and no assembly/binning has been performed yet — the skill requires pre-existing contigs and MetaBAT2 bins
- Input is isolate (single-organism) genome data with no strain mixture — strain heterogeneity assessment is designed for metagenomic, intra-species-diverse communities
- You need functional annotation of bins rather than completeness/redundancy quality estimates

## Inputs

- Trimmed FASTQ reads from a metagenomic WGS sample (e.g. SRA run SRR13060942 / ENA ERR2099157)
- Assembly contigs of length >=500 bp from the MetaCortex SW output
- Bowtie2 read-mapping files over the filtered contigs (for MetaBAT2 binning)
- MetaBAT2 metagenomic bins (155 bins in the reference run)

## Outputs

- Per-bin CheckM completeness and strain heterogeneity estimates
- Bin-quality table (e.g. 155 bins, 23 bins >50% complete, 11 bins with >50% strain heterogeneity)
- Assembly summary table (123720 contigs >=500 bp, N50 5399, max contig 879838 bp)

## How to apply

After assembling trimmed reads with the MetaCortex sequence-graph workflow (k=63, min coverage 2, delta 0.8) and filtering contigs shorter than 500 bp (target: 123720 contigs, N50 5399, max contig 879838 bp), map the trimmed reads back to the filtered contigs with Bowtie2 and cluster contigs into metagenomic bins with MetaBAT2. Then run CheckM on every bin to estimate completeness and strain heterogeneity, which is the article's measure of intra-species diversity captured in the assembly graphs. Bin quality is judged with explicit thresholds: count bins above 50% completeness and bins above 50% strain heterogeneity (article targets: 155 bins, 23 bins >50% complete, 11 bins with >50% strain heterogeneity). Compile the assembly summary and bin-quality table into tabular output files for downstream comparison against target statistics.

## Related tools

- **CheckM** (Assesses each metagenomic bin for completeness and strain heterogeneity, producing the bin-quality metrics)
- **MetaBAT2** (Clusters the filtered contigs into metagenomic bins that are then assessed by CheckM)
- **Bowtie2** (Maps trimmed reads back to the filtered contigs to provide coverage information for binning)
- **MetaCortex** (Upstream assembler whose sequence-graph contigs feed the binning and assessment pipeline) — https://github.com/SR-Martin/metacortex

## Evaluation signals

- Assembly summary matches target: 123720 contigs >=500 bp, N50 5399, maximum contig length 879838 bp
- Bin count matches target: 155 bins produced by MetaBAT2
- Quality thresholds reproduced: 23 bins with >50% completeness and 11 bins with >50% strain heterogeneity
- CheckM outputs completeness and strain heterogeneity for every bin with no missing/failed bins
- Assembly and bin-quality metrics are exported as tabular output files as specified in the workflow

## Limitations

- LLVM will not compile MetaCortex, so building from source requires GCC (on Mac, set CC to point to the GCC binary)
- Results depend on upstream choices (k=63, min coverage 2, SW delta 0.8, 500 bp contig filter); changing any of these shifts the bin-quality statistics away from the reported targets
- MetaCortex 2.0 targets metagenomic data with strain-level diversity; strain heterogeneity estimates are only meaningful for such communities

## Evidence

- [full_text] Map the trimmed reads back to the filtered contigs with Bowtie2 and cluster contigs into metagenomic bins with MetaBAT2.: [evidence span withheld — anchor status: not_found]
- [full_text] CheckM completeness and strain heterogeneity targets: [evidence span withheld — anchor status: not_found]
- [abstract] MetaCortex captures intra-species diversity via assembly graph variation signatures: "MetaCortex, a metagenome assembler that captures intra-species diversity by searching for signatures of local variation along assembled sequences"
- [full_text] Discard contigs <500 bp from the assembly output: [evidence span withheld — anchor status: not_found]
