---
name: kegg-module-brite-functional-category-mapping
description: Use when predicted or observed gene family annotations (KOs, COGs) must
  be aggregated into higher-level functional categories via KEGG Modules or BRITE,
  or when per-category accuracy of gene-content predictions is required.
when_to_use_negative:
- Input is raw 16S OTU abundance data that has not yet been converted to predicted
  gene family content via PICRUSt (normalize by 16S copy number and predict KOs first)
- The analysis targets viral or eukaryotic metagenome components, since PICRUSt only
  includes bacterial and archaeal genomes
- Strain-level functional variation is the question of interest, since predictions
  cannot distinguish strains with identical marker gene sequences
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3174
tools:
- name: PICRUSt
  role: generates the predicted KO gene content for held-out genomes whose categories
    are then mapped via KEGG Modules/BRITE
  repo: github.com/picrust/picrust
- name: APE R library
  role: ancestral state reconstruction (ACE ML, ACE REML, ACE PIC) used to produce
    predicted gene content in holdout validation
- name: COUNT package
  role: Wagner Parsimony ancestral state reconstruction alternative used in the genome
    holdout workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:0a2940044369e97bddbbdce1f8648bba0961a3bbb26fe4cc846ece997e099e80
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - APE R library
  - COUNT package
  - PICRUSt
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# kegg-module-brite-functional-category-mapping

## Summary

Maps predicted and observed KO/COG gene content through KEGG Modules and the KEGG BRITE functional hierarchy to higher-level functional categories, enabling per-category comparison of predicted versus actual gene content (e.g., genome holdout accuracy by functional category).

## When to use

Use when predicted or observed gene family annotations (KOs, COGs) must be aggregated into higher-level functional categories via KEGG Modules or BRITE, or when per-category accuracy of gene-content predictions is required.

## When NOT to use

- Input is raw 16S OTU abundance data that has not yet been converted to predicted gene family content via PICRUSt (normalize by 16S copy number and predict KOs first)
- The analysis targets viral or eukaryotic metagenome components, since PICRUSt only includes bacterial and archaeal genomes
- Strain-level functional variation is the question of interest, since predictions cannot distinguish strains with identical marker gene sequences

## Inputs

- KO (KEGG Ortholog) annotation table, e.g. gene-by-genome table from IMG v3.5
- COG annotations from the same reference source
- Predicted gene family (KO) content per genome or OTU from PICRUSt gene-content prediction
- Observed/held-out genome annotations for comparison
- KEGG Modules and BRITE functional hierarchy mappings

## Outputs

- Per-category accuracy table mapping functional categories (via KEGG Modules/BRITE) to mean accuracy and standard deviation
- Per-genome accuracy table aggregated per taxon and per functional category
- CSV file, e.g. genome_holdout_accuracies.csv

## How to apply

Start with gene-by-genome or gene-by-sample KO (KEGG Ortholog) and COG annotation tables, such as the IMG v3.5 gene-by-genome table used with PICRUSt. Map each KO through KEGG Modules and the BRITE hierarchy to higher-level functional categories (e.g., genetic information processing, environmental information processing). For accuracy assessment, compare predicted KO/COG content (e.g., from leave-one-genome-out holdout prediction with ancestral state reconstruction and phylogenetic-distance-weighted averaging) against the actual held-out annotations within each category, then compute per-category accuracy means and standard deviations. Expect core 'housekeeping' categories such as genetic information processing to score higher (mean accuracy ~0.99 +/- 0.03) than variable, laterally transferred categories such as environmental information processing (~0.95 +/- 0.04). Write the resulting per-category (and per-genome) accuracy table to a CSV file for downstream comparison across taxa and categories.

## Related tools

- **PICRUSt** (generates the predicted KO gene content for held-out genomes whose categories are then mapped via KEGG Modules/BRITE) — github.com/picrust/picrust
- **APE R library** (ancestral state reconstruction (ACE ML, ACE REML, ACE PIC) used to produce predicted gene content in holdout validation)
- **COUNT package** (Wagner Parsimony ancestral state reconstruction alternative used in the genome holdout workflow)

## Examples

```
python scripts/map_ko_categories.py --ko_table img_v35_ko_by_genome.txt --hierarchy kegg_modules_brite.txt --accuracy_table genome_holdout_accuracies.csv
```

## Evaluation signals

- Per-genome accuracy for archaea should fall near mean 0.94 +/- 0.04 (n=103) and for bacteria near 0.95 +/- 0.05 (n=2,487) on IMG v3.5 holdout
- 'Genetic information processing' (housekeeping) categories should show higher accuracy (~0.99 +/- 0.03) than 'environmental information processing' (~0.95 +/- 0.04)
- Every predicted and observed KO must resolve to a category through the KEGG Modules/BRITE mapping before per-category aggregation
- Per-genome accuracy values must be bounded in [0,1] and per-category means must be reproducible when the leave-one-genome-out exclusion is re-run
- Output CSV must contain both per-genome rows and per-category/per-taxon aggregate rows matching the reported means

## Limitations

- Laterally transferred genes and variable gene gain/loss events are poorly predicted, producing wide confidence intervals and lower accuracy in categories like environmental information processing
- Accuracy depends on availability of related reference genomes; NSTI should be used to quantify expected prediction accuracy
- Reference database phylogenetic gaps for some environments require additional genomes for accurate prediction

## Evidence

- [other] Map predicted and observed KOs through KEGG Modules and the BRITE hierarchy to higher-level functional categories and compute per-category accuracies.: [evidence span withheld — anchor status: not_found]
- [other] Holdout validation workflow context from source card task_004.: [evidence span withheld — anchor status: not_found]
- [discussion] Accuracy varies by functional category: housekeeping vs laterally transferred.: [evidence span withheld — anchor status: not_found]
