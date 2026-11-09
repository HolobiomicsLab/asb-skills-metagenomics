---
name: cross-database-virome-overlap-analysis
description: Use when a newly assembled viral genome catalogue (e.g. 14,224 quality-filtered
  genomes >=50% CheckV completeness) must be dereplicated into vOTUs and its novelty
  estimated against existing virome reference databases.
when_to_use_negative:
- Input viral genomes are already dereplicated vOTUs or a published catalogue's representative
  set — reclustering would change the unit of comparison
- The comparator databases are gut/oral-focused catalogues but your study question
  is strain-level overlap rather than database-absence novelty — BLASTn presence/absence
  is too coarse
- Viral contigs have not passed completeness quality filtering (>=50% by CheckV),
  which would inflate the novel-vOTU count with fragments
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
tools:
- name: BLAST
  role: All-vs-all nucleotide comparison of viral genomes for vOTU clustering and
    BLASTn search of vOTU representatives against the five external virome databases
- name: virusClstr.sh
  role: Clustering script that defines vOTUs at 95% identity over 85% of the genome
    from the BLASTn results
  repo: https://github.com/RChGO/VMGC/blob/main/Pipelines/virusClstr.sh
- name: CheckV
  role: Quality assessment upstream of clustering, used to retain viral genomes with
    >=50% completeness
- name: Prodigal
  role: Extraction of protein-coding genes from vOTU representatives in metagenomic
    mode for downstream annotation of the clustered viral set
- name: NCBI RefSeq
  role: Comparator viral genome database used as one of the five references for overlap
    testing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:09e087b751db8e5c7010e29e66ba40b6cadeedd0252132fbdded565ec1bfe099
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - CheckV
  - NCBI RefSeq
  - Prodigal
  - virusClstr.sh
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cross-database-virome-overlap-analysis

## Summary

Quantifies the novelty of a newly assembled viral genome catalogue by clustering its genomes into vOTUs (95% nucleotide identity over 85% of the genome) and cross-referencing vOTU representatives via BLASTn against established virome databases, reporting the fraction absent from all of them.

## When to use

Use when a newly assembled viral genome catalogue (e.g. 14,224 quality-filtered genomes >=50% CheckV completeness) must be dereplicated into vOTUs and its novelty estimated against existing virome reference databases.

## When NOT to use

- Input viral genomes are already dereplicated vOTUs or a published catalogue's representative set — reclustering would change the unit of comparison
- The comparator databases are gut/oral-focused catalogues but your study question is strain-level overlap rather than database-absence novelty — BLASTn presence/absence is too coarse
- Viral contigs have not passed completeness quality filtering (>=50% by CheckV), which would inflate the novel-vOTU count with fragments

## Inputs

- Quality-filtered viral genome FASTA set (e.g. 14,224 genomes with >=50% CheckV completeness from VMGC_virus.tar.gz)
- Virus clustering script (virusClstr.sh from the RChGO/VMGC Pipelines repository)
- Comparator virome databases: Gut Virome Database, Gut Phage Database, Metagenomic Gut Virus catalogue, Oral Virus Database, NCBI RefSeq viral genomes

## Outputs

- vOTU clustering result with representative sequences (4,263 vOTUs for VMGC)
- Per-vOTU cross-database comparison table (votu_overlap_results.csv)
- Summary non-overlap percentage (expected 85.8%)

## How to apply

Retrieve the quality-filtered viral genomes (e.g. VMGC_virus.tar.gz from Zenodo, n=14,224 genomes filtered to >=50% completeness by CheckV) and cluster them into vOTUs using all-vs-all nucleotide BLASTn with virusClstr.sh at 95% identity over 85% of the genome, yielding representative sequences per vOTU (4,263 in the VMGC). Optionally extract protein-coding genes from representatives with Prodigal in metagenomic mode for downstream annotation. Build BLAST databases from each comparator catalogue — Gut Virome Database, Gut Phage Database, Metagenomic Gut Virus catalogue, Oral Virus Database and NCBI RefSeq viral genomes — and search vOTU representatives against all five with BLASTn. Count vOTUs with no significant match in any database and divide by the total vOTU count; the VMGC reported 85.8% (3,657 of 4,263) absent from all five. Write per-vOTU comparison results (e.g. votu_overlap_results.csv) plus a summary non-overlap percentage, and validate by checking that the clustering reproduces the expected vOTU count and the non-overlap fraction matches the published value within tolerance.

## Related tools

- **BLAST** (All-vs-all nucleotide comparison of viral genomes for vOTU clustering and BLASTn search of vOTU representatives against the five external virome databases)
- **virusClstr.sh** (Clustering script that defines vOTUs at 95% identity over 85% of the genome from the BLASTn results) — https://github.com/RChGO/VMGC/blob/main/Pipelines/virusClstr.sh
- **CheckV** (Quality assessment upstream of clustering, used to retain viral genomes with >=50% completeness)
- **Prodigal** (Extraction of protein-coding genes from vOTU representatives in metagenomic mode for downstream annotation of the clustered viral set)
- **NCBI RefSeq** (Comparator viral genome database used as one of the five references for overlap testing)

## Examples

```
blastn -query vOTU_reps.fna -db gut_phage_db -outfmt 6 -out vOTU_vs_GPD.tsv; bash virusClstr.sh  # then aggregate per-database hits into votu_overlap_results.csv
```

## Evaluation signals

- vOTU clustering at 95% identity over 85% genome coverage reproduces the expected 4,263 vOTUs from the 14,224 input genomes
- Computed non-overlap fraction matches the reported 85.8% of vOTUs absent from all five databases within a small tolerance
- Every vOTU appears in the per-database BLASTn results with a presence/absence call for each of the five comparator catalogues
- votu_overlap_results.csv exists with one row per vOTU plus the summary non-overlap percentage
- Input genome count equals 14,224 (the >=50% completeness CheckV-filtered set), confirming no unfiltered sequences leaked into clustering

## Limitations

- Many phages in the VMGC are unknown and their functional exploration based on existing databases remains limited, so BLASTn absence does not imply functional novelty is characterizable
- The microbial sequences captured by the VMGC are incomplete, especially for vaginal samples from non-western populations, so the non-overlap estimate inherits sampling bias
- Potential contamination from experimental operations or kits in the public vaginal metagenomic datasets used to construct the VMGC could affect vOTU calls

## Evidence

- [abstract] 85.8% vOTU non-overlap across five databases: "The result revealed that 85.8% of vOTUs in the VMGC were not found in any of the other virome databases"
- [abstract] Comparator database list: "five large-scale virome databases including Gut Virome Database 40, Gut Phage Database41"
- [abstract] Uncultured viral fraction motivating novelty estimate: "over 25% of prokaryotic species and 85% of viral operational taxonomic units remain uncultured"
- [readme] VMGC viral data deposit used as input: "Full viral genomes (n=14,224) and vOTUs (n=4,263)| 197 MB | VMGC_virus.tar.gz"
