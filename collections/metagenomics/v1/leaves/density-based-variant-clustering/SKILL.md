---
name: density-based-variant-clustering
description: Use when cellsnp-lite AD/DP pileups are available for ≥10 shotgun metagenomic
  samples and you must separate shared informative variants (num_of_clusters >= 2)
  from non-polymorphic background sites.
when_to_use_negative:
- 'Single-sample datasets: a minSample of 5% of total sample size and the ''shared
  informative variant'' concept are undefined for one sample'
- 'Single-cell mitochondrial variant calling: the README directs mtDNA/clonal-substructure
  work to the parent tool MQuad with vireoSNP, not MetaQuad'
- Sites already excluded by the cellsnp-lite pileup thresholds (minor allele frequency
  < 0.02 or count < 100) — clustering cannot recover variants filtered at this stage
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MetaQuad
  role: Runs the density-based BIC clustering on the cellSNP folder to label variants
    as informative (num_of_clusters >= 2) or background
  repo: https://github.com/holab-hku/MetaQuad
- name: cellsnp-lite
  role: Piles up AD/DP allele counts per sample for candidate variants with minMAF
    = 0.02 and minCOUNT = 100, producing the input folder for clustering
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: MQuad
  role: Parent single-cell binomial-mixture tool that MetaQuad extends for shotgun
    metagenomic data
  repo: https://github.com/single-cell-genetics/MQuad
- name: bwa
  role: Aligns shotgun metagenomic reads to the reference database upstream of pileup
    (with samtools, producing .filtered.sorted.bam)
- name: bowtie2
  role: Alternative aligner for producing the .filtered.sorted.bam inputs
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:4884bc99a9b7c7ea46066202085f0a572a932481ad85d66dbfef25c9a4116e6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad
  tools:
  - MQuad
  - MetaQuad
  - Python
  - bowtie2
  - bwa
  - cellsnp-lite
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5789473684210527
n_claims: 19
---

# density-based-variant-clustering

## Summary

Density-based BIC clustering (via MetaQuad, an extension of MQuad) that classifies each variant pileup from shotgun metagenomic samples as either a shared informative variant or background noise, enabling strain-level/population structure analysis.

## When to use

Use when cellsnp-lite AD/DP pileups are available for ≥10 shotgun metagenomic samples and you must separate shared informative variants (num_of_clusters >= 2) from non-polymorphic background sites.

## When NOT to use

- Single-sample datasets: a minSample of 5% of total sample size and the 'shared informative variant' concept are undefined for one sample
- Single-cell mitochondrial variant calling: the README directs mtDNA/clonal-substructure work to the parent tool MQuad with vireoSNP, not MetaQuad
- Sites already excluded by the cellsnp-lite pileup thresholds (minor allele frequency < 0.02 or count < 100) — clustering cannot recover variants filtered at this stage

## Inputs

- .filtered.sorted.bam files of shotgun metagenomic samples aligned to a reference database (bwa or bowtie2)
- cellSNP-lite output folder (sparse AD/DP matrices and cellSNP.cells.vcf.gz / cellSNP.cells_ARG.vcf.gz) produced with minMAF = 0.02 and minCOUNT = 100
- minSample threshold (5% of total sample size, integer)

## Outputs

- BIC_params.csv with per-variant statistics (variant_names, num_of_clusters, mean_DP, mean_AD, number_DP, deltaBIC)
- List of shared informative variants defined as num_of_clusters >= 2 (e.g. 64682 of 67241 BIO-ML variants; 7591 variants across 529 ARGs)

## How to apply

First pile up variants across all samples with cellsnp-lite using minMAF = 0.02 and minCOUNT = 100 (a higher value reduces downstream computation time), writing the cellSNP output folder (e.g. cellSNP.cells_ARG.vcf.gz). Then run MetaQuad on that folder with a minSample threshold set to 5% of the total sample size (e.g. --minSample=3 for 30 BIO-ML samples, 6 for the 121-sample H. pylori ARG dataset, 50 for n=1000 simulations); MetaQuad fits its density-based binomial-mixture/BIC model per variant. Read BIC_params.csv and retain only variants with num_of_clusters >= 2 as shared informative mutations, optionally applying the reported quality thresholds (mean AD 0.1, mean DP 0.5). Variants collapsing into a single cluster are treated as non-polymorphic background.

## Related tools

- **MetaQuad** (Runs the density-based BIC clustering on the cellSNP folder to label variants as informative (num_of_clusters >= 2) or background) — https://github.com/holab-hku/MetaQuad
- **cellsnp-lite** (Piles up AD/DP allele counts per sample for candidate variants with minMAF = 0.02 and minCOUNT = 100, producing the input folder for clustering) — https://github.com/single-cell-genetics/cellsnp-lite
- **MQuad** (Parent single-cell binomial-mixture tool that MetaQuad extends for shotgun metagenomic data) — https://github.com/single-cell-genetics/MQuad
- **bwa** (Aligns shotgun metagenomic reads to the reference database upstream of pileup (with samtools, producing .filtered.sorted.bam))
- **bowtie2** (Alternative aligner for producing the .filtered.sorted.bam inputs)

## Examples

```
metaquad -i cellSNP -o Metaquad --minSample=6
```

## Evaluation signals

- BIC_params.csv exists, is parseable, and contains a num_of_clusters column; all retained variants have num_of_clusters >= 2
- On the BIO-ML dataset (30 samples, PRJNA544527), cellsnp detects 67,241 variants and 64,682 have num_of_clusters >= 2 with minSample=3
- On the 121-sample H. pylori ARG dataset (minSample=6, mean AD 0.1, mean DP 0.5), the callset reproduces 7591 informative variants across 529 ARGs in ~60 minutes
- On the n=1000 simulation at mean depth 11.8 with minSample=50, the calls recover 7598 of 7638 true positives with zero false positives
- Compared against simulated ground truth, the number of false positive SNPs is markedly reduced relative to state-of-the-art methods without greatly affecting the true positive rate

## Limitations

- Higher cellsnp-lite minMAF/minCOUNT values are recommended to reduce computation time for subsequent clustering steps, at some sensitivity cost
- minSample must be scaled to the cohort (5% of total sample size); a fixed value mis-sizes the 'shared' requirement across datasets of 30, 121, or 1000 samples
- Runtime grows with sample count — the 121-sample ARG run already took ~60 minutes, so very large cohorts need scaled thresholds (e.g. minSample=50 at n=1000)

## Evidence

- [abstract] MetaQuad utilizes a density-based clustering technique to effectively distinguish between shared variants and non-polymorphic sites using shotgun metagenomic data.: "MetaQuad utilizes a density-based clustering technique to effectively distinguish between shared variants and non-polymorphic sites using shotgun metagenomic data."
- [intro] recommended MetaQuad parameters: "Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2."
- [intro] recommended cellsnp-lite pileup parameters: "Pile up variants using [cellsnp-lite](https://github.com/single-cell-genetics/cellsnp-lite). <br/> with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100."
- [methods] canonical MetaQuad invocation: "metaquad  -i cellSNP -o Metaquad --minSample=5"
