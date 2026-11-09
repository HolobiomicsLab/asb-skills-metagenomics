---
name: metagenomic-subspecies-stratification
description: Use when sample-level gene content profiles mapped to VIRGO (summary.Count.txt
  restricted via 1.taxon.tbl.txt) are available and the goal is to partition samples
  into within-species MG-subspecies clusters.
when_to_use_negative:
- When the question is strain-level SNP or nucleotide-variant resolution — VIRGO does
  not yet support identifying nucleotide variants within a gene
- When species-specific gene content is too sparse (samples with few mapped genes
  for the target species), making Jaccard distance on presence/absence unstable
- When a reference gene catalog for the body site does not exist and de novo assembly-based
  strain analysis is required instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: VIRGO
  role: Non-redundant vaginal gene catalog providing mapping database, taxonomy table
    (1.taxon.tbl.txt), and per-sample gene count summaries used to build species-specific
    gene content profiles
  repo: github.com/ravel-lab/VIRGO
- name: bowtie
  role: Read mapping engine inside VIRGO's runMapping.step1.sh; -l seed length (default
    25) controls mapping strictness for calling gene hits
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:d87d6a2d15b8ef8fae6265722b41801c40a0e5d4ba018a0576049d956f83ac8d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - VIRGO
  - bowtie
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metagenomic-subspecies-stratification

## Summary

Resolves metagenomic subspecies (MG-subspecies) within a bacterial species by hierarchically clustering samples on species-specific gene content (presence/absence) profiles, using Jaccard distance and ward linkage on VIRGO-mapped vaginal metagenomes.

## When to use

Use when sample-level gene content profiles mapped to VIRGO (summary.Count.txt restricted via 1.taxon.tbl.txt) are available and the goal is to partition samples into within-species MG-subspecies clusters.

## When NOT to use

- When the question is strain-level SNP or nucleotide-variant resolution — VIRGO does not yet support identifying nucleotide variants within a gene
- When species-specific gene content is too sparse (samples with few mapped genes for the target species), making Jaccard distance on presence/absence unstable
- When a reference gene catalog for the body site does not exist and de novo assembly-based strain analysis is required instead

## Inputs

- Per-sample VIRGO mapping outputs: summary.Abundance.txt, summary.Count.txt, summary.Percentage.txt for 1,507 vaginal metagenomes
- VIRGO taxonomy annotation table 1_VIRGO/1.taxon.tbl.txt (gene-to-species assignments)
- Raw metagenomic FASTQ reads (single-end or merged paired-end) for mapping to VIRGO

## Outputs

- Samples x genes binarized presence/absence matrix per target species (e.g., L. gasseri, G. vaginalis)
- Pairwise Jaccard similarity/distance matrix between samples
- Ward-linkage hierarchical clustering dendrogram with MG-subspecies cluster labels per sample
- Cluster membership table (cluster label for every mapped sample of each species)

## How to apply

First map reads from each of the metagenomes (e.g., 1,507 vaginal metagenomes) to the VIRGO non-redundant gene catalog with VIRGO's runMapping scripts (bowtie-based, default seed length -l 25, default read cutoff of 2 reads to call a gene hit), producing summary.Abundance.txt, summary.Count.txt, and summary.Percentage.txt. Restrict genes to the target species by intersecting gene IDs with the VIRGO taxonomy table (1_VIRGO/1.taxon.tbl.txt), selecting genes assigned to e.g. Lactobacillus gasseri or Gardnerella vaginalis. Binarize the per-sample gene read counts into presence/absence calls to build a samples x genes matrix per species, then compute pairwise Jaccard similarity between samples based on shared present genes. Apply ward-linkage hierarchical clustering on the Jaccard distances to partition samples into MG-subspecies clusters, reporting cluster membership per sample; the expected resolution is at least three MG-subspecies for L. gasseri and more than four for G. vaginalis, concordant with previously identified clades.

## Related tools

- **VIRGO** (Non-redundant vaginal gene catalog providing mapping database, taxonomy table (1.taxon.tbl.txt), and per-sample gene count summaries used to build species-specific gene content profiles) — github.com/ravel-lab/VIRGO
- **bowtie** (Read mapping engine inside VIRGO's runMapping.step1.sh; -l seed length (default 25) controls mapping strictness for calling gene hits) — github.com/ravel-lab/VIRGO

## Examples

```
for sample in *.fq; do ./runMapping.step1.sh -r $sample -p $sample -d /path/to/VIRGO/; done; ./runMapping.step2.sh -p ./step1_out -d /path/to/VIRGO/
```

## Evaluation signals

- Cluster assignment table contains a cluster label for every mapped sample of each species
- Number of distinct L. gasseri MG-subspecies clusters is >= 3
- Number of distinct G. vaginalis MG-subspecies clusters is > 4
- G. vaginalis clusters are concordant with previously identified clades within that species
- Gene hit calls respect the default read cutoff (2 reads per gene) and bowtie seed length (-l 25) mapping parameters

## Limitations

- MG-subspecies reflect gene content only; additional studies are needed to characterize differences between metagenomic subspecies and their possible effect on host health
- Observed metagenomic subspecies contain non-redundant genes not observed in isolate genome sequences for that species, complicating direct comparison with reference clades
- Gaps in subspecies resolution may reflect limited isolate genomes or systematic bias in growth and recovery of vaginal species; targeted isolation may be needed
- VIRGO is a reference-based approach; its database is primarily focused on bacteria with limited viral and fungal gene sequences

## Evidence

- [results] Hierarchical clustering of species-specific gene content profiles revealed at least three distinct metagenomic subspecies for L. gasseri and more than four MG-subspecies for G. vaginalis: [evidence span withheld — anchor status: not_found]
- [abstract] identify a high degree of intraspecies diversity within and across vaginal microbiota: "we analyze 1,507 additional vaginal metagenomes, and identify a high degree of intraspecies diversity within and across vaginal microbiota"
- [readme] summary.Count.txt: the number of genes per species: "summary.Count.txt: the number of genes per species"
- [readme] the cutoff of the number of reads for calling a gene hit: "in step 1 scripts, the cutoff of the number of reads for calling a gene hit."
- [discussion] additional studies are needed to characterize differences between metagenomic subspecies: "However, additional studies are needed to characterize differences between metagenomic subspecies and to detail their possible effect on host health."
