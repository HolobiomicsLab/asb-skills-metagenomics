---
name: nsti-accuracy-assessment
description: Use when predicting metagenomes from 16S rRNA OTU tables with PICRUSt
  and you must quantify per-sample prediction reliability from reference-genome coverage,
  especially for host-associated or environmental communities with sparse reference
  genomes.
when_to_use_negative:
- Input is already shotgun WGS metagenome data (annotated with HUMAnN) — functional
  potential can be measured directly rather than predicted and its uncertainty estimated
  via NSTI
- The community is dominated by viral or eukaryotic members — PICRUSt only includes
  bacterial and archaeal 16S marker gene sequences and does not infer viral or eukaryotic
  metagenome components
- Strain-level functional resolution is required — PICRUSt cannot distinguish variation
  at the strain level if the marker gene sequence is identical among strains, so NSTI
  does not capture within-species functional divergence
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: PICRUSt
  role: Computes the weighted NSTI per sample during metagenome prediction from the
    copy-number-normalized OTU table
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Generates the PICRUSt-compatible closed-reference OTU table (with uclust)
    from which NSTI is derived
- name: STAMP v2.0
  role: Used to test whether pathways with acceptable NSTI (e.g. secretion systems
    in coral mucus) show significant shifts across algal-cover groups
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:e970f010e2f6742d718e9b2cce491793bb2acd194112bda61d3be86f850f8df6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PICRUSt
  - QIIME 1.5.0-dev
  - STAMP
  - STAMP v2.0
  - uclust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# nsti-accuracy-assessment

## Summary

Quantify the expected accuracy of PICRUSt 16S-to-metagenome predictions by computing a per-sample weighted Nearest Sequenced Taxon Index (NSTI), which reflects the phylogenetic proximity of community members to reference genomes in IMG/Greengenes. It is the article's recommended gate for judging whether predicted functional shifts (e.g. secretion-system genes correlated with coral mucus algal cover) are trustworthy.

## When to use

Use when predicting metagenomes from 16S rRNA OTU tables with PICRUSt and you must quantify per-sample prediction reliability from reference-genome coverage, especially for host-associated or environmental communities with sparse reference genomes.

## When NOT to use

- Input is already shotgun WGS metagenome data (annotated with HUMAnN) — functional potential can be measured directly rather than predicted and its uncertainty estimated via NSTI
- The community is dominated by viral or eukaryotic members — PICRUSt only includes bacterial and archaeal 16S marker gene sequences and does not infer viral or eukaryotic metagenome components
- Strain-level functional resolution is required — PICRUSt cannot distinguish variation at the strain level if the marker gene sequence is identical among strains, so NSTI does not capture within-species functional divergence

## Inputs

- PICRUSt-compatible closed-reference OTU table (BIOM) built from 16S data, e.g. the 335 coral mucus samples (Study ID 1854) picked against Greengenes+IMG with uclust
- Copy-number-normalized OTU table produced by PICRUSt's 16S copy number normalization step
- Reference marker gene tree with known (IMG reference genome) and unknown tips

## Outputs

- Per-sample weighted NSTI values (one NSTI per microbial community sample), e.g. mean 0.12 +/- 0.02 s.d. for the coral mucus data
- Predicted metagenome table of KO/KEGG pathway abundances per sample, written to coral_mucus_predicted_metagenomes.biom
- A reliability verdict (prediction feasible vs. confidence intervals too wide) used to qualify downstream pathway-correlation findings

## How to apply

Build a PICRUSt-compatible OTU table via closed-reference OTU picking (pick_reference_otus_through_otu_table.py in QIIME 1.5.0-dev with uclust) against Greengenes+IMG, then normalize by predicted 16S copy number. During metagenome prediction, gene contents of tips without reference genomes are weighted exponentially by the reciprocal of phylogenetic distance via ancestral state reconstruction; each OTU's NSTI score is summed to give a single NSTI value per microbial community sample. Inspect the per-sample weighted NSTI rather than only the mean: the coral mucus data (335 samples, Study ID 1854) yielded a weighted NSTI of 0.12 (+/- 0.02 s.d.), confirming prediction remained feasible despite fewer reference genomes, and validated the resulting Spearman correlations of predicted KEGG pathways (secretion systems r=1.0, ribosomal biogenesis r=0.90, galactose metabolism r=-1.0) against algal cover computed in STAMP v2.0. High NSTI signals phylogenetic 'gaps' in the reference database; the prescribed remedy is collecting additional genome sequences to fill those gaps, or reporting the uncertainty alongside the predicted metagenome.

## Related tools

- **PICRUSt** (Computes the weighted NSTI per sample during metagenome prediction from the copy-number-normalized OTU table) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Generates the PICRUSt-compatible closed-reference OTU table (with uclust) from which NSTI is derived)
- **STAMP v2.0** (Used to test whether pathways with acceptable NSTI (e.g. secretion systems in coral mucus) show significant shifts across algal-cover groups)

## Examples

```
predict_metagenomes.py -i otu_table.biom -a 16S_copy_number.biom -o coral_mucus_predicted_metagenomes.biom
```

## Evaluation signals

- Per-sample weighted NSTI for the coral data reproduces the reported 0.12 (+/- 0.02 s.d.), confirming prediction feasibility
- Exactly one NSTI value is emitted per microbial community sample, computed after 16S copy-number normalization of the OTU table
- The OTU table is PICRUSt-compatible (closed-reference against Greengenes+IMG); NSTI computed on a de novo or open-reference table is invalid
- Pathways retained for interpretation (e.g. secretion systems, r=1.0 Spearman vs. algal cover) are reported alongside their NSTI so uncertainty is quantified
- High-NSTI samples correspond to predictions with wide confidence intervals, flagged rather than silently interpreted

## Limitations

- Some instances of gene gain, loss, or laterally transferred elements will be poorly predicted, with broad confidence intervals as a result
- PICRUSt accuracy depends on the availability of related reference genomes; the reference database has phylogenetic gaps for some environments, requiring additional genome sequences
- The current system only includes bacterial and archaeal 16S marker gene sequences and does not infer viral or eukaryotic metagenome components
- PICRUSt cannot distinguish strain-level variation when the marker gene sequence is identical among strains
- No versioned release or changelog information for PICRUSt was provided in the text, complicating reproducibility reporting

## Evidence
