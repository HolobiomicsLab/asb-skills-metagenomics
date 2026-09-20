---
name: wilcoxon-rank-sum-group-comparison
description: Use when comparing a continuous per-gene metric (e.g., nucleotide diversity)
  between two independent patient groups — such as pre-treatment vs post-antibiotic
  time points — without assuming normality.
when_to_use_negative:
- The input is raw variant calls or BAM files — these must first be converted to per-gene
  nucleotide diversity values via cellsnp-lite, MetaQuad, and nucleotide_diversity_code.R.
- Comparing more than two groups or paired samples — the Wilcoxon rank-sum test as
  applied here is for two independent groups.
- Genes with insufficient coverage where diversity is undefined (e.g., zero covered
  bases in genome_length.csv/cov_bases.csv).
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0625
tools:
- name: MetaQuad
  role: Detects shared informative variants via density clustering; its BIC_params.csv
    output supplies variant clustering parameters feeding the nucleotide diversity
    table that is tested.
  repo: https://github.com/holab-hku/MetaQuad
- name: cellsnp-lite
  role: Piles up variants (cellSNP.cells_ARG.vcf.gz) used to compute per-gene nucleotide
    diversity prior to group testing.
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: samtools
  role: Computes per-sample coverage ('samtools coverage ${i}.sorted.bam > ${i}.tab')
    summarized into genome_length.csv for diversity normalization.
- name: vcfR
  role: R package used to load the cellsnp-lite VCF in nucleotide_diversity_code.R,
    producing the diversity table to which the Wilcoxon test is applied.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:c7fe763c1e97747d4a0ccdc84b7827258ae1b924a4c5d8c66c0cb665f8f067eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad
  tools:
  - MetaQuad
  - cellsnp-lite
  - samtools
  - vcfR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# wilcoxon-rank-sum-group-comparison

## Summary

Non-parametric Wilcoxon rank-sum testing used to identify antibiotic resistance genes whose nucleotide diversity differs significantly between LEVO-treated patient groups (pre-treatment '00' vs 6-weeks-post-treatment '01'), flagging the seven ARGs with P < .05.

## When to use

Use when comparing a continuous per-gene metric (e.g., nucleotide diversity) between two independent patient groups — such as pre-treatment vs post-antibiotic time points — without assuming normality.

## When NOT to use

- The input is raw variant calls or BAM files — these must first be converted to per-gene nucleotide diversity values via cellsnp-lite, MetaQuad, and nucleotide_diversity_code.R.
- Comparing more than two groups or paired samples — the Wilcoxon rank-sum test as applied here is for two independent groups.
- Genes with insufficient coverage where diversity is undefined (e.g., zero covered bases in genome_length.csv/cov_bases.csv).

## Inputs

- Nucleotide_diversity.csv (per-gene, per-sample nucleotide diversity from nucleotide_diversity_code.R)
- Sample metadata mapping patient samples to time-point groups ('00' pre-treatment vs '01' 6-weeks-post-LEVO)
- Optional filter: genes with nucleotide diversity > 0.005
- Upstream: cellSNP.cells_ARG.vcf.gz, BIC_params.csv, genome_length.csv / cov_bases.csv

## Outputs

- Per-gene Wilcoxon rank-sum P values comparing '00' vs '01' nucleotide diversity
- List of seven significantly increased ARGs (P < .05)

## How to apply

First compute per-gene, per-sample nucleotide diversity by running nucleotide_diversity_code.R on the cellsnp-lite pile-up (cellSNP.cells.vcf.gz), MetaQuad clustering parameters (BIC_params.csv from 'metaquad -i cellSNP -o Metaquad --minSample=5'), and a genome_length.csv normalization file; optionally filter to genes with nucleotide diversity > 0.005. Split samples into pre-treatment ('00') and 6-weeks-post-LEVO ('01') groups and apply a Wilcoxon rank-sum test per gene. Report genes with significantly increased diversity at P < .05; in the LEVO cohort this yielded seven ARGs whose diversity increase reversed by 6 months, consistent with the article's finding that 'the nucleotide diversity of some genes is increased 6 weeks after antibiotic treatment'.

## Related tools

- **MetaQuad** (Detects shared informative variants via density clustering; its BIC_params.csv output supplies variant clustering parameters feeding the nucleotide diversity table that is tested.) — https://github.com/holab-hku/MetaQuad
- **cellsnp-lite** (Piles up variants (cellSNP.cells_ARG.vcf.gz) used to compute per-gene nucleotide diversity prior to group testing.) — https://github.com/single-cell-genetics/cellsnp-lite
- **samtools** (Computes per-sample coverage ('samtools coverage ${i}.sorted.bam > ${i}.tab') summarized into genome_length.csv for diversity normalization.)
- **vcfR** (R package used to load the cellsnp-lite VCF in nucleotide_diversity_code.R, producing the diversity table to which the Wilcoxon test is applied.)

## Examples

```
Rscript nucleotide_diversity_code.R cellSNP.cells.vcf.gz BIC_params.csv genome_length.csv
```

## Evaluation signals

- Exactly seven ARGs show significantly increased nucleotide diversity at P < .05 for the '00' vs '01' LEVO comparison.
- All reported P values are below 0.05 and correspond to increases (not decreases) in diversity at 6 weeks.
- The diversity shift direction reverses when comparing against the 6-month time point, matching the reported temporal pattern.
- Nucleotide_diversity.csv exists with one value per gene per sample and genes filtered at diversity > 0.005 before testing.
- MetaQuad BIC_params.csv was produced with --minSample=5 and contains variant_names, num_of_clusters, mean_DP, mean_AD, and number_DP columns.

## Limitations

- The reported seven significant ARGs are specific to the LEVO-treated cohort and time points ('00' vs '01'); the effect reverses at 6 months, so results are time-window dependent.
- Diversity normalization depends on genome_length.csv, whose values may be genome lengths or covbases 'depending on the specific situation', affecting comparability.
- MetaQuad and cellsnp-lite recommend specific thresholds (minMAF = 0.02, minCOUNT = 100; minSample = 5% of samples) — deviations change which variants inform the diversity estimates.

## Evidence

- [results] In patients treated with LEVO, seven ARGs showed significantly increased nucleotide diversity (P < .05): [evidence span withheld — anchor status: not_found]
- [abstract] The nucleotide diversity of some genes is increased 6 weeks after antibiotic treatment, potentially indicating the role of these genes in specific antibiotic treatments.: "The nucleotide diversity of some genes is increased 6 weeks after antibiotic treatment, potentially indicating the role of these genes in specific antibiotic treatments."
- [readme] Rscript nucleotide_diversity_code.R cellSNP.cells.vcf.gz BIC_params.csv genome_length.csv: "Rscript nucleotide_diversity_code.R cellSNP.cells.vcf.gz BIC_params.csv genome_length.csv"
- [methods] metaquad -i cellSNP -o Metaquad --minSample=5: "metaquad  -i cellSNP -o Metaquad --minSample=5"
