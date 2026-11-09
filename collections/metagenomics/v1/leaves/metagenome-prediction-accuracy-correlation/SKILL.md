---
name: metagenome-prediction-accuracy-correlation
description: 'Use when evaluating trustworthiness of PICRUSt metagenome predictions
  on 16S-only communities: compute per-sample NSTI on a marker gene tree and correlate
  it with prediction accuracy to detect accuracy decay across host-associated and
  environmental habitats.'
when_to_use_negative:
- 'The community contains only viral or eukaryotic taxa: PICRUSt only includes bacterial
  and archaeal 16S marker gene sequences, so NSTI-based confidence does not apply'
- OTU table is de novo or open-reference picked rather than closed-reference against
  Greengenes+IMG, so OTUs cannot be placed on the marker reference tree for NSTI calculation
- Marker gene sequences are identical among strains and strain-level functional variation
  is the question of interest, which PICRUSt cannot distinguish
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: PICRUSt
  role: Computes per-sample NSTI, performs 16S copy-number normalization, ancestral
    state reconstruction and exponentially distance-weighted gene content prediction
    used to obtain accuracy
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Constructs PICRUSt-compatible closed-reference OTU tables via pick_reference_otus_through_otu_table.py
- name: uclust
  role: Reference OTU picking against Greengenes+IMG during closed-reference OTU table
    construction
- name: HUMAnN v0.98
  role: Annotates WGS metagenome reads to KOs, providing the observed KO abundances
    against which PICRUSt prediction accuracy is measured
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:6f64e9e4ab782358e5e9850c857a89cbae82f653e0b8350a5016a74bad2c607f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HUMAnN v0.98
  - PICRUSt
  - QIIME 1.5.0-dev
  - uclust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metagenome-prediction-accuracy-correlation

## Summary

Quantifies how PICRUSt's 16S-based metagenome prediction accuracy decays with phylogenetic distance to the nearest sequenced reference genome, by correlating per-sample NSTI values with prediction accuracy (Spearman r=-0.4 globally). It grounds confidence in PICRUSt predictions in a measurable, per-sample proxy.

## When to use

Use when evaluating trustworthiness of PICRUSt metagenome predictions on 16S-only communities: compute per-sample NSTI on a marker gene tree and correlate it with prediction accuracy to detect accuracy decay across host-associated and environmental habitats.

## When NOT to use

- The community contains only viral or eukaryotic taxa: PICRUSt only includes bacterial and archaeal 16S marker gene sequences, so NSTI-based confidence does not apply
- OTU table is de novo or open-reference picked rather than closed-reference against Greengenes+IMG, so OTUs cannot be placed on the marker reference tree for NSTI calculation
- Marker gene sequences are identical among strains and strain-level functional variation is the question of interest, which PICRUSt cannot distinguish

## Inputs

- Closed-reference-picked 16S OTU table (QIIME 1.5.0-dev pick_reference_otus_through_otu_table.py with uclust against Greengenes+IMG)
- 16S marker gene phylogenetic tree containing tips with known data (complete reference genomes) and unknown tips
- PICRUSt predicted KO abundance tables per sample (normalized OTU table x KO genome content)
- Sequenced metagenome KO abundances per validation sample (WGS reads annotated to KOs, e.g. with HUMAnN v0.98)

## Outputs

- Per-sample NSTI values (abundance-weighted sum of nearest-sequenced-taxon distances)
- Global and per-habitat Spearman correlation coefficients between NSTI and prediction accuracy (e.g. nsti_accuracy_correlation.csv)
- Per-sample PICRUSt prediction accuracy (agreement of predicted vs observed KO abundances)

## How to apply

Build closed-reference 16S OTU tables (QIIME 1.5.0-dev pick_reference_otus_through_otu_table.py with uclust against Greengenes+IMG) for each validation community. For every OTU, compute the phylogenetic distance to its nearest sequenced reference genome on the 16S marker gene tree, weight by OTU abundance, and sum the scores into a single per-sample NSTI value, as PICRUSt does. Generate PICRUSt metagenome predictions (16S copy-number normalization of the OTU table, ancestral state reconstruction on the pruned tree, exponentially distance-weighted gene content prediction) and compare predicted KO abundances against sequenced metagenomes annotated to KOs (e.g. via HUMAnN v0.98) to obtain per-sample accuracy. Finally compute Spearman rank correlations between per-sample NSTI and accuracy globally (expected r=-0.4, p<0.001) and within each habitat, and write per-sample NSTI values plus global and per-habitat coefficients to a CSV such as nsti_accuracy_correlation.csv.

## Related tools

- **PICRUSt** (Computes per-sample NSTI, performs 16S copy-number normalization, ancestral state reconstruction and exponentially distance-weighted gene content prediction used to obtain accuracy) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Constructs PICRUSt-compatible closed-reference OTU tables via pick_reference_otus_through_otu_table.py)
- **uclust** (Reference OTU picking against Greengenes+IMG during closed-reference OTU table construction)
- **HUMAnN v0.98** (Annotates WGS metagenome reads to KOs, providing the observed KO abundances against which PICRUSt prediction accuracy is measured)

## Examples

```
pick_reference_otus_through_otu_table.py -i seqs.fna -r gg_img_ref_seqs.fna -t gg_img_otu_taxonomy.txt -p pick_params.txt
```

## Evaluation signals

- Per-sample NSTI equals the sum of abundance-weighted nearest-sequenced-taxon OTU scores on the 16S marker tree (single value per community sample)
- Global Spearman correlation between per-sample NSTI and prediction accuracy should be negative (expected r=-0.4, p<0.001)
- Within-habitat Spearman correlations should fall in the reported range (-0.25 to -0.82, p<0.05; workflow cites soil r=0.81, mammal gut restricted to NSTI<0.05 r=0.72, hypersaline mat r=0.25)
- All OTU tables must be PICRUSt-compatible (closed-reference against Greengenes+IMG) and normalized by 16S copy-number predictions before metagenome prediction
- Output CSV contains per-sample NSTI values plus both global and per-habitat correlation coefficients

## Limitations

- PICRUSt only infers bacterial and archaeal components; viral and eukaryotic metagenome components are not predicted
- The reference database has phylogenetic gaps for some environments; additional genome sequences are needed to fill phylogenetic gaps for accurate prediction
- Some gene gains, losses or laterally transferred genes will be poorly predicted (broad confidence intervals); accuracy depends on NSTI and cannot distinguish strain-level variation when marker gene sequences are identical

## Evidence
