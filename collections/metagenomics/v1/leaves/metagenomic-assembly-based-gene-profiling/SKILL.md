---
name: metagenomic-assembly-based-gene-profiling
description: Use when building gene-level functional abundance profiles from shotgun
  metagenomic FASTQ reads via de novo assembly and UniRef90 gene annotation, e.g.
  as input to gene-level MWAS case-control tests.
when_to_use_negative:
- Input reads have not passed read QC (Step 1, OMARU_read_QC.sm) — the workflow expects
  QCed FASTQ in result/QC/QCed_fastq
- The goal is phylogenetic (taxon-level) profiling from a clade reference such as
  UHGG/CHOCO rather than gene-level functional profiling
- Only amplicon (16S) data are available — the pipeline is designed for shotgun sequencing
  reads
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: OMARU
  role: End-to-end workflow that packages the assembly-based gene profiling stage
    (MEGAHIT -> Prokka -> DIAMOND/UniRef90 -> quantification)
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine executing the profiling stage via Snakefiles/OMARU_QCed1.sm
    and downstream gene QC (OMARU_QCed2.sm)
- name: Conda
  role: Installation/distribution mechanism for OMARU (conda-pack, version 4.8 or
    greater)
  repo: https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:766452ab6e307fd70dba876da41b7606ba13ec4d8a3d9c98a88ac2d9762c7f2a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/toshi-kishikawa/OMARU
  repo_url: https://github.com/toshi-kishikawa/OMARU
  tools:
  - Conda
  - OMARU
  - Snakemake
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# metagenomic-assembly-based-gene-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs OMARU's assembly-based functional (gene-level) profiling, in which shotgun reads are de novo assembled per sample with MEGAHIT, ORFs are predicted with Prokka, assigned gene identities via DIAMOND alignment against UniRef90, and gene abundances are quantified, length-normalized, and rank-based inverse normal transformed.

## When to use

Use when building gene-level functional abundance profiles from shotgun metagenomic FASTQ reads via de novo assembly and UniRef90 gene annotation, e.g. as input to gene-level MWAS case-control tests.

## When NOT to use

- Input reads have not passed read QC (Step 1, OMARU_read_QC.sm) — the workflow expects QCed FASTQ in result/QC/QCed_fastq
- The goal is phylogenetic (taxon-level) profiling from a clade reference such as UHGG/CHOCO rather than gene-level functional profiling
- Only amplicon (16S) data are available — the pipeline is designed for shotgun sequencing reads

## Inputs

- QCed shotgun metagenomic FASTQ files (<Sample_ID>_R1.fastq.gz / _R2.fastq.gz in <OMARU_project_dir>/result/QC/QCed_fastq)
- QCed1_sample_list.txt (samples passing read QC)
- UniRef90 functional reference database with gene/pathway annotation files in <OMARU_dir>/OMARU_databases
- <OMARU_project_dir>/config.yaml (REF: GENE = UniRef90, REF: PATH = GO, THREAD, etc.)

## Outputs

- Gene abundance matrix (ORF alignment depth normalized by gene length and total read count, rank-based inverse normal transformed)
- Assembled contigs (MEGAHIT) and predicted ORFs (Prokka) per sample
- Profiling summary tables and figures at <OMARU_project_dir>/result/<Phenotype>_summary
- Abundance QC tables/figures at <OMARU_project_dir>/result/PHYL_QCed2/<Gene_reference>_graph_basic (via OMARU_QCed2.sm)

## How to apply

First prepare the functional reference by running Prepare_reference_UniRef90.sh against the OMARU_databases directory (this can take several hours and ~200 GB of disk). Then execute the profiling stage of the OMARU Snakemake workflow (Snakefiles/OMARU_QCed1.sm) on QCed FASTQ files placed in <OMARU_project_dir>/result/QC/QCed_fastq, assembling reads de novo per sample with MEGAHIT. Predict open reading frames from assembled contigs with Prokka and align them against UniRef90 with DIAMOND to assign gene identities. Quantify gene abundance from ORF alignment depth, normalizing by gene length and each sample's total read count, then apply a rank-based inverse normal transformation to the abundance values. Select samples with sufficient profiling quality and update the sample list to QCed2_sample_list.txt before downstream gene association tests (Snakefiles/OMARU_Func_AS.sm).

## Related tools

- **OMARU** (End-to-end workflow that packages the assembly-based gene profiling stage (MEGAHIT -> Prokka -> DIAMOND/UniRef90 -> quantification)) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine executing the profiling stage via Snakefiles/OMARU_QCed1.sm and downstream gene QC (OMARU_QCed2.sm))
- **Conda** (Installation/distribution mechanism for OMARU (conda-pack, version 4.8 or greater)) — https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html

## Examples

```
cd OMARU_project_dir && snakemake -s Snakefiles/OMARU_QCed1.sm
```

## Evaluation signals

- Output gene abundance matrix exists and is populated after OMARU_QCed1.sm completes
- Gene-level QC retains the reported counts: 185,663 genes for SCZ and 104,487 genes for T2D case-control comparison
- Profiling summary tables and figures are written to <OMARU_project_dir>/result/<Phenotype>_summary
- QCed FASTQ inputs are present in <OMARU_project_dir>/result/QC/QCed_fastq before running the profiling stage
- Abundance QC graphs appear under result/PHYL_QCed2/<Gene_reference>_graph_basic after OMARU_QCed2.sm

## Limitations

- Reference database preparation (Prepare_reference_UniRef90.sh) is time and resource-intensive: several hours and almost 200 GB of free disk space
- Metagenome analysis remains highly dependent on reference databases; database development is a stated future challenge
- Hidden confounding factors cannot currently be adjusted by integrating covariates into the case-control model — not implemented in OMARU
- Disk space for reference databases: ~200 GB free required

## Evidence

- [other] open reading frames (ORFs) are predicted on de novo assembled contigs with Prokka and aligned against UniRef90 using DIAMOND: [evidence span withheld — anchor status: not_found]
