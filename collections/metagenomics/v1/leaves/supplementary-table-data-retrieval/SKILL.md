---
name: supplementary-table-data-retrieval
description: Use when a task asks to reproduce a study's reported genome counts or
  per-phylum breakdown, and the source is article tables/abstract rather than raw
  sequence data (e.g. 8 complete vs 789 draft CPR genomes).
when_to_use_negative:
- The task requires re-deriving genome counts from raw metagenomic assemblies (224
  Gb paired-end reads, scaffold binning) rather than retrieving reported values
- The needed phylum assignments are not present in the article tables or supplementary
  data, so numbers would have to be inferred rather than retrieved
- The target output is per-sample abundance or 16S rRNA insertion statistics rather
  than genome yield counts
edam_operation: http://edamontology.org/operation_0306
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:1e214c1967c25b062f54805d9c467c75c6fd2aa835be82a33981f8209df8c268
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools: []
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# supplementary-table-data-retrieval

## Summary

Extract quantitative genome-yield and taxonomy-distribution figures from a CPR study's abstract, Table 1, and Supplementary Table 3, then assemble them into a validated CSV (genome_yields_phylum_distribution.csv) recording 8 complete and 789 draft genomes across >35 phyla.

## When to use

Use when a task asks to reproduce a study's reported genome counts or per-phylum breakdown, and the source is article tables/abstract rather than raw sequence data (e.g. 8 complete vs 789 draft CPR genomes).

## When NOT to use

- The task requires re-deriving genome counts from raw metagenomic assemblies (224 Gb paired-end reads, scaffold binning) rather than retrieving reported values
- The needed phylum assignments are not present in the article tables or supplementary data, so numbers would have to be inferred rather than retrieved
- The target output is per-sample abundance or 16S rRNA insertion statistics rather than genome yield counts

## Inputs

- Article abstract text reporting genome yields (8 complete, 789 draft, >35 phyla)
- Table 1 with genome counts and genome size notes
- Supplementary Table 3 with per-genome phylum assignments for the manually curated complete genomes

## Outputs

- genome_yields_phylum_distribution.csv with columns: genome_category (complete/draft), phylum, count, genome_size_note

## How to apply

Parse the article abstract and Table 1 to pull the reported yields: 8 complete and 789 draft genomes from bacteria representing >35 phyla, noting the characteristic small (<1 Mb) genome size of the complete set. Then extract from Table 1 and Supplementary Table 3 the phylum assignments of the eight manually curated complete genomes: 3 Microgenomates, 2 Parcubacteria, 1 Kazan, 1 Berkelbacteria, and 1 Saccharibacteria. Record each value in a CSV with columns for genome category (complete/draft), phylum, count, and genome size note. Justify the draft-genome category using the study's binning criterion (≥50% of single-copy genes with fewer than 1.125 copies) so the counts reflect the paper's own definitions. Validate by confirming the yield rows show 8 complete and 789 draft, and that the complete-genome phylum counts (3+2+1+1+1) sum to 8.

## Examples

```
python -c "import csv; csv.writer(open('genome_yields_phylum_distribution.csv','w',newline='')).writerows([['complete','Microgenomates',3,'<1 Mb'],['complete','Parcubacteria',2,'<1 Mb'],['complete','Kazan',1,'<1 Mb'],['complete','Berkelbacteria',1,'<1 Mb'],['complete','Saccharibacteria',1,'<1 Mb'],['draft','>35 phyla',789,'']])"
```

## Evaluation signals

- CSV contains a draft row with count 789 and a complete row with count 8
- Complete-genome phylum rows sum to 8: Microgenomates=3, Parcubacteria=2, Kazan=1, Berkelbacteria=1, Saccharibacteria=1
- A genome size note is present for complete genomes reflecting the small (<1 Mb) size characteristic
- CSV schema includes the columns genome_category, phylum, count, and genome_size_note
- Every retrieved count traces to a specific source location (abstract, Table 1, or Supplementary Table 3)

## Limitations

- The per-phylum breakdown of the eight complete genomes is only available from Table 1 / Supplementary Table 3; if those tables are not accessible, the distribution cannot be verified from the abstract alone
- Draft-quality status is defined by the study's binning threshold (≥50% SCGs with <1.125 copies), so counts cannot be compared directly to genomes binned under different completeness criteria
- The reported figures describe bacteria from CPR (>35 phyla, Rifle, Colorado aquifer groundwater) and do not generalize to other sample types

## Evidence
