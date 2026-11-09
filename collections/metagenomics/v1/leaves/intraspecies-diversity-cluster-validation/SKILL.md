---
name: intraspecies-diversity-cluster-validation
description: Use when VIRGO mapping outputs (summary.Count.txt, summary.Abundance.txt)
  are available for vaginal metagenomes and the goal is to confirm that ward-linkage
  clustering of Jaccard distances over species-assigned gene content recovers the
  reported number of MG-subspecies.
when_to_use_negative:
- The goal is taxonomic profiling of community composition rather than resolving strain-level
  gene-content structure — VIRGO's summary.Percentage.txt species abundances already
  serve that
- Input samples were not mapped to the VIRGO gene catalog (e.g. 16S amplicon data,
  which the article states cannot capture accessory-gene-content diversity)
- The target species has too few mapped samples or genes to support stable Jaccard/ward
  clustering, so cluster counts would not be comparable to the reported MG-subspecies
  structure
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: VIRGO
  role: Non-redundant vaginal gene catalog and toolkit whose taxonomy table (1.taxon.tbl.txt)
    and mapping summaries provide the species-specific gene content used for clustering
  repo: github.com/ravel-lab/VIRGO
- name: bowtie
  role: Short-read mapper used in VIRGO's runMapping.step1.sh to align metagenome
    reads to the VIRGO bowtie database (seed length -l 25 by default)
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:2334574c0d9e778aa85ecce9399b2bcf2cc55773575df2bb7bc486cf50e4a67d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - VIRGO
  - bowtie
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# intraspecies-diversity-cluster-validation

## Summary

Validates that hierarchical clustering of species-specific gene presence/absence profiles from VIRGO-mapped vaginal metagenomes resolves metagenomic subspecies (MG-subspecies) at the expected diversity: >=3 MG-subspecies for Lactobacillus gasseri and >4 for Gardnerella vaginalis across the 1,507 vaginal metagenomes.

## When to use

Use when VIRGO mapping outputs (summary.Count.txt, summary.Abundance.txt) are available for vaginal metagenomes and the goal is to confirm that ward-linkage clustering of Jaccard distances over species-assigned gene content recovers the reported number of MG-subspecies.

## When NOT to use

- The goal is taxonomic profiling of community composition rather than resolving strain-level gene-content structure — VIRGO's summary.Percentage.txt species abundances already serve that
- Input samples were not mapped to the VIRGO gene catalog (e.g. 16S amplicon data, which the article states cannot capture accessory-gene-content diversity)
- The target species has too few mapped samples or genes to support stable Jaccard/ward clustering, so cluster counts would not be comparable to the reported MG-subspecies structure

## Inputs

- summary.Abundance.txt, summary.Count.txt, summary.Percentage.txt from VIRGO runMapping.step2.sh over the 1,507 vaginal metagenomes
- 1_VIRGO/1.taxon.tbl.txt (VIRGO taxonomy table linking VIRGO gene IDs to species)
- per-sample mapping output files (sampleName.out: VIRGO gene ID, mapped read count, gene length)

## Outputs

- Per-species samples x genes presence/absence gene-content matrices for L. gasseri and G. vaginalis
- Pairwise Jaccard distance matrices per species
- Cluster assignment table with one MG-subspecies label per mapped sample per species
- Validation report showing >=3 L. gasseri and >4 G. vaginalis MG-subspecies

## How to apply

Map all 1,507 metagenomes to the VIRGO non-redundant gene catalog (0.95 million genes) with bowtie via runMapping.step1.sh/step2.sh, producing summary.Abundance.txt, summary.Count.txt and summary.Percentage.txt. Using 1_VIRGO/1.taxon.tbl.txt, subset genes assigned to Lactobacillus gasseri or Gardnerella vaginalis and build a samples x genes presence/absence matrix by binarizing per-sample gene read counts (default gene-hit cutoff is 2 reads, bowtie seed length -l 25). Compute pairwise Jaccard distances from shared present genes and apply ward-linkage hierarchical clustering per species. Judge success by schema and threshold checks: every mapped sample must receive a cluster label, L. gasseri must partition into >=3 distinct MG-subspecies and G. vaginalis into >4, and G. vaginalis clusters should be concordant with previously identified clades, reflecting the reported high within- and across-community intraspecies diversity.

## Related tools

- **VIRGO** (Non-redundant vaginal gene catalog and toolkit whose taxonomy table (1.taxon.tbl.txt) and mapping summaries provide the species-specific gene content used for clustering) — github.com/ravel-lab/VIRGO
- **bowtie** (Short-read mapper used in VIRGO's runMapping.step1.sh to align metagenome reads to the VIRGO bowtie database (seed length -l 25 by default)) — github.com/ravel-lab/VIRGO

## Examples

```
./runMapping.step2.sh -p /path/to/step1/out -d /path/to/VIRGO/  # produces summary.Count.txt used to build species-specific presence/absence matrices
```

## Evaluation signals

- Cluster assignment table contains a label for every mapped sample of each species (no missing assignments)
- Number of distinct L. gasseri MG-subspecies clusters is >= 3
- Number of distinct G. vaginalis MG-subspecies clusters is > 4 and concordant with previously identified clades within that species
- Presence/absence matrices are built only from genes annotated to the target species in 1.taxon.tbl.txt, with gene hits called at the default cutoff of 2 reads
- Dendrogram cutting on Jaccard distances yields stable clusters reflecting high intraspecies diversity within and across vaginal microbiota (CSTs)

## Limitations

- VIRGO's reference-based approach means observed MG-subspecies may contain non-redundant genes not observed in isolate genome sequences for that species
- Additional studies are needed to characterize differences between metagenomic subspecies and their possible effect on host health; this skill only resolves cluster structure, not functional consequences
- VIRGO's database is primarily bacterial with limited viral/fungal gene content, so subspecies resolution is restricted to bacterial gene content

## Evidence

- [abstract] intraspecies diversity magnitude: "we analyze 1,507 additional vaginal metagenomes, and identify a high degree of intraspecies diversity within and across vaginal microbiota"
- [discussion] subspecies follow-up needed: "However, additional studies are needed to characterize differences between metagenomic subspecies and to detail their possible effect on host health."
- [readme] taxonomy file for gene subsetting: "1.taxon.table` contains taxonomic information"
- [readme] mapping summary outputs used as clustering inputs: "summary.Count.txt: the number of genes per species"
- [readme] gene hit read cutoff: "the cutoff of the number of reads for calling a gene hit"
