---
name: gene-taxa-link-analysis
description: Use when a case-control gene association test has flagged disease-associated
  genes (e.g. from KEGG/UniRef90-based gene analysis of SCZ metagenomes) and you need
  a gene-to-taxon attribution by tracing sequencing reads back to source organisms.
when_to_use_negative:
- The input reads have not passed OMARU read QC and profiling steps (OMARU_read_QC.sm,
  OMARU_QCed1.sm, OMARU_QCed2.sm) and no QCed sample list has been generated
- The goal is a case-control differential abundance MWAS (phylogenetic, gene, or pathway
  association testing) rather than assigning organisms of origin to already-identified
  genes
- No target genes have been shortlisted from the gene association test — TARGETS would
  be empty and there is nothing to trace back
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: OMARU
  role: End-to-end workflow providing the phylogenetic-functional link module (OMARU_Phy_Fun_link.sm)
    that maps target-gene reads against the phylogenetic reference to assign organisms
    of origin
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine used to execute the OMARU_Phy_Fun_link.sm Snakefile for the
    gene-to-taxon traceback
- name: Conda
  role: Installation/packaging mechanism for OMARU (version 4.8 or greater required;
    conda-pack bundle from Zenodo)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:b7094f54b69d5114126bc9bf0013c9f1347c3d4bff560c151894464036a18321
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
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# gene-taxa-link-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Attribute disease-associated metagenomic genes to their organisms of origin via read-level phylogenetic traceback in OMARU. This links gene-level MWAS findings (e.g. the SCZ-associated gene Vpar_1847) to taxa, resolving which organisms carry the associated genes.

## When to use

Use when a case-control gene association test has flagged disease-associated genes (e.g. from KEGG/UniRef90-based gene analysis of SCZ metagenomes) and you need a gene-to-taxon attribution by tracing sequencing reads back to source organisms.

## When NOT to use

- The input reads have not passed OMARU read QC and profiling steps (OMARU_read_QC.sm, OMARU_QCed1.sm, OMARU_QCed2.sm) and no QCed sample list has been generated
- The goal is a case-control differential abundance MWAS (phylogenetic, gene, or pathway association testing) rather than assigning organisms of origin to already-identified genes
- No target genes have been shortlisted from the gene association test — TARGETS would be empty and there is nothing to trace back

## Inputs

- QCed shotgun metagenomic FASTQ files (<Sample_ID>_R1.fastq.gz, <Sample_ID>_R2.fastq.gz)
- QCed sample list (e.g. QCed_sample_list.txt) in <OMARU_project_dir>/data
- List of disease-associated target genes (e.g. Vpar_1847) set as TARGETS in config.yaml
- UHGG-based phylogenetic reference with NCBI_species_scaffold and NCBI_lineage files

## Outputs

- Per-gene taxonomic attribution summary tables at <OMARU_project_dir>/result/PHYL_FUNC_link/<TARGETS>
- Attribution figures at <OMARU_project_dir>/result/PHYL_FUNC_link/<TARGETS>
- Compiled gene-to-taxon attribution table listing each target gene and the taxa its reads are estimated to derive from

## How to apply

After completing OMARU QC and association testing on shotgun FASTQ data, place the QCed paired reads (<Sample_ID>_R1.fastq.gz/<Sample_ID>_R2.fastq.gz) in <OMARU_project_dir>/data/original_fastq along with the QCed sample list. Set the disease-associated genes to trace (e.g. Vpar_1847) as the TARGETS parameter in <OMARU_project_dir>/config.yaml. Run the phylogenetic-functional link module with Snakemake: snakemake -s Snakefiles/OMARU_Phy_Fun_link.sm, which maps reads of the target genes against the phylogenetic reference (UHGG-based, e.g. CHOCO) to assign organisms of origin. Collect the per-gene taxonomic attribution summary tables and figures written to <OMARU_project_dir>/result/PHYL_FUNC_link/<TARGETS>, then compile a gene-to-taxon attribution table for all target genes.

## Related tools

- **OMARU** (End-to-end workflow providing the phylogenetic-functional link module (OMARU_Phy_Fun_link.sm) that maps target-gene reads against the phylogenetic reference to assign organisms of origin) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine used to execute the OMARU_Phy_Fun_link.sm Snakefile for the gene-to-taxon traceback)
- **Conda** (Installation/packaging mechanism for OMARU (version 4.8 or greater required; conda-pack bundle from Zenodo))

## Examples

```
cd OMARU_project_dir && snakemake -s Snakefiles/OMARU_Phy_Fun_link.sm
```

## Evaluation signals

- The per-gene attribution table exists at <OMARU_project_dir>/result/PHYL_FUNC_link/<TARGETS>
- The output table contains both gene and taxon fields for every gene listed in TARGETS
- In the paper's SCZ validation case, Vpar_1847 is attributed to multiple Veillonella spp., confirming a gene-to-taxon attribution was produced
- Every TARGETS gene receives a read-level taxonomic assignment derived from the UHGG-based phylogenetic reference rather than an empty or placeholder result
- The QCed FASTQ inputs used match the sample list that passed all prior OMARU QC stages

## Limitations

- Metagenome analysis is highly dependent on reference databases; gene-to-taxon resolution is bounded by the UHGG-based phylogenetic reference and database development remains a future challenge
- Attribution is read-level estimation, so genes shared across related organisms (e.g. multiple Veillonella spp.) cannot be pinned to a single species
- Hidden confounding factors are corrected as hidden factors rather than by integrating covariates into a case-control model, which is not currently implemented in OMARU

## Evidence
