---
name: blastn-all-vs-all-genome-comparison
description: Use when a set of viral genomes (e.g. CheckV-quality-filtered FASTAs
  from metagenomic assembly) must be dereplicated into species-level vOTUs, or when
  vOTU representatives need to be screened against external virome databases.
when_to_use_negative:
- Input sequences are prokaryotic MAGs rather than viral genomes — those are dereplicated
  into 786 SGBs with the 95% nucleotide similarity clustering workflow, not vOTU clustering
- Viral genomes have not passed the >=50% completeness CheckV filter, since the vOTU
  counts and novelty estimates were computed on the filtered set
- The goal is host prediction or taxonomic annotation of viruses (virusHost.sh / virusTaxAnno.sh),
  not clustering or database overlap
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
tools:
- name: BLAST
  role: All-vs-all nucleotide comparison of viral genomes and BLASTn of vOTU representatives
    against the five external virome databases
- name: CheckV
  role: Quality filter defining the viral genome input set (>=50% completeness) prior
    to clustering
- name: virusClstr.sh
  role: Clustering of BLASTn all-vs-all hits into vOTUs at 95% identity over 85% of
    the genome
  repo: https://github.com/RChGO/VMGC/blob/main/Pipelines/virusClstr.sh
- name: Prodigal
  role: Extraction of protein-coding genes from vOTU representatives in metagenomic
    mode for downstream annotation
- name: GNU parallel
  role: Parallelization of the per-sample/per-genome commands in the VMGC pipelines
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:a7669e8ce64d39dab8ff23c07bd34f90950c0045226510750ffbaee3aba14abc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - CheckV
  - GNU parallel
  - NCBI RefSeq
  - Prodigal
  - virusClstr.sh
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# blastn-all-vs-all-genome-comparison

## Summary

Defines species-level viral operational taxonomic units (vOTUs) by all-vs-all nucleotide BLASTn comparison of quality-filtered viral genomes, clustering at 95% identity over 85% of the genome. In the VMGC paper this reduced 14,224 CheckV-filtered vaginal viral genomes to 4,263 vOTUs and enabled the database-overlap analysis showing 85.8% novelty.

## When to use

Use when a set of viral genomes (e.g. CheckV-quality-filtered FASTAs from metagenomic assembly) must be dereplicated into species-level vOTUs, or when vOTU representatives need to be screened against external virome databases.

## When NOT to use

- Input sequences are prokaryotic MAGs rather than viral genomes — those are dereplicated into 786 SGBs with the 95% nucleotide similarity clustering workflow, not vOTU clustering
- Viral genomes have not passed the >=50% completeness CheckV filter, since the vOTU counts and novelty estimates were computed on the filtered set
- The goal is host prediction or taxonomic annotation of viruses (virusHost.sh / virusTaxAnno.sh), not clustering or database overlap

## Inputs

- Quality-filtered viral genome FASTA files (>=50% completeness by CheckV; n=14,224 from VMGC_virus.tar.gz)
- Reference virome databases: Gut Virome Database, Gut Phage Database, Metagenomic Gut Virus catalogue, Oral Virus Database, NCBI RefSeq viral genomes
- virusClstr.sh clustering script from the RChGO/VMGC repository

## Outputs

- vOTU cluster assignments for 4,263 vOTUs with representative sequences per cluster
- Prodigal (metagenomic mode) protein-coding gene predictions from vOTU representatives
- Per-vOTU comparison results and summary non-overlap percentage written to votu_overlap_results.csv

## How to apply

Retrieve the quality-filtered viral genomes (here the 14,224 sequences with >=50% completeness by CheckV, e.g. VMGC_virus.tar.gz from Zenodo record 10457006) and run an all-vs-all nucleotide comparison with BLASTn. Cluster the BLASTn results with the VMGC virusClstr.sh script using the 95% identity over 85% of the genome threshold to yield vOTUs and one representative sequence per vOTU (4,263 in the paper). Extract protein-coding genes from vOTU representatives with Prodigal in metagenomic mode for downstream annotation. The same BLASTn machinery is then reused to compare vOTU representatives against Gut Virome Database, Gut Phage Database, Metagenomic Gut Virus catalogue, Oral Virus Database and NCBI RefSeq viral genomes; vOTUs with no significant match in any of the five databases are counted to compute the non-overlap fraction (85.8%). The 95% nucleotide similarity threshold mirrors the species-level genome bin (SGB) criterion used for the prokaryotic genomes in the same study.

## Related tools

- **BLAST** (All-vs-all nucleotide comparison of viral genomes and BLASTn of vOTU representatives against the five external virome databases)
- **CheckV** (Quality filter defining the viral genome input set (>=50% completeness) prior to clustering)
- **virusClstr.sh** (Clustering of BLASTn all-vs-all hits into vOTUs at 95% identity over 85% of the genome) — https://github.com/RChGO/VMGC/blob/main/Pipelines/virusClstr.sh
- **Prodigal** (Extraction of protein-coding genes from vOTU representatives in metagenomic mode for downstream annotation)
- **GNU parallel** (Parallelization of the per-sample/per-genome commands in the VMGC pipelines)

## Examples

```
blastn -query votu_reps.fna -subject all_viral_genomes.fna -outfmt "6 qseqid sseqid pident length qlen" -perc_identity 95 | virusClstr.sh > votu_clusters.tsv
```

## Evaluation signals

- Clustering of the 14,224 CheckV-filtered viral genomes yields exactly 4,263 vOTUs
- Computed non-overlap fraction against the five virome databases matches the reported 85.8% within a small tolerance
- votu_overlap_results.csv contains a per-vOTU comparison row and a summary percentage
- Every vOTU has exactly one representative genome sequence from which Prodigal (metagenomic mode) gene calls were produced
- BLASTn search parameters enforce 95% identity over 85% of the genome length before sequences are merged into a vOTU

## Limitations

- Many phages in the VMGC are unknown, and their functional exploration based on existing databases remains limited
- 85.8% absence from the five queried databases does not prove true novelty — unexplored database coverage could still contain homologs
- Potential contamination from experimental operations or kits in the public vaginal metagenome datasets used to construct VMGC may propagate into the vOTU set

## Evidence

- [abstract] The result revealed that 85.8% of vOTUs in the VMGC were not found in any of the other virome databases: "The result revealed that 85.8% of vOTUs in the VMGC were not found in any of the other virome databases"
- [abstract] comprising 33,804 microbial genomes spanning 786 prokaryotic species, 11 fungal species and 4,263 vOTUs: "comprising 33,804 microbial genomes spanning 786 prokaryotic species, 11 fungal species and 4,263 viral operational taxonomic units"
- [methods] Perform all-vs-all nucleotide comparison with BLASTn and cluster with virusClstr.sh at 95% identity over 85% of the genome: [evidence span withheld — anchor status: not_found]
- [readme] Full viral genomes (n=14,224) and vOTUs (n=4,263) stored as VMGC_virus.tar.gz: "Full viral genomes (n=14,224) and vOTUs (n=4,263)| 197 MB | VMGC_virus.tar.gz"
