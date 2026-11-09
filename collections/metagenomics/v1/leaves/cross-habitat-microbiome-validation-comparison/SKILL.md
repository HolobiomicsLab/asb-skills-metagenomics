---
name: cross-habitat-microbiome-validation-comparison
description: Use when you have closed-reference 16S OTU tables from multiple habitat
  types (e.g. mammal gut, soil, hypersaline mat) plus matched shotgun metagenomes,
  and need to test whether prediction accuracy decays with phylogenetic distance to
  reference genomes.
when_to_use_negative:
- Your community's NSTI is very high (distant from any sequenced reference genome)
  — the paper shows accuracy decays with NSTI and recommends NSTI be used to qualify
  expected accuracy rather than trust predictions unconditionally
- The community contains viral or eukaryotic components you hope to recover — PICRUSt
  only covers bacterial and archaeal 16S marker gene sequences
- You need strain-level resolution — PICRUSt cannot distinguish variation at the strain
  level when the marker gene sequence is identical among strains
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: PICRUSt
  role: Generates metagenome predictions (copy-number normalization, ancestral state
    reconstruction, exponential distance-weighted gene content prediction) and computes
    per-sample NSTI
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Closed-reference OTU picking (pick_reference_otus_through_otu_table.py) to
    produce PICRUSt-compatible OTU tables
- name: uclust
  role: Reference OTU picker used in closed-reference picking against Greengenes+IMG
- name: HUMAnN v0.98
  role: Annotates WGS metagenome reads to KOs to provide the ground-truth for accuracy
    comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:5c615446a92ec411f0387186c7c1db6c15bc3daa84847459f6f983d234ad5085
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
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cross-habitat-microbiome-validation-comparison

## Summary

Benchmark PICRUSt metagenome-prediction accuracy across host-associated and environmental microbiomes by regressing per-sample prediction accuracy against NSTI, the abundance-weighted phylogenetic distance to the nearest sequenced reference genome on the 16S marker tree.

## When to use

Use when you have closed-reference 16S OTU tables from multiple habitat types (e.g. mammal gut, soil, hypersaline mat) plus matched shotgun metagenomes, and need to test whether prediction accuracy decays with phylogenetic distance to reference genomes.

## When NOT to use

- Your community's NSTI is very high (distant from any sequenced reference genome) — the paper shows accuracy decays with NSTI and recommends NSTI be used to qualify expected accuracy rather than trust predictions unconditionally
- The community contains viral or eukaryotic components you hope to recover — PICRUSt only covers bacterial and archaeal 16S marker gene sequences
- You need strain-level resolution — PICRUSt cannot distinguish variation at the strain level when the marker gene sequence is identical among strains

## Inputs

- Closed-reference-picked 16S OTU tables (QIIME 1.5.0-dev, uclust vs Greengenes+IMG) for mammal gut, soil, and hypersaline mat validation communities
- Sequenced shotgun metagenomes matched to the 16S samples, KO-annotated (e.g. HUMAnN v0.98)
- 16S marker gene tree with reference genome tips and pre-calculated predicted gene contents for unknown tips (Greengenes+IMG, IMG v3.5 KO/COG annotations)
- 16S copy-number predictions for OTUs

## Outputs

- Per-sample NSTI values for each validation community sample
- Per-sample PICRUSt prediction accuracy (predicted vs sequenced KO abundances)
- Global Spearman rank correlation between NSTI and accuracy (expected r=-0.4, p<0.001)
- Per-habitat Spearman correlations (soil r=0.81; mammal gut NSTI<0.05 subset r=0.72; hypersaline mat r=0.25)
- Summary CSV (nsti_accuracy_correlation.csv)

## How to apply

Build PICRUSt-compatible OTU tables with closed-reference OTU picking in QIIME 1.5.0-dev (pick_reference_otus_through_otu_table.py with uclust against Greengenes+IMG) for each validation community (mammal gut, soil, hypersaline mat). For each sample, determine each OTU's phylogenetic distance to its nearest sequenced reference genome on the 16S marker gene tree, weight by OTU abundance, and sum the OTU scores to give a single NSTI value per sample. Generate PICRUSt metagenome predictions — normalizing the OTU table by 16S copy number predictions, applying ancestral state reconstruction, and weighting gene content predictions exponentially by the reciprocal of phylogenetic distance — then compare predicted KO abundances to sequenced metagenomes (HUMAnN v0.98-annotated) to obtain per-sample accuracy. Compute Spearman rank correlations between per-sample NSTI and accuracy globally and within each habitat, expecting a global decay of r=-0.4 (p<0.001) and habitat-specific coefficients (soil r=0.81; mammal gut restricted to NSTI<0.05 r=0.72; hypersaline mat r=0.25). Write per-sample NSTI values and global/per-habitat correlation coefficients to a CSV (nsti_accuracy_correlation.csv) for downstream inspection.

## Related tools

- **PICRUSt** (Generates metagenome predictions (copy-number normalization, ancestral state reconstruction, exponential distance-weighted gene content prediction) and computes per-sample NSTI) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Closed-reference OTU picking (pick_reference_otus_through_otu_table.py) to produce PICRUSt-compatible OTU tables)
- **uclust** (Reference OTU picker used in closed-reference picking against Greengenes+IMG)
- **HUMAnN v0.98** (Annotates WGS metagenome reads to KOs to provide the ground-truth for accuracy comparison)

## Examples

```
pick_reference_otus_through_otu_table.py -i seqs.fna -r greengenes_img_ref_seqs.fna -m uclust -t greengenes_img_ref_tree.tre -o closed_ref_otus/
```

## Evaluation signals

- Global Spearman correlation between per-sample NSTI and prediction accuracy should reproduce the expected decay (r=-0.4, p<0.001)
- Habitat-specific correlations fall in the reported ranges (soil r=0.81; mammal gut restricted to NSTI<0.05 r=0.72; hypersaline mat r=0.25) with p<0.05
- Every sample has a single summed NSTI value (abundance-weighted sum of per-OTU nearest-sequenced-taxon scores), not per-OTU values
- Predicted KO abundances were produced after 16S copy-number normalization of the OTU table (un-normalized predictions inflate error)
- Accuracy is computed by comparing PICRUSt KOs to HUMAnN-annotated sequenced metagenomes for the same samples

## Limitations

- Only 16S marker gene sequences corresponding to bacterial and archaeal genomes are included, so viral and eukaryotic metagenome components are not inferred
- PICRUSt cannot distinguish strain-level variation if the marker gene sequence is identical among strains
- Some gene gains, losses, and laterally transferred genes will be poorly predicted (with broad confidence intervals, e.g. Supplemental Fig. 7)
- The reference database has phylogenetic gaps for some environments; additional genome sequences may be needed to fill gaps for accurate prediction
- No versioned release/changelog for PICRUSt given in the article text, complicating exact reproduction

## Evidence
