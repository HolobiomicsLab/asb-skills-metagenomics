---
name: orf-prediction-from-contigs
description: 'Use when you have per-sample QCed shotgun FASTQ files and need gene-level
  functional profiling: contigs are assembled de novo (MEGAHIT) and ORFs must be called
  on them before UniRef90 annotation.'
when_to_use_negative:
- Input is a pre-computed gene abundance/feature table rather than raw or QCed FASTQ
  reads — this skill only covers the read-to-contig-to-ORF construction stage
- You only need phylogenetic (clade-level) profiling, which OMARU runs against UHGG
  via Prepare_reference_UHGG.sh instead of the UniRef90 functional path
- Reads are amplicon/16S rather than shotgun metagenomic, since OMARU's workflow is
  designed for shotgun sequencing reads
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0085
tools:
- name: Prokka
  role: Predicts open reading frames from de novo assembled contigs prior to gene
    annotation
- name: MEGAHIT
  role: Assembles QCed reads de novo per sample into contigs on which ORFs are predicted
- name: DIAMOND
  role: Aligns Prokka-predicted ORFs against UniRef90 to assign gene identities for
    abundance quantification
- name: Snakemake
  role: Orchestrates the OMARU_QCed1.sm profiling stage containing the assembly and
    ORF-prediction steps
  repo: https://snakemake.readthedocs.io/en/v5.1.4/index.html
- name: OMARU
  role: End-to-end workflow that packages the assembly, ORF prediction, annotation,
    and quantification stages as a Conda installable pipeline
  repo: https://github.com/toshi-kishikawa/OMARU
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:6f4fc49fc4010caf5bcb9a0e96a70b0dcaad1a7f192e191763ef743b41912cf0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/toshi-kishikawa/OMARU
  tools:
  - Conda
  - DIAMOND
  - MEGAHIT
  - OMARU
  - Prokka
  - Snakemake
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# orf-prediction-from-contigs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts open reading frames (ORFs) on de novo assembled shotgun metagenome contigs with Prokka, as the gene-identity step of the OMARU assembly-based functional profiling pipeline that precedes DIAMOND alignment against UniRef90.

## When to use

Use when you have per-sample QCed shotgun FASTQ files and need gene-level functional profiling: contigs are assembled de novo (MEGAHIT) and ORFs must be called on them before UniRef90 annotation.

## When NOT to use

- Input is a pre-computed gene abundance/feature table rather than raw or QCed FASTQ reads — this skill only covers the read-to-contig-to-ORF construction stage
- You only need phylogenetic (clade-level) profiling, which OMARU runs against UHGG via Prepare_reference_UHGG.sh instead of the UniRef90 functional path
- Reads are amplicon/16S rather than shotgun metagenomic, since OMARU's workflow is designed for shotgun sequencing reads

## Inputs

- QCed shotgun metagenomic FASTQ files (per-sample paired-end) in <OMARU_project_dir>/result/QC/QCed_fastq
- Prepared UniRef90 functional reference database with gene and pathway annotation files in OMARU_dir/OMARU_databases
- OMARU project directory with config.yaml (REF: GENE defaults to UniRef90)

## Outputs

- De novo assembled contigs per sample (MEGAHIT)
- Predicted ORFs annotated with UniRef90 gene IDs via DIAMOND
- Gene abundance matrix normalized by gene length and total read count, with rank-based inverse normal transformation
- Profiling summary tables and figures in <OMARU_project_dir>/result/<Phenotype>_summary

## How to apply

First prepare the functional reference by running Prepare_reference_UniRef90.sh against the OMARU_databases directory, then execute the OMARU Snakemake profiling stage (Snakefiles/OMARU_QCed1.sm) on QCed FASTQ files placed in <OMARU_project_dir>/result/QC/QCed_fastq, which assembles reads de novo per sample with MEGAHIT. Prokka then predicts ORFs on the assembled contigs; the rationale is that assembled contigs give longer coding sequences than raw reads, improving DIAMOND assignment against UniRef90. Predicted ORFs are aligned against UniRef90 with DIAMOND to assign gene identities, and gene abundance is quantified from ORF alignment depth, normalized by gene length and each sample's total read count, then transformed with a rank-based inverse normal transformation. After profiling, select samples with sufficient quality, update the sample list as QCed2_sample_list.txt in <OMARU_project_dir>/data, and check the gene abundance summary in <OMARU_project_dir>/result/<Phenotype>_summary before proceeding to OMARU_QCed2.sm.

## Related tools

- **Prokka** (Predicts open reading frames from de novo assembled contigs prior to gene annotation)
- **MEGAHIT** (Assembles QCed reads de novo per sample into contigs on which ORFs are predicted)
- **DIAMOND** (Aligns Prokka-predicted ORFs against UniRef90 to assign gene identities for abundance quantification)
- **Snakemake** (Orchestrates the OMARU_QCed1.sm profiling stage containing the assembly and ORF-prediction steps) — https://snakemake.readthedocs.io/en/v5.1.4/index.html
- **OMARU** (End-to-end workflow that packages the assembly, ORF prediction, annotation, and quantification stages as a Conda installable pipeline) — https://github.com/toshi-kishikawa/OMARU

## Examples

```
cd OMARU_project_dir && snakemake -s Snakefiles/OMARU_QCed1.sm
```

## Evaluation signals

- Verify the output gene abundance matrix exists after OMARU_QCed1.sm completes
- Check that gene-level QC retains gene counts consistent with the published runs: 185,663 genes for SCZ (n=344) and 104,487 genes for T2D (n=171)
- Confirm profiling summary tables and figures were written to <OMARU_project_dir>/result/<Phenotype>_summary
- Confirm ORFs are annotated against UniRef90 (config.yaml REF: GENE default) and abundances are normalized by gene length and per-sample total read counts with rank-based inverse normal transformation applied
- Confirm sample filtering steps were followed: QCed1_sample_list.txt updated after profiling, then QCed2_sample_list.txt for samples with sufficient quality

## Limitations

- The downstream statistical model does not integrate covariates into a case-control model for hidden confounding — adjustment is instead done via correction of hidden confounding factors outside the model
- Reference database preparation (Prepare_reference_UniRef90.sh) is time and resource-intensive, taking several hours and almost 200 GB of free disk space
- Metagenome analysis is highly dependent on reference databases; using an up-to-date UniRef90 requires customization of the reference data
- No specific versions of tools or reference databases are stated, which may affect reproducibility of ORF-prediction and annotation results

## Evidence

- [methods] ORFs predicted on assembled contigs with Prokka: [evidence span withheld — anchor status: not_found]
