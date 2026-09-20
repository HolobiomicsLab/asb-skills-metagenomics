---
name: nucleotide-diversity-calculation
description: Use when you have a cellsnp-lite VCF pile-up (e.g., cellSNP.cells_ARG.vcf.gz)
  plus MetaQuad BIC_params.csv and need per-gene, per-sample nucleotide diversity
  to detect polymorphism shifts such as ARG diversity changes after antibiotic treatment.
when_to_use_negative:
- You only need raw variant calling/clustering — MetaQuad's BIC_params.csv already
  provides shared informative variants without diversity estimates.
- No per-gene normalization is available (neither genome lengths nor covbases), since
  raw pile-up counts cannot be fairly compared across genes of different lengths.
- Input is single-cell mitochondrial data intended for clonal substructure inference
  (MQuad/vireoSNP use case), not shotgun metagenomic diversity analysis.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaQuad
  role: Detects shared informative variants and produces BIC_params.csv used as clustering
    input for the diversity calculation
  repo: https://github.com/holab-hku/MetaQuad
- name: cellsnp-lite
  role: Piles up variants into the cellSNP.cells_ARG.vcf.gz VCF that the diversity
    script reads via vcfR
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: samtools
  role: Computes per-BAM coverage statistics ('samtools coverage ${i}.sorted.bam')
    used to build genome_length.csv normalization factors
- name: vcfR
  role: R package required by nucleotide_diversity_code.R to parse the cellsnp-lite
    VCF
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:be5e06bb0f5cf3e9e7911d18198a8b940dec2887651cea35ca50ebf17e53caaf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad
  tools:
  - MetaQuad
  - cellsnp-lite
  - samtools
  - vcfR
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# nucleotide-diversity-calculation

## Summary

Computes nucleotide diversity per gene and sample from cellsnp-lite variant pile-ups and MetaQuad cluster parameters, normalized by genome length or covbases, to quantify polymorphism within metagenomic populations (e.g., ARGs after antibiotic treatment).

## When to use

Use when you have a cellsnp-lite VCF pile-up (e.g., cellSNP.cells_ARG.vcf.gz) plus MetaQuad BIC_params.csv and need per-gene, per-sample nucleotide diversity to detect polymorphism shifts such as ARG diversity changes after antibiotic treatment.

## When NOT to use

- You only need raw variant calling/clustering — MetaQuad's BIC_params.csv already provides shared informative variants without diversity estimates.
- No per-gene normalization is available (neither genome lengths nor covbases), since raw pile-up counts cannot be fairly compared across genes of different lengths.
- Input is single-cell mitochondrial data intended for clonal substructure inference (MQuad/vireoSNP use case), not shotgun metagenomic diversity analysis.

## Inputs

- cellSNP.cells_ARG.vcf.gz — cellsnp-lite variant pile-up VCF
- BIC_params.csv — MetaQuad clustering parameters (variant_names, num_of_clusters, mean_DP, mean_AD, number_DP)
- genome_length.csv — per-gene/per-sample normalization values (genome lengths or covbases, from samtools coverage + Genome_coverage.R)

## Outputs

- Nucleotide_diversity.csv — per-gene, per-sample nucleotide diversity table
- genome_length.csv and cov_bases.csv — intermediate normalization tables from Genome_coverage.R
- Filtered list of ARGs with significantly increased diversity (Wilcoxon rank-sum, P < .05)

## How to apply

Run the R script nucleotide_diversity_code.R with the cellsnp-lite output VCF (loaded via the vcfR package), the MetaQuad BIC_params.csv (produced with 'metaquad -i cellSNP -o Metaquad --minSample=5'), and genome_length.csv as normalization factors. Generate genome_length.csv by running 'samtools coverage ${i}.sorted.bam > ${i}.tab' on each sorted BAM, filtering with 'grep -Ff genomes.txt', and summarizing with Rscript Genome_coverage.R (values may be genome lengths or covbases per gene/sample depending on the situation). The script writes Nucleotide_diversity.csv with per-gene, per-sample diversity values. Filter to genes with nucleotide diversity > 0.005 and apply per-gene Wilcoxon rank-sum tests comparing pre-treatment (00) vs 6-weeks-post-LEVO (01) groups at P < .05 to identify significantly shifted ARGs.

## Related tools

- **MetaQuad** (Detects shared informative variants and produces BIC_params.csv used as clustering input for the diversity calculation) — https://github.com/holab-hku/MetaQuad
- **cellsnp-lite** (Piles up variants into the cellSNP.cells_ARG.vcf.gz VCF that the diversity script reads via vcfR) — https://github.com/single-cell-genetics/cellsnp-lite
- **samtools** (Computes per-BAM coverage statistics ('samtools coverage ${i}.sorted.bam') used to build genome_length.csv normalization factors)
- **vcfR** (R package required by nucleotide_diversity_code.R to parse the cellsnp-lite VCF)

## Examples

```
Rscript nucleotide_diversity_code.R cellSNP.cells.vcf.gz BIC_params.csv genome_length.csv
```

## Evaluation signals

- nucleotide_diversity_code.R runs without error and produces a file named Nucleotide_diversity.csv
- Genome_coverage.R yields both genome_length.csv and cov_bases.csv from the samtools coverage .tab files
- Normalization values in genome_length.csv match either genome lengths or covbases for each gene and sample (per README guidance)
- Genes surviving the nucleotide diversity > 0.005 filter show significant Wilcoxon rank-sum differences (P < .05) between 00 and 01 groups — in the LEVO study this yields seven ARGs
- BIC_params.csv columns conform to the documented schema: variant_names, num_of_clusters, mean_DP, mean_AD, number_DP

## Limitations

- genome_length.csv values depend on the situation — either genome lengths or covbases may be used, so the normalization choice affects cross-gene comparability.
- cellsnp-lite recommended parameters (minMAF = 0.02, minCOUNT = 100) influence which variants are piled up; higher values reduce computation time but may drop informative variants.
- MetaQuad minSample is recommended as 5% of the total sample size; genes observed in fewer samples cannot be assessed.
- Diversity changes may reverse over time — the LEVO finding showed increased diversity at 6 weeks reversing at 6 months, so single-timepoint comparisons can mislead.

## Evidence

- [methods] Rscript nucleotide_diversity_code.R cellSNP.cells.vcf.gz BIC_params.csv genome_length.csv: "Rscript nucleotide_diversity_code.R cellSNP.cells.vcf.gz BIC_params.csv genome_length.csv"
- [readme] expected output from the nucleotide_diversity_code.R script: "The expected output from the nucleotide_diversity_code.R script is a CSV file called "Nucleotide_diversity.csv"."
- [readme] normalization with genome lengths or covbases: "The values in genome_length.csv can be either genome lengths or covbases for each gene and sample, depending on the specific situation."
- [methods] samtools coverage workflow: [evidence span withheld — anchor status: not_found]
- [abstract] increased diversity 6 weeks after treatment: "The nucleotide diversity of some genes is increased 6 weeks after antibiotic treatment, potentially indicating the role of these genes in specific antibiotic treatments."
- [readme] vcfR requirement: "To calculate nucleotide diversity, you can use the nucleotide_diversity_code.R script, which requires the R package "vcfR"."
