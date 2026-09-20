---
name: metagenomic-read-taxonomic-attribution
description: Use when disease-associated genes from MWAS must be assigned to taxa
  in a shotgun metagenome, e.g., tracing which organisms reads of a specific gene
  derive from.
when_to_use_negative:
- When the goal is de novo taxonomic profiling of the whole metagenome rather than
  attributing specific known target genes — use the phylogenetic profiling steps (OMARU_Phyl_AS.sm)
  instead
- When reads have not passed OMARU read/sample QC stages; raw FASTQ should not be
  fed directly to the traceback module
- When no suitable phylogenetic reference database (e.g., UHGG-based) covers the candidate
  organisms, since attribution depends on reference databases
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: OMARU
  role: End-to-end workflow providing the phylogenetic-functional link module that
    assigns organisms of origin to target genes via read-level traceback
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine used to execute the OMARU_Phy_Fun_link.sm Snakefile
- name: Conda
  role: Installation/dependency management for OMARU (version 4.8 or greater; conda-pack
    from Zenodo record 5201539)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:6e735fa674974b7daaa1f6223d2ba5a5b47ad0854334c1167b3a18f27fdf60ed
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metagenomic-read-taxonomic-attribution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies OMARU's phylogenetic-functional link module to trace sequencing reads of target genes (e.g., disease-associated genes such as the SCZ-associated Vpar_1847) back to their organisms of origin, producing a gene-to-taxon attribution table for the metagenomic sample.

## When to use

Use when disease-associated genes from MWAS must be assigned to taxa in a shotgun metagenome, e.g., tracing which organisms reads of a specific gene derive from.

## When NOT to use

- When the goal is de novo taxonomic profiling of the whole metagenome rather than attributing specific known target genes — use the phylogenetic profiling steps (OMARU_Phyl_AS.sm) instead
- When reads have not passed OMARU read/sample QC stages; raw FASTQ should not be fed directly to the traceback module
- When no suitable phylogenetic reference database (e.g., UHGG-based) covers the candidate organisms, since attribution depends on reference databases

## Inputs

- QCed shotgun metagenomic FASTQ files (<Sample_ID>_R1.fastq.gz / <Sample_ID>_R2.fastq.gz) in <OMARU_project_dir>/data/original_fastq
- QCed sample list (e.g., QCed1/QCed2/QCed3_sample_list.txt) in <OMARU_project_dir>/data
- List of target gene IDs to trace (e.g., SCZ-associated gene Vpar_1847) set as TARGETS in <OMARU_project_dir>/config.yaml
- Phylogenetic reference database (UHGG-based FASTA/Bowtie index with NCBI_species_scaffold and NCBI_lineage files) in <OMARU_dir>/OMARU_databases

## Outputs

- Per-gene taxonomic attribution summary tables and figures in <OMARU_project_dir>/result/PHYL_FUNC_link/<TARGETS>
- Compiled gene-to-taxon attribution table listing each target gene and the taxa from which its reads are estimated to derive (e.g., Vpar_1847 attributed to multiple Veillonella spp.)

## How to apply

First complete the OMARU QC chain (OMARU_read_QC.sm, OMARU_QCed1.sm, OMARU_QCed2.sm) so only QCed shotgun reads (<Sample_ID>_R1/R2.fastq.gz) and a QCed sample list remain in <OMARU_project_dir>/data/original_fastq. Set the genes to be traced (e.g., Vpar_1847) as the TARGETS parameter in <OMARU_project_dir>/config.yaml. Then run the phylogenetic-functional link module with `snakemake -s Snakefiles/OMARU_Phy_Fun_link.sm`, which maps reads of the target genes against the phylogenetic reference (UHGG-based, Bowtie format with NCBI_species_scaffold/NCBI_lineage files) to assign organisms of origin via read-level traceback. Collect per-gene taxonomic attribution summary tables and figures from <OMARU_project_dir>/result/PHYL_FUNC_link/<TARGETS>, and compile the gene-to-taxon attribution table for all target genes.

## Related tools

- **OMARU** (End-to-end workflow providing the phylogenetic-functional link module that assigns organisms of origin to target genes via read-level traceback) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine used to execute the OMARU_Phy_Fun_link.sm Snakefile)
- **Conda** (Installation/dependency management for OMARU (version 4.8 or greater; conda-pack from Zenodo record 5201539))

## Examples

```
cd OMARU_project_dir && snakemake -s Snakefiles/OMARU_Phy_Fun_link.sm
```

## Evaluation signals

- The output directory <OMARU_project_dir>/result/PHYL_FUNC_link/<TARGETS> exists and contains summary tables and figures for each target gene
- The gene-to-taxon attribution table contains both gene and taxon fields
- The known SCZ-associated gene Vpar_1847 is attributed to multiple Veillonella spp., matching the article's reported finding
- Only samples that passed all QC stages (QCed1/QCed2/QCed3 sample lists) contribute reads to the traceback
- TARGETS entries in config.yaml match the genes appearing in the attribution output

## Limitations

- Metagenome analysis is highly dependent on reference databases; attribution quality is limited by the phylogenetic reference (e.g., UHGG) and database development remains a challenge
- Confounding-factor adjustment via covariate integration into a case-control model is not currently implemented in OMARU, which may affect downstream association gene lists used as targets
- Preparing the reference databases is time and resource-intensive, requiring several hours and almost 200 GB of free disk space

## Evidence
