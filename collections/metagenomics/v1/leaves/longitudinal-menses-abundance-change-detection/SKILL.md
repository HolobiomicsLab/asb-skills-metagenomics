---
name: longitudinal-menses-abundance-change-detection
description: Use when you have a longitudinal 16S OTU table (e.g. a vaginal time course)
  with per-sample metadata marking menses, and want to test whether PICRUSt-predicted
  KEGG module abundances change significantly during menses versus other cycle phases.
when_to_use_negative:
- Data are WGS shotgun metagenomes already annotated to KOs — direct functional profiling
  (e.g. HUMAnN) is appropriate rather than PICRUSt prediction
- Samples are from static body-site comparisons (e.g. supragingival vs subgingival
  plaque) rather than a longitudinal series with menses annotations — use a between-group
  module enrichment test instead
- The 16S data are not closed-reference OTU picked against a PICRUSt-compatible reference
  (Greengenes+IMG), since predicted genome contents assume matching tips on the marker
  reference tree
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: PICRUSt
  role: Normalizes the OTU table by 16S copy number predictions and predicts sample
    KO abundances used for the module-level menses-vs-rest abundance tests
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Produces the PICRUSt-compatible closed-reference OTU table via pick_reference_otus_through_otu_table.py
- name: uclust
  role: OTU picking against Greengenes+IMG during closed-reference OTU picking
- name: STAMP v2.0
  role: Statistical testing and visualization of module abundance differences between
    conditions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:3dacc0bd440206b68fe77a02b116022eebf1fdd7d14283c2d80af7b7cc0345cb
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# longitudinal-menses-abundance-change-detection

## Summary

Detects mean-abundance changes of predicted KEGG functional modules across a longitudinal 16S time course by contrasting menses time points against the rest of the cycle, as applied to 993 vaginal samples with PICRUSt-predicted metagenomes.

## When to use

Use when you have a longitudinal 16S OTU table (e.g. a vaginal time course) with per-sample metadata marking menses, and want to test whether PICRUSt-predicted KEGG module abundances change significantly during menses versus other cycle phases.

## When NOT to use

- Data are WGS shotgun metagenomes already annotated to KOs — direct functional profiling (e.g. HUMAnN) is appropriate rather than PICRUSt prediction
- Samples are from static body-site comparisons (e.g. supragingival vs subgingival plaque) rather than a longitudinal series with menses annotations — use a between-group module enrichment test instead
- The 16S data are not closed-reference OTU picked against a PICRUSt-compatible reference (Greengenes+IMG), since predicted genome contents assume matching tips on the marker reference tree

## Inputs

- Closed-reference OTU table (uclust vs Greengenes+IMG, QIIME 1.5.0-dev) for a longitudinal sample series (e.g. 993 vaginal time course samples)
- Per-sample time-point metadata marking menses time points vs the rest of the cycle
- PICRUSt pre-calculated 16S copy number and KO genome-content tables shipped with the PICRUSt package

## Outputs

- KEGG module-level abundance table collapsed from predicted KO abundances
- Per-sample NSTI values (one per microbial community sample)
- Test statistics, p-values, and rank orderings of module mean-abundance changes (menses vs rest of cycle), e.g. written to pathway_enrichment_results.tsv

## How to apply

First obtain closed-reference OTU tables picked against Greengenes+IMG with uclust in QIIME 1.5.0-dev (pick_reference_otus_through_otu_table.py) for PICRUSt compatibility. Run PICRUSt: normalize the OTU table by 16S copy number predictions, then multiply normalized OTU abundance by each KO abundance in the genome and sum KOs per sample, recording a single NSTI value per sample. Collapse predicted KO abundances to KEGG module level (e.g. M00240 iron complex transport system). For each module, test mean-abundance changes across the time course by contrasting menses time points against the rest of the cycle, and rank modules by mean-abundance increase. In the HMP vaginal analysis, the M00240 iron complex transport system showed the largest significant mean-abundance increase during menses.

## Related tools

- **PICRUSt** (Normalizes the OTU table by 16S copy number predictions and predicts sample KO abundances used for the module-level menses-vs-rest abundance tests) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Produces the PICRUSt-compatible closed-reference OTU table via pick_reference_otus_through_otu_table.py)
- **uclust** (OTU picking against Greengenes+IMG during closed-reference OTU picking)
- **STAMP v2.0** (Statistical testing and visualization of module abundance differences between conditions)

## Evaluation signals

- Each sample carries a single NSTI value; flag or filter samples with high NSTI (poor expected prediction accuracy)
- The KO-to-module collapse must retain the modules of interest (e.g. M00240 iron complex transport system) before testing
- Statistical outputs include test statistics, p-values, and rank orderings; the expected pattern is that M00240 shows the largest significant mean-abundance increase during menses
- Compare predicted-metagenome-derived trends against the article's reproduced findings to confirm consistency (e.g. magnitude and direction of change for iron transport modules)

## Limitations

- PICRUSt does not infer viral or eukaryotic metagenome components; only bacterial and archaeal 16S sequences are included
- PICRUSt cannot distinguish strain-level variation when the marker gene sequence is identical among strains
- Some gene gain/loss or laterally transferred genes will be poorly predicted, with wide confidence intervals (e.g. Supplemental Fig. 7)
- Prediction accuracy depends on availability of related reference genomes, quantified by NSTI

## Evidence
