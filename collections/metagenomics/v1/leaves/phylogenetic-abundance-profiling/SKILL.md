---
name: phylogenetic-abundance-profiling
description: Use when starting from shotgun metagenomic FASTQ files and you need clade-resolved
  (UHGG-based) phylogenetic abundance profiles with rigorous read, sample, and clade
  QC ahead of case-control MWAS testing.
when_to_use_negative:
- Input is already a phylogenetic/clade abundance table rather than raw shotgun FASTQ
  reads — use the association-test Snakefiles (Step 4) directly instead
- The goal is functional (gene/pathway) profiling based on UniRef90 and GO rather
  than clade-level phylogenetic profiling
- Only short-read 16S amplicon data are available — OMARU's workflow is built for
  shotgun sequencing reads
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: OMARU
  role: End-to-end workflow performing read QC, phylogenetic profiling against UHGG,
    and clade/sample abundance QC via Snakemake Snakefiles
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine executing the OMARU read-QC, profiling (OMARU_QCed1.sm) and
    abundance-QC (OMARU_QCed2.sm) Snakefiles
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Conda
  role: Required (version 4.8 or greater) to install and activate the OMARU conda-pack
    environment
- name: UHGG
  role: Reference database of phylogenetic analyses downloaded in FASTA format and
    prepared into OMARU_databases
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:de27368cdcf3a33d7f228fbd6108f17d1479e7112c9f7f25892b31843d4cd2f6
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
  - UHGG
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# phylogenetic-abundance-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct clade-level phylogenetic abundance profiles of shotgun metagenomic samples by mapping reads to a UHGG-based reference with OMARU's Snakemake workflow, then quality-control clades and samples before case-control association testing.

## When to use

Use when starting from shotgun metagenomic FASTQ files and you need clade-resolved (UHGG-based) phylogenetic abundance profiles with rigorous read, sample, and clade QC ahead of case-control MWAS testing.

## When NOT to use

- Input is already a phylogenetic/clade abundance table rather than raw shotgun FASTQ reads — use the association-test Snakefiles (Step 4) directly instead
- The goal is functional (gene/pathway) profiling based on UniRef90 and GO rather than clade-level phylogenetic profiling
- Only short-read 16S amplicon data are available — OMARU's workflow is built for shotgun sequencing reads

## Inputs

- Shotgun metagenomic reads in FASTQ format named <Sample_ID>_R1.fastq.gz / <Sample_ID>_R2.fastq.gz in <OMARU_project_dir>/data/original_fastq
- Sample list with metadata (original_sample_list.txt with columns Sample, Phenotype, Sex, Age) in <OMARU_project_dir>/data
- UHGG-based phylogenetic reference database in FASTA/Bowtie format (via Prepare_reference_UHGG.sh)
- config.yaml with parameters such as PHYL_THRESHOLD, THREAD, and REF: PHYL

## Outputs

- QCed FASTQ files in <OMARU_project_dir>/result/QC/QCed_fastq
- Tables and figures of profiling QC summaries in <OMARU_project_dir>/result/<Phenotype>_summary
- Phylogenetic abundance profiles and basic statistics in result/PHYL_QCed2/<Phylogenetic_reference>_graph_basic
- Progressively filtered sample lists: QCed1_sample_list.txt, QCed2_sample_list.txt, QCed3_sample_list.txt

## How to apply

Prepare the phylogenetic reference with Prepare_reference_UHGG.sh into OMARU_dir/OMARU_databases (custom references require a Bowtie-formatted FASTA plus NCBI_species_scaffold, NCBI_lineage, and eachL_lineage text files). Run read QC (snakemake -s Snakefiles/OMARU_read_QC.sm) against phix/adapters and the human genome reference, updating the sample list to QCed1_sample_list.txt, then construct phylogenetic and functional profiling with OMARU_QCed1.sm. Inspect profiling QC summaries, keep only samples with sufficient quality (QCed2_sample_list.txt), and check phylogenetic abundance data via OMARU_QCed2.sm before saving the final QCed3_sample_list.txt. Control clade retention with PHYL_THRESHOLD in config.yaml (default 5, meaning a 1e-5 relative abundance cutoff); note that reference preparation can take several hours and nearly 200GB of disk.

## Related tools

- **OMARU** (End-to-end workflow performing read QC, phylogenetic profiling against UHGG, and clade/sample abundance QC via Snakemake Snakefiles) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine executing the OMARU read-QC, profiling (OMARU_QCed1.sm) and abundance-QC (OMARU_QCed2.sm) Snakefiles) — https://github.com/toshi-kishikawa/OMARU
- **Conda** (Required (version 4.8 or greater) to install and activate the OMARU conda-pack environment)
- **UHGG** (Reference database of phylogenetic analyses downloaded in FASTA format and prepared into OMARU_databases)

## Examples

```
snakemake -s Snakefiles/OMARU_QCed1.sm
```

## Evaluation signals

- QCed FASTQ files exist in result/QC/QCed_fastq for every sample passing read QC
- Sample list is successively updated to QCed1_sample_list.txt, QCed2_sample_list.txt, and QCed3_sample_list.txt as filters are applied
- Profiling QC tables and figures appear in result/<Phenotype>_summary and result/PHYL_QCed2/<Phylogenetic_reference>_graph_basic
- Clades retained respect the PHYL_THRESHOLD relative abundance cutoff (default 5 = 1e-5)
- Custom references include all three required lineage files (NCBI_species_scaffold, NCBI_lineage, eachL_lineage) in OMARU_databases

## Limitations

- Reference database download and preparation is time and resource-intensive, requiring several hours and almost 200GB of free disk space
- Metagenome analysis remains highly dependent on reference databases, which the authors flag as a future challenge
- Custom phylogenetic references must be manually converted to Bowtie format with three accompanying lineage annotation files

## Evidence
