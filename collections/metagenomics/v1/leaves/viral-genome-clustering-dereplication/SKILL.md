---
name: viral-genome-clustering-dereplication
description: Use when you have a set of quality-filtered viral genomes (e.g. CheckV
  >=50% completeness) and need to dereplicate them into species-level vOTUs and/or
  test their novelty against existing virome catalogues.
when_to_use_negative:
- Input viral genomes have not been quality-filtered (no CheckV completeness assessment);
  clustering unfiltered contigs inflates vOTU counts with fragments
- The goal is prokaryotic species dereplication at 95% ANI over whole genomes (SGB
  clustering), not viral OTU definition
- Only a handful of viral genomes are present; all-vs-all BLASTn clustering and database
  overlap statistics are not meaningful at that scale
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: BLAST
  role: All-vs-all nucleotide comparison of viral genomes for clustering, and BLASTn
    of vOTU representatives against external virome databases
- name: CheckV
  role: 'Quality filter upstream of clustering: retains viral genomes with >=50% completeness
    (10.3% complete, 29.3% high quality, 60.4% medium quality)'
- name: Prodigal
  role: Extracts protein-coding genes from vOTU representatives in metagenomic mode
    for downstream annotation of the clustered viral set
- name: virusClstr.sh
  role: VMGC pipeline script that clusters viral genomes into vOTUs at 95% identity
    over 85% of the genome
  repo: https://github.com/RChGO/VMGC/blob/main/Pipelines/virusClstr.sh
- name: GNU parallel 20201122
  role: Parallelizes the all-vs-all comparison and database search steps
  repo: https://github.com/RChGO/VMGC/tree/main/Pipelines
- name: NCBI RefSeq
  role: One of the five external virome databases BLASTn-searched to quantify vOTU
    novelty
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:8b1e2d261fa3303a857343ade5517d976f44916a4faa02bd8987ecdb8361aeb2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - CheckV
  - GNU parallel 20201122
  - NCBI RefSeq
  - Prodigal
  - virusClstr.sh
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# viral-genome-clustering-dereplication

## Summary

Dereplicate quality-filtered viral genomes into species-level viral operational taxonomic units (vOTUs) via all-vs-all nucleotide comparison, then quantify overlap against external virome databases. In the VMGC workflow this reduced 14,224 viral genomes to 4,263 vOTUs at 95% identity over 85% of the genome.

## When to use

Use when you have a set of quality-filtered viral genomes (e.g. CheckV >=50% completeness) and need to dereplicate them into species-level vOTUs and/or test their novelty against existing virome catalogues.

## When NOT to use

- Input viral genomes have not been quality-filtered (no CheckV completeness assessment); clustering unfiltered contigs inflates vOTU counts with fragments
- The goal is prokaryotic species dereplication at 95% ANI over whole genomes (SGB clustering), not viral OTU definition
- Only a handful of viral genomes are present; all-vs-all BLASTn clustering and database overlap statistics are not meaningful at that scale

## Inputs

- Quality-filtered viral genome FASTA files (VMGC: 14,224 genomes with >=50% CheckV completeness from VMGC_virus.tar.gz)
- Reference virome databases for cross-comparison (Gut Virome Database, Gut Phage Database, Metagenomic Gut Virus catalogue, Oral Virus Database, NCBI RefSeq viral genomes)
- virusClstr.sh clustering script from github.com/RChGO/VMGC

## Outputs

- vOTU clusters with representative sequences (VMGC: 4,263 vOTUs from 14,224 genomes)
- Per-vOTU database comparison results (votu_overlap_results.csv)
- Summary non-overlap percentage (VMGC: 85.8% of vOTUs absent from all five external virome databases)

## How to apply

Retrieve the quality-filtered viral genomes (VMGC: 14,224 sequences at >=50% completeness by CheckV, from VMGC_virus.tar.gz on Zenodo record 10457006) and run an all-vs-all nucleotide comparison with BLASTn, clustering the genomes with the VMGC virusClstr.sh script at 95% nucleotide identity over 85% of the genome length to define vOTUs (VMGC yields 4,263). Optionally extract protein-coding genes from vOTU representatives with Prodigal in metagenomic mode for downstream annotation of the clustered set. To assess novelty, compare vOTU representative sequences via BLASTn against external databases (Gut Virome Database, Gut Phage Database, Metagenomic Gut Virus catalogue, Oral Virus Database, NCBI RefSeq viral genomes) and count vOTUs with no significant match in any database. The rationale is that 95%/85% is a de facto species-level threshold for viruses, and database cross-referencing quantifies how much of the diversity is unexplored (VMGC reported 85.8% absent from all five databases, Fig. 4b).

## Related tools

- **BLAST** (All-vs-all nucleotide comparison of viral genomes for clustering, and BLASTn of vOTU representatives against external virome databases)
- **CheckV** (Quality filter upstream of clustering: retains viral genomes with >=50% completeness (10.3% complete, 29.3% high quality, 60.4% medium quality))
- **Prodigal** (Extracts protein-coding genes from vOTU representatives in metagenomic mode for downstream annotation of the clustered viral set)
- **virusClstr.sh** (VMGC pipeline script that clusters viral genomes into vOTUs at 95% identity over 85% of the genome) — https://github.com/RChGO/VMGC/blob/main/Pipelines/virusClstr.sh
- **GNU parallel 20201122** (Parallelizes the all-vs-all comparison and database search steps) — https://github.com/RChGO/VMGC/tree/main/Pipelines
- **NCBI RefSeq** (One of the five external virome databases BLASTn-searched to quantify vOTU novelty)

## Examples

```
find viral_genomes/*.fasta | parallel -j 10 blastn -query {} -db vOTU_all_vs_all -outfmt 6 -perc_identity 95 -qcovhsp 85 > vOTU_hits.tsv; bash virusClstr.sh vOTU_hits.tsv viral_genomes/ 4263_vOTUs/
```

## Evaluation signals

- Clustering of the 14,224 quality-filtered VMGC viral genomes at 95% identity / 85% genome coverage yields exactly 4,263 vOTUs
- BLASTn of vOTU representatives against the five databases gives a non-overlap fraction of 85.8% (matching Fig. 4b within a small tolerance)
- Per-vOTU comparison results written to votu_overlap_results.csv cover all 4,263 representatives with a hit/no-hit status per database
- All clustered genomes passed the upstream CheckV filter of at least 50% completeness
- Representative FASTA sequences exist for every vOTU and are the only sequences used in the downstream database comparison

## Limitations

- Many VMGC phages are unknown and their functional exploration based on existing databases remains limited
- The 85.8% non-overlap estimate depends on which external databases are queried; unexplored vaginal viral diversity means absence of a hit is not proof of novelty
- Potential contamination from experimental operations or kits in the public vaginal metagenomic datasets underlying the genomes can propagate into vOTUs

## Evidence

- [abstract] The result revealed that 85.8% of vOTUs in the VMGC were not found in any of the other virome databases: "The result revealed that 85.8% of vOTUs in the VMGC were not found in any of the other virome databases"
- [methods] vOTU clustering threshold: [evidence span withheld — anchor status: not_found]
- [methods] Prodigal gene extraction from representatives: [evidence span withheld — anchor status: not_found]
- [abstract] viral genome filter: "resulting in 14,224 viral sequences achieving at least 50% completeness"
- [readme] data deposit: "Full viral genomes (n=14,224) and vOTUs (n=4,263)| 197 MB | VMGC_virus.tar.gz"
