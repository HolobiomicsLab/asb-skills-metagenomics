---
name: oral-niche-differential-abundance-testing
description: Use when comparing predicted functional (KO/KEGG module) abundances between
  body sites or physiological states from 16S-only data, e.g. supragingival vs subgingival
  plaque or menses vs rest-of-cycle vaginal samples.
when_to_use_negative:
- Data are already shotgun metagenomes annotated to KOs (e.g. via HUMaN); direct KO
  comparison, not PICRUSt prediction, is appropriate
- Communities contain substantial viral or eukaryotic components, or strains share
  an identical marker gene sequence — PICRUSt cannot infer these or resolve strain-level
  variation
- Samples have high NSTI values with wide confidence intervals, indicating too few
  related reference genomes for reliable prediction
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: PICRUSt
  role: Predicts KO abundance metagenomes from 16S OTU tables by copy-number normalization
    and ancestral-state reconstruction
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Closed-reference OTU picking (pick_reference_otus_through_otu_table.py) to
    build PICRUSt-compatible OTU tables
- name: uclust
  role: OTU picking engine against the Greengenes+IMG reference used in the closed-reference
    step
- name: STAMP v2.0
  role: Welch's t-test with Bonferroni correction for KEGG module abundance comparisons
    between niches
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:045a93b6b0e5139c2691fbb04160b95fb4731210a1fc0a73b7806a75a6b68a29
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# oral-niche-differential-abundance-testing

## Summary

Differential testing of PICRUSt-predicted KEGG module abundances between oral/vaginal niches: 16S OTU tables are converted to predicted metagenomes, collapsed to KEGG modules (e.g. M00009 citrate cycle, M00240 iron complex transport), and compared with Welch's t-test plus Bonferroni correction in STAMP.

## When to use

Use when comparing predicted functional (KO/KEGG module) abundances between body sites or physiological states from 16S-only data, e.g. supragingival vs subgingival plaque or menses vs rest-of-cycle vaginal samples.

## When NOT to use

- Data are already shotgun metagenomes annotated to KOs (e.g. via HUMaN); direct KO comparison, not PICRUSt prediction, is appropriate
- Communities contain substantial viral or eukaryotic components, or strains share an identical marker gene sequence — PICRUSt cannot infer these or resolve strain-level variation
- Samples have high NSTI values with wide confidence intervals, indicating too few related reference genomes for reliable prediction

## Inputs

- HMP 16S closed-reference OTU table (6,431 samples, hmpdacc.org/HMQCP), Greengenes+IMG reference via uclust in QIIME 1.5.0-dev
- 993-sample vaginal time course OTU table
- Greengenes+IMG reference with pre-calculated predicted genome contents for all tips in the marker reference tree (PICRUSt package file)

## Outputs

- PICRUSt-predicted metagenome table (KO abundances per sample) with one NSTI value per sample
- KEGG module-level collapsed abundance table including M00009 (citrate cycle) and M00240 (iron complex transport system)
- pathway_enrichment_results.tsv with test statistics, p-values, and rank orderings for supragingival-vs-subgingival and menses-vs-rest-of-cycle comparisons

## How to apply

First build PICRUSt-compatible OTU tables by closed-reference OTU picking against Greengenes+IMG using uclust in QIIME 1.5.0-dev (pick_reference_otus_through_otu_table.py). Run PICRUSt on the HMP 16S tables (6,431 samples from hmpdacc.org/HMQCP and the 993-sample vaginal time course): normalize the OTU table by 16S copy number predictions, multiply normalized OTU abundance by each KO abundance and sum per sample, recording a single NSTI value per sample as a confidence filter. Collapse predicted KO abundances to KEGG module level, extracting modules of interest such as M00009 (citrate cycle) and M00240 (iron complex transport system). Compare module abundances between supragingival and subgingival plaque with Welch's t-test and Bonferroni correction in STAMP v2.0, and test mean-abundance changes across the vaginal time course contrasting menses time points against the rest of the cycle. Write all test statistics, p-values, and rank orderings to pathway_enrichment_results.tsv.

## Related tools

- **PICRUSt** (Predicts KO abundance metagenomes from 16S OTU tables by copy-number normalization and ancestral-state reconstruction) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Closed-reference OTU picking (pick_reference_otus_through_otu_table.py) to build PICRUSt-compatible OTU tables)
- **uclust** (OTU picking engine against the Greengenes+IMG reference used in the closed-reference step)
- **STAMP v2.0** (Welch's t-test with Bonferroni correction for KEGG module abundance comparisons between niches)

## Examples

```
pick_reference_otus_through_otu_table.py -i seqs.fna -r greengenes_img_16S.fasta -o closed_ref_otus/
```

## Evaluation signals

- Citrate cycle (M00009) is significantly enriched in supragingival versus subgingival plaque with Bonferroni-corrected Welch's t-test p<1e-10
- M00240 iron complex transport system shows the largest significant mean-abundance increase during menses time points vs the rest of the cycle
- Every sample carries a single recorded NSTI value; samples used for testing should have acceptable NSTI (wide confidence intervals flag unreliable predictions)
- OTU tables were produced by closed-reference picking against Greengenes+IMG — checking the pipeline is pick_reference_otus_through_otu_table.py in QIIME 1.5.0-dev
- pathway_enrichment_results.tsv contains test statistics, p-values, and rank orderings for all tested KEGG modules

## Limitations

- PICRUSt accuracy depends on availability of related reference genomes, quantified by NSTI; genes gained/lost or laterally transferred can be poorly predicted with broad confidence intervals
- Only bacterial and archaeal marker gene sequences are included, so viral and eukaryotic metagenome components are not inferred
- The software cannot distinguish variation at the strain level if the marker gene sequence used is identical among strains

## Evidence
