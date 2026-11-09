---
name: species-specific-gene-presence-absence-profiling
description: Use when metagenomic reads have been mapped to the VIRGO gene catalog
  and the goal is to quantify intraspecies diversity — e.g. resolving MG-subspecies
  of L. gasseri or G. vaginalis from per-sample gene content rather than 16S or abundance
  profiles.
when_to_use_negative:
- When the question is taxonomic composition or relative abundance of communities
  — summary.Percentage.txt species-level abundance already answers that without gene-content
  binarization
- When 16S rRNA amplicon data are the only input — the article states 16S cannot infer
  complete function because accessory-gene content drives strain diversity
- When species of interest lack enough mapped samples/genes in the sample to form
  stable Jaccard clusters (e.g. low-abundance taxa)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: VIRGO
  role: Provides the non-redundant gene catalog, taxonomy table (1.taxon.tbl.txt),
    and per-sample mapping summary files from which species-specific gene content
    matrices are built
  repo: https://github.com/ravel-lab/VIRGO
- name: bowtie
  role: Reads from each metagenome are mapped to the VIRGO Bowtie database with seed
    length -l 25 and reads cutoff 2 to produce per-gene counts feeding the presence/absence
    calls
  repo: https://github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:76a39eb240b1f2a8ef217da2740a6066376074214a779ac16c5471ecf0743093
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ravel-lab/VIRGO
  tools:
  - VIRGO
  - bowtie
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# species-specific-gene-presence-absence-profiling

## Summary

Resolves metagenomic subspecies (MG-subspecies) within a bacterial species by restricting the VIRGO non-redundant gene catalog to species-assigned genes, binarizing per-sample gene read counts into presence/absence, and clustering samples by Jaccard distance with ward linkage to recover intraspecies gene-content diversity across the 1,507 vaginal metagenomes.

## When to use

Use when metagenomic reads have been mapped to the VIRGO gene catalog and the goal is to quantify intraspecies diversity — e.g. resolving MG-subspecies of L. gasseri or G. vaginalis from per-sample gene content rather than 16S or abundance profiles.

## When NOT to use

- When the question is taxonomic composition or relative abundance of communities — summary.Percentage.txt species-level abundance already answers that without gene-content binarization
- When 16S rRNA amplicon data are the only input — the article states 16S cannot infer complete function because accessory-gene content drives strain diversity
- When species of interest lack enough mapped samples/genes in the sample to form stable Jaccard clusters (e.g. low-abundance taxa)

## Inputs

- Per-sample mapping outputs from VIRGO step-2 summarization: summary.Abundance.txt, summary.Count.txt, summary.Percentage.txt, summary.NR.abundance.txt for 1,507 vaginal metagenomes
- VIRGO taxonomy table 1_VIRGO/1.taxon.tbl.txt (gene-to-species assignments, e.g. Cluster_219398 V1000057 Lactobacillus_iners 990)

## Outputs

- Species-restricted samples x genes presence/absence (binarized) matrix for each target species
- Pairwise Jaccard similarity/distance matrix between samples per species
- Hierarchical (ward) clustering assignment table with one MG-subspecies cluster label per mapped sample
- Cluster membership counts (>=3 MG-subspecies for L. gasseri; >4 for G. vaginalis)

## How to apply

Start from the per-sample mapping outputs produced by runMapping.step2.sh against the VIRGO non-redundant gene catalog (summary.Abundance.txt, summary.Count.txt, summary.Percentage.txt, summary.NR.abundance.txt). Subset genes to the target species using the VIRGO taxonomy table (1_VIRGO/1.taxon.tbl.txt), selecting genes assigned to Lactobacillus gasseri or Gardnerella vaginalis, and be mindful of the default 'reads = 2' cutoff for calling a gene hit in step-1 scripts. Binarize the samples x genes count matrix into presence/absence calls, compute pairwise Jaccard similarity between samples from shared present genes, and apply ward-linkage hierarchical clustering on the Jaccard distances to partition samples into MG-subspecies clusters per species. Report cluster membership per sample and validate that the L. gasseri clustering yields at least three distinct MG-subspecies and G. vaginalis more than four, consistent with previously identified clades; the rationale is that accessory-gene content, not 16S rRNA profiles, captures strain-level diversity within and across vaginal microbiota.

## Related tools

- **VIRGO** (Provides the non-redundant gene catalog, taxonomy table (1.taxon.tbl.txt), and per-sample mapping summary files from which species-specific gene content matrices are built) — https://github.com/ravel-lab/VIRGO
- **bowtie** (Reads from each metagenome are mapped to the VIRGO Bowtie database with seed length -l 25 and reads cutoff 2 to produce per-gene counts feeding the presence/absence calls) — https://github.com/ravel-lab/VIRGO

## Examples

```
awk -F"\t" '$3=="Lactobacillus_gasseri" {print $2}' 1_VIRGO/1.taxon.tbl.txt > gasseri_genes.lst
```

## Evaluation signals

- Cluster assignment table contains a cluster label for every mapped sample of each species
- Number of distinct L. gasseri MG-subspecies clusters is >= 3
- Number of distinct G. vaginalis MG-subspecies clusters is > 4, concordant with previously identified clades within the species
- Each binarized matrix restricted via 1.taxon.tbl.txt contains only genes assigned to the target species
- Intraspecies diversity is observed both within and across vaginal microbiota community state types, not driven by a single sample group

## Limitations

- Observed metagenomic subspecies contain non-redundant genes not observed in isolate genome sequences for that species, so cluster gene content may exceed what reference isolates explain
- Reference-based approach: VIRGO is focused on gene-level de-redundancy of the vaginal microbiome with primarily bacterial coverage and limited viral/fungal gene sequences
- Additional studies are needed to characterize functional differences between metagenomic subspecies and their effect on host health

## Evidence

- [results] mg-subspecies count finding: [evidence span withheld — anchor status: not_found]
- [abstract] datasets analyzed: "we analyze 1,507 additional vaginal metagenomes, and identify a high degree of intraspecies diversity within and across vaginal microbiota"
- [readme] mapping outputs documented in README: "summary.Abundance.txt: the number of reads per species
- summary.Count.txt: the number of genes per species
- summary.Percentage.txt: normalized abundance in percentage per species"
- [readme] taxonomy table structure: "1.taxon.tbl.txt :Cluster_219398 V1000057  Lactobacillus_iners 990"
- [methods] mapping cutoff parameters: ""reads** = " default value is 2. : *in step 1 scripts, the cutoff of the number of reads for calling a gene hit.**"
- [discussion] subspecies limitations: "additional studies are needed to characterize differences between metagenomic subspecies and to detail their possible effect on host health"
