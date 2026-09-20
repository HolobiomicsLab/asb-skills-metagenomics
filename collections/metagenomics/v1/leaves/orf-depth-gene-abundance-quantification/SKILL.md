---
name: orf-depth-gene-abundance-quantification
description: Use when building gene-level functional abundance/profiling data from
  QCed shotgun metagenomic FASTQ reads (e.g. as part of OMARU's Step 2, OMARU_QCed1.sm),
  before gene-level MWAS association testing.
when_to_use_negative:
- When the goal is phylogenetic (taxonomic/clade) profiling from UHGG-based references
  — use the OMARU_Phyl_AS module instead of the UniRef90 functional profiling module
- When input reads are 16S amplicon data rather than shotgun metagenomic FASTQ, since
  the pipeline assembles de novo whole-metagenome reads
- When the analysis goal is MWAS-GWAS pathway interaction or taxon-gene linking on
  an existing abundance matrix — those steps consume, rather than construct, the gene
  abundance output
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3273
tools:
- name: OMARU
  role: End-to-end Snakemake workflow that implements the assembly-based functional
    profiling stage
  repo: https://github.com/toshi-kishikawa/OMARU
- name: MEGAHIT
  role: De novo assembly of QCed shotgun reads into contigs per sample
- name: Prokka
  role: Open reading frame (ORF) prediction on assembled contigs
- name: DIAMOND
  role: Alignment of predicted ORFs against UniRef90 to assign gene identities
- name: Snakemake
  role: Workflow engine driving the profiling stage (Snakefiles/OMARU_QCed1.sm)
  repo: https://snakemake.readthedocs.io/en/v5.1.4/index.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:9d75d46788ea7a5d6d6472eb79a64779fb2f4fb53379e4f37c92e046b2c06a12
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
  - DIAMOND
  - MEGAHIT
  - OMARU
  - Prokka
  - Snakemake
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3273
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# orf-depth-gene-abundance-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs gene-level functional abundance matrices from shotgun metagenomic reads in the OMARU pipeline: per-sample de novo assembly with MEGAHIT, ORF prediction with Prokka, DIAMOND alignment against UniRef90, alignment-depth quantification normalized by gene length and total reads, and rank-based inverse normal transformation of abundance values.

## When to use

Use when building gene-level functional abundance/profiling data from QCed shotgun metagenomic FASTQ reads (e.g. as part of OMARU's Step 2, OMARU_QCed1.sm), before gene-level MWAS association testing.

## When NOT to use

- When the goal is phylogenetic (taxonomic/clade) profiling from UHGG-based references — use the OMARU_Phyl_AS module instead of the UniRef90 functional profiling module
- When input reads are 16S amplicon data rather than shotgun metagenomic FASTQ, since the pipeline assembles de novo whole-metagenome reads
- When the analysis goal is MWAS-GWAS pathway interaction or taxon-gene linking on an existing abundance matrix — those steps consume, rather than construct, the gene abundance output

## Inputs

- QCed shotgun metagenomic FASTQ files (<Sample_ID>_R1.fastq.gz / <Sample_ID>_R2.fastq.gz) in <OMARU_project_dir>/result/QC/QCed_fastq
- UniRef90 functional reference database with gene annotation files (<gene_reference>_annotatioin.txt.gz) and pathway annotation files (<pathway_reference>_annotatioin.txt.gz, GO term based) prepared in <OMARU_dir>/OMARU_databases
- QCed1_sample_list.txt with sample metadata (Sample, Phenotype, Sex, Age columns) and covariates.txt

## Outputs

- Gene abundance matrix (ORF alignment depth normalized by gene length and per-sample total read count, rank-based inverse normal transformed)
- Profiling summary tables and figures at <OMARU_project_dir>/result/<Phenotype>_summary
- Gene-level QC-filtered abundance data (185,663 genes for SCZ; 104,487 genes for T2D in the paper)

## How to apply

First prepare the functional reference by running Prepare_reference_UniRef90.sh against <OMARU_dir>/OMARU_databases so the UniRef90 FASTA plus gene and pathway annotation files (e.g. GO term annotations) are available. Then execute the profiling stage of the Snakemake workflow on QCed FASTQ files placed in <OMARU_project_dir>/result/QC/QCed_fastq (snakemake -s Snakefiles/OMARU_QCed1.sm): assemble reads de novo per sample with MEGAHIT, predict open reading frames from the assembled contigs with Prokka, and align the predicted ORFs against UniRef90 using DIAMOND to assign gene identities. Quantify gene abundance from ORF alignment depth, normalizing by gene length and each sample's total read count, then apply a rank-based inverse normal transformation to the abundance values so they follow a standard normal distribution for downstream association tests. Apply gene-level QC before association testing; in the paper this retained 185,663 genes for the SCZ case-control comparison and 104,487 genes for T2D. Profiling summary tables and figures are written to <OMARU_project_dir>/result/<Phenotype>_summary for inspection before updating the sample list (QCed2_sample_list.txt).

## Related tools

- **OMARU** (End-to-end Snakemake workflow that implements the assembly-based functional profiling stage) — https://github.com/toshi-kishikawa/OMARU
- **MEGAHIT** (De novo assembly of QCed shotgun reads into contigs per sample)
- **Prokka** (Open reading frame (ORF) prediction on assembled contigs)
- **DIAMOND** (Alignment of predicted ORFs against UniRef90 to assign gene identities)
- **Snakemake** (Workflow engine driving the profiling stage (Snakefiles/OMARU_QCed1.sm)) — https://snakemake.readthedocs.io/en/v5.1.4/index.html

## Examples

```
cd OMARU_project_dir && snakemake -s Snakefiles/OMARU_QCed1.sm
```

## Evaluation signals

- Gene-level QC retains the reported counts of 185,663 genes for SCZ and 104,487 genes for T2D case-control comparison
- Profiling summary tables and figures are present at <OMARU_project_dir>/result/<Phenotype>_summary
- Abundance values are normalized by gene length and each sample's total read count, and follow a standard normal distribution after the rank-based inverse normal transformation
- Downstream gene association tests (OMARU_Func_AS.sm) run on the transformed matrix with P-value inflation suppressed by covariate adjustment (sex, age, BMI, top PCs)

## Limitations

- Preparing the reference databases (including UniRef90) can take several hours and requires almost 200GB of free disk space
- Metagenome analysis is highly dependent on reference databases; database development is noted as a challenge for the future
- No specific versions of tools or reference databases are stated in the paper
- The OMARU README lists 'Currently under development' for publication/citation details, so pin versions when reproducing

## Evidence
