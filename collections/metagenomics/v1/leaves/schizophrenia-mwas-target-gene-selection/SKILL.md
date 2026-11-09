---
name: schizophrenia-mwas-target-gene-selection
description: Use when shotgun metagenome MWAS (e.g., SCZ vs. controls) has yielded
  significant disease-associated genes and you need a gene-to-taxon attribution by
  tracing sequencing reads back to their source organisms.
when_to_use_negative:
- The goal is a case-control association test itself (phylogenetic/gene/pathway MWAS)
  rather than taxonomic attribution of already-identified target genes — use the association-test
  Snakefiles (OMARU_Phyl_AS.sm, OMARU_Func_AS.sm) instead
- Input is 16S amplicon data or already-summarized abundance feature tables — the
  traceback module requires shotgun reads that can be mapped to a phylogenetic reference
- Genes of interest have no representation in the UHGG-based phylogenetic reference
  (e.g., custom gene IDs outside UniRef90/UHGG annotation)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3174
tools:
- name: OMARU
  role: End-to-end workflow providing the phylogenetic-functional link module that
    maps target-gene reads against the phylogenetic reference to assign organisms
    of origin
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine used to execute the OMARU_Phy_Fun_link.sm Snakefile for read-level
    traceback
- name: Conda
  role: Installation/operation requirement (version 4.8 or greater); OMARU distributed
    as a conda-pack tarball from Zenodo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:02c97dcb49fe54dd832b978e44362e991ae01e603989fe810f1e6468aacf34a8
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
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# schizophrenia-mwas-target-gene-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Attribution of disease-associated metagenomic genes (e.g., SCZ-associated gene Vpar_1847) to their organisms of origin via OMARU's read-level traceback phylogenetic-functional link module, which links taxa and genes in the metagenome.

## When to use

Use when shotgun metagenome MWAS (e.g., SCZ vs. controls) has yielded significant disease-associated genes and you need a gene-to-taxon attribution by tracing sequencing reads back to their source organisms.

## When NOT to use

- The goal is a case-control association test itself (phylogenetic/gene/pathway MWAS) rather than taxonomic attribution of already-identified target genes — use the association-test Snakefiles (OMARU_Phyl_AS.sm, OMARU_Func_AS.sm) instead
- Input is 16S amplicon data or already-summarized abundance feature tables — the traceback module requires shotgun reads that can be mapped to a phylogenetic reference
- Genes of interest have no representation in the UHGG-based phylogenetic reference (e.g., custom gene IDs outside UniRef90/UHGG annotation)

## Inputs

- QCed shotgun metagenomic FASTQ files (<Sample_ID>_R1.fastq.gz / <Sample_ID>_R2.fastq.gz) in <OMARU_project_dir>/data/original_fastq
- QCed sample list (QCed1/QCed2/QCed3_sample_list.txt) in <OMARU_project_dir>/data
- List of disease-associated target genes (e.g., Vpar_1847) set as TARGETS in config.yaml
- UHGG-based phylogenetic reference database in <OMARU_dir>/OMARU_databases (with NCBI_species_scaffold, NCBI_lineage, and eachL_lineage files)

## Outputs

- Per-gene taxonomic attribution summary tables in <OMARU_project_dir>/result/PHYL_FUNC_link/<TARGETS>
- Summary figures of taxa-gene links in result/PHYL_FUNC_link/<TARGETS>
- Compiled gene-to-taxon attribution table (gene and taxa from which its reads are estimated to derive)

## How to apply

After completing read QC and profiling, place QCed shotgun FASTQ files (<Sample_ID>_R1.fastq.gz / <Sample_ID>_R2.fastq.gz) in <OMARU_project_dir>/data/original_fastq alongside the QCed sample list. Set the disease-associated genes of interest (including Vpar_1847) as the TARGETS parameter in <OMARU_project_dir>/config.yaml — the README defines TARGETS as 'genes to be evaluated for links with phylogenetic data'. Run the phylogenetic-functional link module with snakemake -s Snakefiles/OMARU_Phy_Fun_link.sm, which maps reads of the target genes against the phylogenetic reference (UHGG-based, e.g., CHOCO) to assign organisms of origin. Collect the per-gene taxonomic attribution summary tables and figures from <OMARU_project_dir>/result/PHYL_FUNC_link/<TARGETS> and compile a gene-to-taxon attribution table recording each gene and the taxa from which its reads derive.

## Related tools

- **OMARU** (End-to-end workflow providing the phylogenetic-functional link module that maps target-gene reads against the phylogenetic reference to assign organisms of origin) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine used to execute the OMARU_Phy_Fun_link.sm Snakefile for read-level traceback)
- **Conda** (Installation/operation requirement (version 4.8 or greater); OMARU distributed as a conda-pack tarball from Zenodo)

## Examples

```
snakemake -s Snakefiles/OMARU_Phy_Fun_link.sm
```

## Evaluation signals

- The output directory result/PHYL_FUNC_link/<TARGETS> exists and contains per-gene taxonomic attribution summary tables and figures for every entry in TARGETS
- The compiled gene-to-taxon table contains both gene and taxon fields
- Vpar_1847 is attributed to multiple Veillonella spp. in the SCZ target metagenomic sample
- Input FASTQ files match the required naming convention <Sample_ID>_R1.fastq.gz / <Sample_ID>_R2.fastq.gz and the samples used are those that passed all QC stages (QCed sample list)

## Limitations

- Metagenome analysis is highly dependent on reference databases; attributions are only as good as the UHGG-based phylogenetic reference (database development is 'a challenge for the future')
- Gene-to-taxon assignment is an estimate from read-level traceback, not a definitive species-of-origin determination; reads shared between related taxa (e.g., within Veillonella) may not resolve at species level
- Preparing the default reference databases is time and resource-intensive, requiring several hours and almost 200GB of free disk space

## Evidence
