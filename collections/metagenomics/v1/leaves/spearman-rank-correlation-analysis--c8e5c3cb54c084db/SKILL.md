---
name: spearman-rank-correlation-analysis
description: Use when per-sample phylogenetic novelty scores (e.g., NSTI from a 16S
  marker gene tree) are paired with per-sample prediction accuracy and a monotonic
  rank-correlation trend must be quantified, globally and per habitat.
when_to_use_negative:
- The community contains viral or eukaryotic components — only bacterial and archaeal
  16S marker gene sequences are covered, so NSTI-based accuracy decay does not apply
  to those fractions
- Samples fall in phylogenetic 'gaps' far from any reference genome — accuracy will
  be poor with broad confidence intervals and the rank-correlation trend should be
  interpreted with those uncertainty bounds
- Strain-level resolution is required — the marker gene cannot distinguish variation
  at the strain level if the 16S sequence is identical among strains, biasing NSTI
  downward
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: PICRUSt
  role: Produces the metagenome predictions and per-sample NSTI scores whose relationship
    is quantified by the Spearman correlation
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Closed-reference OTU picking (pick_reference_otus_through_otu_table.py) that
    yields the OTU tables feeding NSTI and accuracy computation
- name: uclust
  role: OTU picker used against Greengenes+IMG during closed-reference OTU picking
- name: HUMAnN v0.98
  role: Annotates WGS metagenome reads to KOs to provide the ground-truth accuracy
    targets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:587b75a1cc588190c7effaf978a8e3b1751199f07cc88ce6c3268b521ac6de70
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# spearman-rank-correlation-analysis

## Summary

Computes Spearman rank correlations between per-sample NSTI (Nearest Sequenced Taxon Index) values and PICRUSt prediction accuracy to quantify whether metagenome prediction error increases with phylogenetic distance to the nearest sequenced reference genome, globally and within each microbiome type.

## When to use

Use when per-sample phylogenetic novelty scores (e.g., NSTI from a 16S marker gene tree) are paired with per-sample prediction accuracy and a monotonic rank-correlation trend must be quantified, globally and per habitat.

## When NOT to use

- The community contains viral or eukaryotic components — only bacterial and archaeal 16S marker gene sequences are covered, so NSTI-based accuracy decay does not apply to those fractions
- Samples fall in phylogenetic 'gaps' far from any reference genome — accuracy will be poor with broad confidence intervals and the rank-correlation trend should be interpreted with those uncertainty bounds
- Strain-level resolution is required — the marker gene cannot distinguish variation at the strain level if the 16S sequence is identical among strains, biasing NSTI downward

## Inputs

- Closed-reference-picked 16S OTU tables (QIIME 1.5.0-dev, uclust vs. Greengenes+IMG) for mammal gut, soil, and hypersaline mat validation communities
- 16S marker gene reference tree with tips for known reference genomes and unknown tips
- Sequenced shotgun metagenomes annotated to KOs (HUMAnN v0.98) for the validation communities
- PICRUSt metagenome predictions per sample

## Outputs

- Per-sample NSTI values (one abundance-weighted NSTI per microbial community sample)
- Global Spearman correlation between per-sample NSTI and prediction accuracy (expected r=-0.4, p<0.001)
- Per-habitat Spearman correlations (soil r=0.81; mammal gut NSTI<0.05 r=0.72; hypersaline mat r=0.25)
- nsti_accuracy_correlation.csv containing per-sample NSTI values and global/per-habitat correlation coefficients

## How to apply

Start from closed-reference-picked 16S OTU tables built with QIIME 1.5.0-dev (pick_reference_otus_through_otu_table.py, uclust against Greengenes+IMG) for each validation community (mammal gut, soil, hypersaline mat). For each OTU, determine the phylogenetic distance to the nearest sequenced reference genome on the 16S marker gene tree, weight by OTU abundance, and sum the OTU scores to give a single NSTI value per microbial community sample. Generate PICRUSt metagenome predictions (16S copy-number normalization, ancestral state reconstruction, exponential distance-weighted gene content prediction) and compare predicted KO abundances (annotated with HUMAnN v0.98) to sequenced metagenomes to obtain per-sample accuracy. Compute Spearman rank correlations between per-sample NSTI and prediction accuracy globally across all validation communities (expected r=-0.4, p<0.001) and within each habitat (soil r=0.81; mammal gut restricted to NSTI<0.05 r=0.72; hypersaline mat r=0.25, p<0.05). Write per-sample NSTI values plus global and per-habitat correlation coefficients to nsti_accuracy_correlation.csv.

## Related tools

- **PICRUSt** (Produces the metagenome predictions and per-sample NSTI scores whose relationship is quantified by the Spearman correlation) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Closed-reference OTU picking (pick_reference_otus_through_otu_table.py) that yields the OTU tables feeding NSTI and accuracy computation)
- **uclust** (OTU picker used against Greengenes+IMG during closed-reference OTU picking)
- **HUMAnN v0.98** (Annotates WGS metagenome reads to KOs to provide the ground-truth accuracy targets)

## Examples

```
python -c "from scipy.stats import spearmanr; r, p = spearmanr(nsti_values, accuracy); print(r, p)"
```

## Evaluation signals

- Global Spearman correlation between per-sample NSTI and prediction accuracy is negative and significant (expected r=-0.4, p<0.001)
- Within-habitat correlations are all significant at p<0.05 and fall in the reported range r=-0.25 to -0.82 (soil 0.81, mammal gut 0.72 within NSTI<0.05, hypersaline mat 0.25)
- Every sample has a single non-negative abundance-weighted NSTI derived by summing per-OTU nearest-sequenced-taxon scores
- Output file nsti_accuracy_correlation.csv contains per-sample NSTI values plus global and per-habitat correlation coefficients
- Accuracy is measured by comparing predicted KO abundances to HUMAnN-annotated (v0.98) true metagenomes, not to taxonomy-level proxies

## Limitations

- Some instances of gene gain, loss, or laterally transferred genes will be poorly predicted, with broad confidence intervals limiting correlation interpretability
- The system does not infer viral or eukaryotic metagenome components, so NSTI only reflects bacterial/archaeal members
- PICRUSt cannot distinguish variation at the strain level if the marker gene sequence is identical among strains; NSTI may understate true phylogenetic distance
- Reference database coverage varies across environments; additional genome sequences can be needed to fill phylogenetic 'gaps' before accuracy can be trusted

## Evidence

- [discussion] Accuracy depends on NSTI: [evidence span withheld — anchor status: not_found]
