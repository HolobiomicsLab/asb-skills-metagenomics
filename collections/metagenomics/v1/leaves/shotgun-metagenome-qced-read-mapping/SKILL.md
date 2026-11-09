---
name: shotgun-metagenome-qced-read-mapping
description: Use when you have shotgun metagenomic reads that passed all QC stages
  and need to assign reads to reference taxa/genes, or attribute a disease-associated
  gene to its source organisms via read-level traceback.
when_to_use_negative:
- Input reads have not passed read/sample QC — run OMARU_read_QC.sm and update the
  sample list first
- You need de novo assembly or binning of the metagenome rather than reference-based
  mapping against UHGG/CHOCO
- Your question is case-control association testing rather than gene-to-taxon attribution
  or profiling
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: OMARU
  role: End-to-end workflow whose Snakefiles orchestrate mapping of QCed shotgun reads
    to the phylogenetic reference and the phylogenetic-functional link (gene-to-taxon
    traceback) module
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine used to execute the mapping/link modules (e.g., snakemake
    -s Snakefiles/OMARU_Phy_Fun_link.sm)
- name: Conda
  role: Package manager required (version 4.8+) to install and activate the OMARU
    conda-pack environment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:7578f1b58de5ff57476037350122d1521efede202d72e09a1f8a20418daaee05
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# shotgun-metagenome-qced-read-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps QCed shotgun metagenomic FASTQ reads against a phylogenetic reference (e.g., UHGG-based CHOCO in Bowtie format) with OMARU to construct taxonomic/functional profiles and perform read-level traceback linking genes (e.g., SCZ-associated Vpar_1847) to their organisms of origin.

## When to use

Use when you have shotgun metagenomic reads that passed all QC stages and need to assign reads to reference taxa/genes, or attribute a disease-associated gene to its source organisms via read-level traceback.

## When NOT to use

- Input reads have not passed read/sample QC — run OMARU_read_QC.sm and update the sample list first
- You need de novo assembly or binning of the metagenome rather than reference-based mapping against UHGG/CHOCO
- Your question is case-control association testing rather than gene-to-taxon attribution or profiling

## Inputs

- QCed paired-end shotgun metagenomic reads named <Sample_ID>_R1.fastq.gz / <Sample_ID>_R2.fastq.gz
- QCed sample list (e.g., QCed1_sample_list.txt) in <OMARU_project_dir>/data
- Bowtie-format phylogenetic reference database (UHGG/CHOCO FASTA with NCBI_species_scaffold_, NCBI_lineage_, eachL_lineage_ files)
- <OMARU_project_dir>/config.yaml with TARGETS set to genes of interest (e.g., ["Vpar_1847"])

## Outputs

- Per-gene taxonomic attribution summary tables and figures in <OMARU_project_dir>/result/PHYL_FUNC_link/<TARGETS>
- Compiled gene-to-taxon attribution table listing each target gene and the taxa its reads derive from
- Phylogenetic/functional profiling abundance tables and figures under result/ when used in the main OMARU workflow

## How to apply

Place the QCed paired-end reads as <Sample_ID>_R1.fastq.gz / <Sample_ID>_R2.fastq.gz in <OMARU_project_dir>/data/original_fastq alongside the QCed sample list (e.g., QCed1_sample_list.txt). Prepare the phylogenetic reference with Prepare_reference_UHGG.sh (Bowtie-format FASTA plus NCBI_species_scaffold_, NCBI_lineage_, and eachL_lineage_ tables) and record the reference name (default CHOCO) in config.yaml. For gene-to-taxon traceback, set the target genes (e.g., "Vpar_1847") as the TARGETS parameter in <OMARU_project_dir>/config.yaml, then run the phylogenetic-functional link module with Snakemake (snakemake -s Snakefiles/OMARU_Phy_Fun_link.sm), which maps reads of the target genes against the phylogenetic reference to assign organisms of origin. Collect per-gene attribution summaries from <OMARU_project_dir>/result/PHYL_FUNC_link/<TARGETS> and compile the gene-to-taxon table; for profiling-only use, the mapping feeds phylogenetic/functional abundance tables under result/ via the OMARU_QCed1.sm step.

## Related tools

- **OMARU** (End-to-end workflow whose Snakefiles orchestrate mapping of QCed shotgun reads to the phylogenetic reference and the phylogenetic-functional link (gene-to-taxon traceback) module) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine used to execute the mapping/link modules (e.g., snakemake -s Snakefiles/OMARU_Phy_Fun_link.sm))
- **Conda** (Package manager required (version 4.8+) to install and activate the OMARU conda-pack environment)

## Examples

```
snakemake -s Snakefiles/OMARU_Phy_Fun_link.sm
```

## Evaluation signals

- The output table <result>/PHYL_FUNC_link/<TARGETS> exists and contains gene and taxon fields
- Vpar_1847 is attributed to multiple Veillonella spp. in the gene-to-taxon attribution table
- Input FASTQ files conform to the <Sample_ID>_R1.fastq.gz / <Sample_ID>_R2.fastq.gz naming convention in data/original_fastq
- The sample list used for mapping matches the QCed samples (e.g., QCed1/QCed2/QCed3_sample_list.txt), excluding samples that failed QC
- Custom phylogenetic references include all three required tables (NCBI_species_scaffold_, NCBI_lineage_, eachL_lineage_) in <OMARU_dir>/OMARU_databases

## Limitations

- Metagenome mapping is highly dependent on reference databases; genes absent from the UHGG/CHOCO reference cannot be attributed, and database development is stated as a future challenge
- Reference database download/preparation is resource-intensive: several hours and almost 200GB of free disk space
- Custom phylogenetic references require manual conversion to Bowtie format plus three annotation tables; gene-to-taxon attribution is a read-level estimate, not a provenance guarantee

## Evidence

- [other] Read-level traceback finding: [evidence span withheld — anchor status: not_found]
