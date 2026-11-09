---
name: kegg-module-abundance-quantification
description: Use when PICRUSt-predicted KO metagenome tables from 16S data must be
  aggregated to KEGG module abundances and statistically compared between sample groups
  (e.g., supragingival vs subgingival plaque, menses vs rest of cycle).
when_to_use_negative:
- Input is raw 16S reads or an unnormalized OTU table that has not yet been processed
  by PICRUSt (no 16S copy-number normalization, no KO prediction, no NSTI values).
- The goal is strain-level or viral/eukaryotic functional inference — PICRUSt only
  covers bacterial and archaeal marker genes and cannot distinguish strains with identical
  marker sequences.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: PICRUSt
  role: Predicts KO-level metagenomes from the normalized OTU table and records per-sample
    NSTI before module collapse
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Produces PICRUSt-compatible closed-reference OTU tables via pick_reference_otus_through_otu_table.py
- name: uclust
  role: Closed-reference OTU picking against Greengenes+IMG for PICRUSt compatibility
- name: STAMP v2.0
  role: Welch's t-test with Bonferroni correction on module abundances between groups;
    figures for enriched modules
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:5f874ec64f6215f2138558c81ee6e84fe9f8f4e9795cf8cd749af89ee0452a36
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PICRUSt
  - QIIME
  - QIIME 1.5.0-dev
  - STAMP
  - STAMP v2.0
  - uclust
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# kegg-module-abundance-quantification

## Summary

Collapses PICRUSt-predicted KEGG Ortholog (KO) abundances to KEGG module level (e.g., M00009 citrate cycle, M00240 iron complex transport) and compares module abundances across body sites or time points using Welch's t-test with Bonferroni correction in STAMP v2.0.

## When to use

Use when PICRUSt-predicted KO metagenome tables from 16S data must be aggregated to KEGG module abundances and statistically compared between sample groups (e.g., supragingival vs subgingival plaque, menses vs rest of cycle).

## When NOT to use

- Input is raw 16S reads or an unnormalized OTU table that has not yet been processed by PICRUSt (no 16S copy-number normalization, no KO prediction, no NSTI values).
- The goal is strain-level or viral/eukaryotic functional inference — PICRUSt only covers bacterial and archaeal marker genes and cannot distinguish strains with identical marker sequences.

## Inputs

- PICRUSt predicted metagenome (KO abundance) table per sample, with per-sample NSTI values
- Closed-reference OTU table from QIIME 1.5.0-dev (Greengenes+IMG reference, uclust), e.g., 6,431 HMP samples from hmpdacc.org/HMQCP or 993 vaginal time course samples
- Sample metadata mapping file defining groups (body site, menses vs rest of cycle)

## Outputs

- KEGG module-level abundance table (e.g., M00009 citrate cycle, M00240 iron complex transport system per sample)
- pathway_enrichment_results.tsv with test statistics, Bonferroni-corrected p-values, and rank orderings

## How to apply

First run PICRUSt on the closed-reference OTU table (picked against Greengenes+IMG with uclust via QIIME 1.5.0-dev pick_reference_otus_through_otu_table.py), normalizing the OTU table by 16S copy number predictions, multiplying normalized OTU abundance by each KO abundance, and summing per sample to obtain predicted KO counts plus one NSTI value per sample. Then collapse the predicted KO abundances to KEGG module level, extracting the modules of interest such as M00009 (citrate cycle) and M00240 (iron complex transport system). Compare module abundances between groups (e.g., supragingival vs subgingival plaque samples) using Welch's t-test with Bonferroni correction in STAMP v2.0, and for time-course data (e.g., 993 vaginal samples) test mean-abundance changes by contrasting menses time points against the rest of the cycle. Write all test statistics, p-values, and rank orderings to an output table such as pathway_enrichment_results.tsv.

## Related tools

- **PICRUSt** (Predicts KO-level metagenomes from the normalized OTU table and records per-sample NSTI before module collapse) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Produces PICRUSt-compatible closed-reference OTU tables via pick_reference_otus_through_otu_table.py)
- **uclust** (Closed-reference OTU picking against Greengenes+IMG for PICRUSt compatibility)
- **STAMP v2.0** (Welch's t-test with Bonferroni correction on module abundances between groups; figures for enriched modules)

## Examples

```
predict_metagenomes.py -i otu_table.biom -o ko_predictions.biom; categorize_by_function.py -i ko_predictions.biom -c ko -l 3 -o pathway_abundances.biom
```

## Evaluation signals

- Citrate cycle module M00009 is significantly enriched in supragingival versus subgingival plaque at p<1e-10 (Welch's t-test with Bonferroni correction).
- M00240 iron complex transport system shows the largest significant mean-abundance increase during menses time points versus the rest of the vaginal cycle.
- Each sample carries a single summed NSTI value computed from all OTU scores; samples with high NSTI should be flagged as low-confidence predictions.
- pathway_enrichment_results.tsv contains test statistics, p-values, and rank orderings for every tested KEGG module.

## Limitations

- PICRUSt accuracy depends on availability of related reference genomes; NSTI should be used to quantify expected prediction accuracy, and some gain/loss or laterally transferred genes will be poorly predicted with wide confidence intervals.
- Only bacterial and archaeal 16S marker gene sequences are included, so viral and eukaryotic metagenome components are not inferred.
- The software cannot distinguish strain-level variation when the marker gene sequence is identical among strains.

## Evidence

- [other] Collapse predicted KO abundances to KEGG module level, extracting the modules of interest including M00009 (citrate cycle) and M00240 (iron complex transport system).: [evidence span withheld — anchor status: not_found]
- [other] Compare module abundances between supragingival and subgingival plaque samples using Welch's t-test with Bonferroni correction in STAMP v2.0.: [evidence span withheld — anchor status: not_found]
