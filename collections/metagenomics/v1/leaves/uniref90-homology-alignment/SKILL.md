---
name: uniref90-homology-alignment
description: Use when constructing gene-level functional abundance profiles from shotgun
  metagenomic FASTQ within the OMARU workflow, or when reproducing assembly-based
  gene annotation with UniRef90 via DIAMOND.
when_to_use_negative:
- Input is already a gene/pathway abundance table or the goal is only phylogenetic
  profiling (use Prepare_reference_UHGG.sh and OMARU_Phyl_AS.sm instead)
- Shotgun reads have not passed Step 1 read QC (OMARU_read_QC.sm) and samples have
  not been filtered into QCed1_sample_list.txt
- Disk space or runtime budget cannot accommodate the UniRef90 preparation (~200 GB,
  several hours) or a smaller functional reference is preferred
edam_operation: http://edamontology.org/operation_3663
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0623
- http://edamontology.org/topic_0091
tools:
- name: OMARU
  role: End-to-end workflow that wraps assembly, ORF prediction, DIAMOND/UniRef90
    alignment, and gene abundance quantification
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine executing the profiling stage via Snakefiles/OMARU_QCed1.sm
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Conda
  role: Installation and environment management for OMARU (conda-pack distribution
    from Zenodo)
  repo: https://github.com/toshi-kishikawa/OMARU
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:da4ebe5c2e69f74198bc2c14cafc578056b05ba9025583a6abac05593789e69f
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
  edam_operation: http://edamontology.org/operation_3663
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0623
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# uniref90-homology-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

OMARU's assembly-based functional profiling step in which open reading frames predicted on de novo MEGAHIT-assembled contigs are assigned gene identities by DIAMOND alignment against the UniRef90 reference, followed by gene abundance quantification.

## When to use

Use when constructing gene-level functional abundance profiles from shotgun metagenomic FASTQ within the OMARU workflow, or when reproducing assembly-based gene annotation with UniRef90 via DIAMOND.

## When NOT to use

- Input is already a gene/pathway abundance table or the goal is only phylogenetic profiling (use Prepare_reference_UHGG.sh and OMARU_Phyl_AS.sm instead)
- Shotgun reads have not passed Step 1 read QC (OMARU_read_QC.sm) and samples have not been filtered into QCed1_sample_list.txt
- Disk space or runtime budget cannot accommodate the UniRef90 preparation (~200 GB, several hours) or a smaller functional reference is preferred

## Inputs

- QCed shotgun metagenomic FASTQ files (<Sample_ID>_R1.fastq.gz / <Sample_ID>_R2.fastq.gz) in <OMARU_project_dir>/result/QC/QCed_fastq
- Prepared UniRef90 functional reference database with gene and pathway annotation files in OMARU_databases
- QCed1_sample_list.txt of samples passing read QC
- config.yaml parameters (e.g. REF: GENE = UniRef90, REF: PATH = GO, THREAD)

## Outputs

- Gene abundance matrix (length- and depth-normalized, rank-based inverse normal transformed)
- Profiling summary tables and figures in <OMARU_project_dir>/result/<Phenotype>_summary
- QCed2_sample_list.txt of samples passing profiling QC for downstream abundance checks

## How to apply

First prepare the functional reference by running Prepare_reference_UniRef90.sh against the OMARU_databases directory, which downloads and formats UniRef90 (with GO term pathway annotation); note this takes several hours and ~200 GB of disk space. Then execute the profiling stage of the OMARU Snakemake workflow (Snakefiles/OMARU_QCed1.sm) on QCed FASTQ files in <OMARU_project_dir>/result/QC/QCed_fastq, assembling reads de novo per sample with MEGAHIT. Predict ORFs from the assembled contigs with Prokka, then align the predicted ORFs against UniRef90 with DIAMOND to assign gene identities. Quantify gene abundance from ORF alignment depth, normalizing by gene length and each sample's total read count, and finally apply a rank-based inverse normal transformation to the abundance values. The gene-level QC downstream retained 185,663 genes for SCZ and 104,487 genes for T2D case-control comparisons, providing expected magnitudes for validation.

## Related tools

- **OMARU** (End-to-end workflow that wraps assembly, ORF prediction, DIAMOND/UniRef90 alignment, and gene abundance quantification) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine executing the profiling stage via Snakefiles/OMARU_QCed1.sm) — https://github.com/toshi-kishikawa/OMARU
- **Conda** (Installation and environment management for OMARU (conda-pack distribution from Zenodo)) — https://github.com/toshi-kishikawa/OMARU

## Examples

```
cd OMARU_project_dir && snakemake -s Snakefiles/OMARU_QCed1.sm
```

## Evaluation signals

- Confirm the prepared UniRef90 database and its gene/pathway annotation files exist in OMARU_databases before profiling
- Verify the output gene abundance matrix exists and summary tables/figures are written to <OMARU_project_dir>/result/<Phenotype>_summary
- Check gene-level QC magnitudes against the reported counts: 185,663 genes retained for SCZ and 104,487 genes for T2D
- Confirm MEGAHIT assembly and Prokka ORF prediction logs completed per sample without failures in the Snakemake run of OMARU_QCed1.sm
- Verify abundance values are normalized by gene length and total read count and rank-based inverse normal transformed before association testing

## Limitations

- Preparing the UniRef90 functional reference is time and resource-intensive, taking several hours and almost 200 GB of free disk space
- Metagenome analysis remains highly dependent on reference databases; database development is a challenge for the future
- Only samples passing prior read QC and profiling QC (QCed1/QCed2 sample lists) should enter this stage; low-quality samples must be excluded upstream

## Evidence
