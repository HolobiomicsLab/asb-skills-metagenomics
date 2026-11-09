---
name: 16s-predictive-metagenome-inference
description: Use when input is a PICRUSt-compatible OTU table (BIOM format, closed-reference
  picked against Greengenes+IMG) and the goal is predicting metagenome/gene family
  abundance from marker-gene data alone.
when_to_use_negative:
- The community contains viral or eukaryotic components, since only bacterial and
  archaeal 16S marker gene sequences are included in the reference
- Strain-level functional variation matters and strains share identical marker gene
  sequences, which PICRUSt cannot distinguish
- Reference genomes phylogenetically close to the sampled organisms are lacking (high
  NSTI), indicating unreliable predictions
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: PICRUSt
  role: Performs 16S copy-number normalization, metagenome inference, and NSTI calculation
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Builds the PICRUSt-compatible OTU table via closed-reference OTU picking (pick_reference_otus_through_otu_table.py)
- name: uclust
  role: OTU picking algorithm used for closed-reference picking against Greengenes+IMG
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:8a6dc91e101f618d8d55b2e8f278716072123abdd97925371e80c4918280b4f4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PICRUSt
  - QIIME
  - QIIME 1.5.0-dev
  - STAMP
  - uclust
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# 16s-predictive-metagenome-inference

## Summary

Predicts gene family (KO) abundance metagenomes from 16S rRNA OTU tables using PICRUSt's metagenome inference step: 16S copy-number normalization followed by multiplying normalized OTU abundances by pre-calculated per-taxon gene contents and summing per sample.

## When to use

Use when input is a PICRUSt-compatible OTU table (BIOM format, closed-reference picked against Greengenes+IMG) and the goal is predicting metagenome/gene family abundance from marker-gene data alone.

## When NOT to use

- The community contains viral or eukaryotic components, since only bacterial and archaeal 16S marker gene sequences are included in the reference
- Strain-level functional variation matters and strains share identical marker gene sequences, which PICRUSt cannot distinguish
- Reference genomes phylogenetically close to the sampled organisms are lacking (high NSTI), indicating unreliable predictions

## Inputs

- PICRUSt-compatible OTU table in BIOM format (closed-reference picked against Greengenes+IMG with uclust via QIIME)
- PICRUSt pre-calculated gene family (KO) content file for all tips in the marker reference tree
- 16S copy number prediction table

## Outputs

- predicted_metagenome.biom: annotated predicted gene family (KO) count table
- Per-sample NSTI values summarizing phylogenetic uncertainty
- Downstream KEGG module-level abundance tables (e.g. M00009, M00240) for pathway enrichment testing

## How to apply

First construct a PICRUSt-compatible OTU table via closed-reference OTU picking in QIIME 1.5.0-dev (pick_reference_otus_through_otu_table.py) with uclust against the Greengenes+IMG reference. Normalize the OTU table by predicted 16S copy number so abundances represent estimated organism counts rather than marker gene counts. Load the pre-calculated file supplied with the PICRUSt package containing predicted gene family (KO) abundances for all tips in the marker reference tree, multiply each normalized OTU abundance by the per-taxon KO abundances, and sum KO abundances per sample across all OTUs. Attach a single per-sample NSTI value summarizing phylogenetic uncertainty, and write the result as predicted_metagenome.biom. Downstream, collapse KO abundances to KEGG module level (e.g. M00009 citrate cycle, M00240 iron complex transport) for comparative statistics.

## Related tools

- **PICRUSt** (Performs 16S copy-number normalization, metagenome inference, and NSTI calculation) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Builds the PICRUSt-compatible OTU table via closed-reference OTU picking (pick_reference_otus_through_otu_table.py))
- **uclust** (OTU picking algorithm used for closed-reference picking against Greengenes+IMG)

## Examples

```
normalize_by_copy_number.py -i otu_table.biom -o normalized_otus.biom; predict_metagenomes.py -i normalized_otus.biom -o predicted_metagenome.biom
```

## Evaluation signals

- Output file predicted_metagenome.biom exists and is a valid annotated BIOM table of predicted gene family counts
- Every sample in the output has an associated NSTI value; samples with high NSTI should be flagged as low-confidence
- Normalized OTU abundances reflect organism counts (16S copy-number corrected), not raw marker gene counts
- Recapture of known findings: citrate cycle (M00009) enriched in supragingival vs subgingival plaque (p<1e-10, Welch's t-test with Bonferroni correction in STAMP v2.0), and M00240 iron complex transport showing the largest significant increase during menses
- KO totals per sample equal the sum over OTUs of normalized OTU abundance times per-taxon KO abundance

## Limitations

- Prediction accuracy depends on availability of related reference genomes, quantified by NSTI; phylogenetic gaps in the reference database require additional genomes for accurate prediction
- Some gene gains, losses, and laterally transferred genes will be poorly predicted, with broad confidence intervals as a result
- Only bacterial and archaeal genomes are modeled; viral and eukaryotic metagenome components are not inferred
- Strain-level variation cannot be distinguished when the marker gene sequence is identical among strains

## Evidence
