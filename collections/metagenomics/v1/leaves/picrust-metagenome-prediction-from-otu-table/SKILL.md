---
name: picrust-metagenome-prediction-from-otu-table
description: Use when only 16S marker-gene OTU tables (e.g. QIIME closed-reference
  picked with uclust against Greengenes+IMG) are available and KO-level functional
  profiles are needed without deep shotgun sequencing.
when_to_use_negative:
- Deep shotgun metagenomic data are available (at ~10^5 reads the metagenome reaches
  r=0.82 vs PICRUSt's r=0.18 for the shallow metagenome, and PICRUSt is outperformed
  above ~72,000 sequences per sample)
- Viral or eukaryotic metagenome components must be inferred (only bacterial and archaeal
  16S sequences are included)
- Strain-level functional variation must be resolved when the marker gene sequence
  is identical among strains
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: PICRUSt
  role: Normalizes OTU abundances by predicted 16S copy number and predicts metagenome
    KO abundances via extended ancestral-state reconstruction; reports per-sample
    NSTI
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Constructs PICRUSt-compatible OTU tables via the closed-reference picking
    protocol (pick_reference_otus_through_otu_table.py)
- name: uclust
  role: Reference OTU picker used to assign 16S reads against Greengenes+IMG
- name: HUMAnN v0.98
  role: Annotates WGS reads to KOs to produce the observed metagenome used for benchmarking
    PICRUSt predictions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:fedf1d92a2ebd71cea885b4f6e15b111cfffba232488dd028f1876681d3e2f45
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HUMAnN
  - HUMAnN v0.98
  - PICRUSt
  - QIIME 1.5.0-dev
  - uclust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# picrust-metagenome-prediction-from-otu-table

## Summary

Predicts KO/gene-family abundances of a metagenome from 16S rRNA OTU tables by normalizing OTUs for 16S copy number and combining per-genome KO content via ancestral-state reconstruction, providing an alternative to shallow shotgun sequencing.

## When to use

Use when only 16S marker-gene OTU tables (e.g. QIIME closed-reference picked with uclust against Greengenes+IMG) are available and KO-level functional profiles are needed without deep shotgun sequencing.

## When NOT to use

- Deep shotgun metagenomic data are available (at ~10^5 reads the metagenome reaches r=0.82 vs PICRUSt's r=0.18 for the shallow metagenome, and PICRUSt is outperformed above ~72,000 sequences per sample)
- Viral or eukaryotic metagenome components must be inferred (only bacterial and archaeal 16S sequences are included)
- Strain-level functional variation must be resolved when the marker gene sequence is identical among strains

## Inputs

- Closed-reference OTU table (BIOM) built with QIIME 1.5.0-dev pick_reference_otus_through_otu_table.py against Greengenes+IMG using uclust
- PICRUSt pre-calculated genome-content (KO by OTU) table shipped with the package
- Predicted 16S copy number table for OTUs
- For benchmarking: shotgun metagenome KO abundance table annotated with HUMAnN v0.98

## Outputs

- Copy-number-normalized OTU table
- Predicted metagenome KO abundance table per sample
- Per-sample NSTI values
- For benchmarking: Spearman correlation-vs-depth table (spearman_vs_depth.csv) and plot (spearman_vs_depth.png)

## How to apply

Build a PICRUSt-compatible OTU table with the closed-reference OTU picking protocol in QIIME 1.5.0-dev (pick_reference_otus_through_otu_table.py) against Greengenes+IMG using uclust. PICRUSt then normalizes the OTU table by 16S copy number predictions, and predicts the metagenome by multiplying the normalized OTU abundance by each KO abundance in the genome and summing these KO abundances per sample. Sum OTU scores to give a single NSTI value per sample; NSTI quantifies expected prediction accuracy, so report it alongside predictions. When benchmarking against real sequencing, rarefy both the OTU table and the HUMAnN v0.98-annotated KO table to matched depths, compute Spearman rank correlation per depth, and note that accounting for gene annotation survival (17.3%) and OTU-picking rates (68.9%), PICRUSt may outperform metagenomic sequencing below ~72,000 total sequences per sample.

## Related tools

- **PICRUSt** (Normalizes OTU abundances by predicted 16S copy number and predicts metagenome KO abundances via extended ancestral-state reconstruction; reports per-sample NSTI) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Constructs PICRUSt-compatible OTU tables via the closed-reference picking protocol (pick_reference_otus_through_otu_table.py))
- **uclust** (Reference OTU picker used to assign 16S reads against Greengenes+IMG)
- **HUMAnN v0.98** (Annotates WGS reads to KOs to produce the observed metagenome used for benchmarking PICRUSt predictions)

## Examples

```
pick_reference_otus_through_otu_table.py -i seqs.fna -r gg_img_16S.fasta -t gg_img_taxonomy.txt -m uclust
```

## Evaluation signals

- Per-sample NSTI values are computed and reported; low NSTI indicates closely related reference genomes and expected high accuracy
- In holdout/rarefaction benchmarks, Spearman r between predicted and rarefied observed KO abundances rises with depth, reaching r=0.82 at 10^5 reads (vs r=0.18 for the shallow metagenome)
- Depth-adjusted comparison accounts for 17.3% gene annotation survival and 68.9% OTU-picking rates; the crossover is ~72,000 total sequences per sample (~15,000 annotated metagenomic sequences matching 105 assigned 16S reads)
- Output is a KO-abundance metagenome per sample in a PICRUSt/QIIME-compatible table format derived from the normalized OTU table

## Limitations

- Only bacterial and archaeal genomes are included; the system does not infer viral or eukaryotic metagenome components
- The software cannot distinguish variation at the strain level if the marker gene sequence used is identical among strains
- Some instances of gene gain or loss or laterally transferred genes will be poorly predicted, with wide confidence intervals
- Reference database phylogenetic gaps for some environments require additional genome sequences for accurate prediction

## Evidence
