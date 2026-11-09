---
name: pathway-algal-cover-correlation-analysis
description: Use when only 16S rRNA data are available along a continuous environmental
  gradient (e.g., algal cover in coral mucus) and functional shifts must be inferred
  by correlating predicted KEGG pathway abundances against that variable.
when_to_use_negative:
- Shotgun metagenomic data are already available — direct functional profiling (e.g.,
  HUMAnN KO annotation) is preferable to prediction
- The community's taxa lack closely related reference genomes (high NSTI), so pathway
  predictions carry wide confidence intervals and correlation-based interpretation
  is unreliable
- The question concerns viral or eukaryotic metagenome components, which PICRUSt does
  not infer since only bacterial and archaeal 16S marker gene sequences are included
tools:
- name: PICRUSt
  role: Predicts gene family (KO) abundances from the 16S OTU table via copy-number
    normalization and ancestral state reconstruction; reports per-sample weighted
    NSTI
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Constructs the PICRUSt-compatible OTU table via closed-reference OTU picking
    (pick_reference_otus_through_otu_table.py)
- name: uclust
  role: Clustering engine used for closed-reference OTU picking against Greengenes+IMG
- name: STAMP v2.0
  role: Used to compare pathway abundances between high- and low-algal-cover groups
    and compute Spearman correlations against algal cover
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:5216477a81329dbe118f3e77bac2fcd6e30d7eb970a022ff452e8d9d51e121d8
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pathway-algal-cover-correlation-analysis

## Summary

Correlates PICRUSt-predicted KEGG pathway abundances (collapsed from KO predictions) against a continuous environmental gradient — relative algal cover in coral mucus samples — to detect biologically meaningful functional shifts from 16S rRNA data alone.

## When to use

Use when only 16S rRNA data are available along a continuous environmental gradient (e.g., algal cover in coral mucus) and functional shifts must be inferred by correlating predicted KEGG pathway abundances against that variable.

## When NOT to use

- Shotgun metagenomic data are already available — direct functional profiling (e.g., HUMAnN KO annotation) is preferable to prediction
- The community's taxa lack closely related reference genomes (high NSTI), so pathway predictions carry wide confidence intervals and correlation-based interpretation is unreliable
- The question concerns viral or eukaryotic metagenome components, which PICRUSt does not infer since only bacterial and archaeal 16S marker gene sequences are included

## Inputs

- 16S rRNA OTU table for coral mucus samples (335 samples, Study ID 1854 from microbio.me/QIIME database)
- Closed-reference OTU table picked against Greengenes+IMG (PICRUSt-compatible format)
- Algal cover measurements per sample (continuous gradient variable)
- PICRUSt pre-calculated predicted genome contents for all tips in the marker reference tree

## Outputs

- PICRUSt-predicted metagenome table of KO abundances per sample (e.g., coral_mucus_predicted_metagenomes.biom)
- Per-sample weighted NSTI values (expected ~0.12 ± 0.02 s.d. for coral mucus data)
- KEGG pathway-level abundance table collapsed from predicted KOs
- Spearman correlation statistics of pathway abundances vs. algal cover (e.g., secretion systems r=1.0, ribosomal biogenesis r=0.90, galactose metabolism r=-1.0, ascorbate/alderate metabolism r=-0.90)

## How to apply

Construct a PICRUSt-compatible OTU table by closed-reference OTU picking (QIIME 1.5.0-dev, pick_reference_otus_through_otu_table.py with uclust against Greengenes+IMG) from the 335 coral mucus 16S samples (Study ID 1854, microbio.me). Normalize the OTU table by predicted 16S copy number, then predict metagenomes by multiplying each normalized OTU abundance by the KO abundances of its predicted genome — gene contents of tips without reference genomes weighted exponentially by the reciprocal of phylogenetic distance via ancestral state reconstruction — and summing KO abundances per sample. Record the per-sample weighted NSTI (expected ~0.12 ± 0.02 s.d. for the coral data) to confirm prediction feasibility. Collapse predicted KOs into KEGG pathways and compare high- vs low-algal-cover samples (secretion system genes at 46% higher relative abundance in high-cover plots) and compute Spearman correlations of pathway abundances against algal cover using STAMP v2.0.

## Related tools

- **PICRUSt** (Predicts gene family (KO) abundances from the 16S OTU table via copy-number normalization and ancestral state reconstruction; reports per-sample weighted NSTI) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Constructs the PICRUSt-compatible OTU table via closed-reference OTU picking (pick_reference_otus_through_otu_table.py))
- **uclust** (Clustering engine used for closed-reference OTU picking against Greengenes+IMG)
- **STAMP v2.0** (Used to compare pathway abundances between high- and low-algal-cover groups and compute Spearman correlations against algal cover)

## Examples

```
pick_reference_otus_through_otu_table.py -i coral_mucus_16s_seqs.fna -r gg_img_13_5_otus.fasta -o picrust_otu_table.biom
```

## Evaluation signals

- Weighted NSTI equals ~0.12 (± 0.02 s.d.) for the coral mucus data, confirming prediction feasibility despite fewer reference genomes
- Spearman correlation of secretion system pathway abundance vs. algal cover reproduces r=1.0 (p=0.0)
- Ribosomal biogenesis correlation reproduces r=0.90; galactose metabolism r=-1.0; ascorbate/alderate metabolism r=-0.90
- Secretion system genes show 46% higher relative abundance in high- vs. low-algal-cover plots
- Predicted metagenome table is written as a valid BIOM file (coral_mucus_predicted_metagenomes.biom)

## Limitations

- PICRUSt does not infer viral or eukaryotic metagenome components — only bacterial and archaeal marker gene sequences are included
- The software cannot distinguish variation at the strain level if the marker gene sequence used is identical among strains
- Some instances of gene gain, loss, or laterally transferred genes will be poorly predicted, with broad confidence intervals
- Accuracy depends on availability of related reference genomes; phylogenetic gaps in the reference database require additional genome sequences for accurate prediction, quantified via NSTI

## Evidence
