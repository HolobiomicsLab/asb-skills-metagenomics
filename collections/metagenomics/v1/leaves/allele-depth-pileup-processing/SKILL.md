---
name: allele-depth-pileup-processing
description: Use when shotgun metagenomic BAMs aligned to a reference gene database
  must be converted into allele-depth pile-ups (cellSNP VCF), clustered via MetaQuad,
  and normalized into per-gene nucleotide diversity for group comparisons.
when_to_use_negative:
- Input is already a computed Nucleotide_diversity.csv table — proceed directly to
  the diversity > 0.005 filter and Wilcoxon rank-sum comparison rather than re-running
  the pile-up pipeline.
- The goal is single-cell mitochondrial clonal substructure inference (MQuad/vireoSNP
  path), not shared informative variant discovery in shotgun metagenomic populations.
- Variants of interest are non-bi-allelic or occur at frequencies below the cellsnp-lite
  minMAF = 0.02 / minCOUNT = 100 pile-up thresholds, so AD/DP counts will be too sparse
  to cluster.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: cellsnp-lite
  role: Piles up bi-allelic SNP allele depths/depths (AD/DP) per sample into the VCF
    consumed by MetaQuad and the nucleotide diversity script
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: MetaQuad
  role: Density-based clustering of pile-up variants to separate shared informative
    variants from background noise; emits BIC_params.csv used as nucleotide_diversity_code.R
    input
  repo: https://github.com/holab-hku/MetaQuad
- name: samtools
  role: Computes per-sample per-gene coverage stats (`samtools coverage ${i}.sorted.bam`)
    that feed genome_length.csv / cov_bases.csv normalization
- name: vcfR
  role: R package used by nucleotide_diversity_code.R to load the cellsnp-lite VCF
    and compute nucleotide diversity
- name: bwa
  role: Alignment tool used upstream to map filtered shotgun reads to the reference
    gene database before pile-up
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:a9f1ae16a2e77d86daa9acebd02d89278c04b2603ceb360afbf4d3d950bda58d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/cellsnp-lite
  tools:
  - MetaQuad
  - bwa
  - cellsnp-lite
  - samtools
  - vcfR
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# allele-depth-pileup-processing

## Summary

Processes cellSNP-lite allele-depth (AD/DP) pile-ups of shotgun metagenomic reads aligned to ARG references into MetaQuad cluster parameters and per-gene, per-sample nucleotide diversity, enabling detection of shared informative variants (e.g., ARG diversity shifts after LEVO treatment).

## When to use

Use when shotgun metagenomic BAMs aligned to a reference gene database must be converted into allele-depth pile-ups (cellSNP VCF), clustered via MetaQuad, and normalized into per-gene nucleotide diversity for group comparisons.

## When NOT to use

- Input is already a computed Nucleotide_diversity.csv table — proceed directly to the diversity > 0.005 filter and Wilcoxon rank-sum comparison rather than re-running the pile-up pipeline.
- The goal is single-cell mitochondrial clonal substructure inference (MQuad/vireoSNP path), not shared informative variant discovery in shotgun metagenomic populations.
- Variants of interest are non-bi-allelic or occur at frequencies below the cellsnp-lite minMAF = 0.02 / minCOUNT = 100 pile-up thresholds, so AD/DP counts will be too sparse to cluster.

## Inputs

- .filtered.sorted.bam files of shotgun metagenomic reads aligned to a reference gene database (e.g., ARGs)
- cellSNP.cells_ARG.vcf.gz allele-depth pile-up from cellsnp-lite
- BIC_params.csv from MetaQuad clustering (variant_names, num_of_clusters, mean_DP, mean_AD, number_DP)
- genome_length.csv or cov_bases.csv per gene and sample (from samtools coverage + Genome_coverage.R)
- genomes.txt listing genes/genomes of interest for coverage filtering

## Outputs

- cellSNP.cells_ARG.vcf.gz allele x sample pile-up VCF
- BIC_params.csv with predicted number of clusters and mean AD/DP per variant
- genome_length.csv and cov_bases.csv normalization tables
- Nucleotide_diversity.csv with per-gene, per-sample nucleotide diversity
- List of genes with significantly shifted diversity (e.g., 7 ARGs at Wilcoxon P < .05 in LEVO-treated patients)

## How to apply

First align filtered reads to the reference gene database with bwa or bowtie2 (output suffix .filtered.sorted.bam), then pile up bi-allelic SNPs with cellsnp-lite using minMAF = 0.02 and minCOUNT = 100 to produce cellSNP.cells_ARG.vcf.gz. Detect informative shared variants with `metaquad -i cellSNP -o Metaquad --minSample=5`, which writes BIC_params.csv (columns: variant_names, num_of_clusters, mean_DP, mean_AD, number_DP; recommended num_of_clusters >= 2, minSample = 5% of total sample size). Compute per-gene normalization factors by running `samtools coverage ${i}.sorted.bam > ${i}.tab` per sample, filtering with `grep -Ff genomes.txt`, and summarizing with `Rscript Genome_coverage.R` to yield genome_length.csv or cov_bases.csv. Finally run `Rscript nucleotide_diversity_code.R cellSNP.cells.vcf.gz BIC_params.csv genome_length.csv` (requires the R vcfR package) to obtain Nucleotide_diversity.csv with diversity per gene and sample; downstream, filter genes with nucleotide diversity > 0.005 and compare pre-treatment (00) vs 6-weeks-post-LEVO (01) groups per gene with a Wilcoxon rank-sum test at P < .05.

## Related tools

- **cellsnp-lite** (Piles up bi-allelic SNP allele depths/depths (AD/DP) per sample into the VCF consumed by MetaQuad and the nucleotide diversity script) — https://github.com/single-cell-genetics/cellsnp-lite
- **MetaQuad** (Density-based clustering of pile-up variants to separate shared informative variants from background noise; emits BIC_params.csv used as nucleotide_diversity_code.R input) — https://github.com/holab-hku/MetaQuad
- **samtools** (Computes per-sample per-gene coverage stats (`samtools coverage ${i}.sorted.bam`) that feed genome_length.csv / cov_bases.csv normalization)
- **vcfR** (R package used by nucleotide_diversity_code.R to load the cellsnp-lite VCF and compute nucleotide diversity)
- **bwa** (Alignment tool used upstream to map filtered shotgun reads to the reference gene database before pile-up)

## Examples

```
Rscript nucleotide_diversity_code.R cellSNP.cells.vcf.gz BIC_params.csv genome_length.csv
```

## Evaluation signals

- BIC_params.csv conforms to the documented schema: variant_names, num_of_clusters, mean_DP, mean_AD, number_DP columns are present.
- MetaQuad was run with minSample = 5 (5% of total sample size) and num_of_clusters >= 2; cellsnp-lite pile-up used minMAF = 0.02 and minCOUNT = 100.
- Nucleotide_diversity.csv is produced by nucleotide_diversity_code.R and diversity values are normalized by genome_length.csv (either genome lengths or covbases per gene and sample).
- Applying the diversity > 0.005 filter and Wilcoxon rank-sum test on 00 vs 01 LEVO groups recovers seven ARGs with significantly increased diversity at P < .05, consistent with the published finding of increased diversity 6 weeks after treatment.
- In the published application, MetaQuad detected 7591 variants across 529 antibiotic resistance genes in H. pylori eradication therapy patients — a sanity-check scale for variant counts.

## Limitations

- Normalization values in genome_length.csv can be either genome lengths or covbases per gene and sample depending on the situation — choosing the wrong basis will bias diversity estimates.
- cellsnp-lite requires bi-allelic SNPs and its authors note it was 'still at testing stage so might be unstable' in some contexts.
- Higher cellsnp-lite thresholds are recommended to reduce computation time for subsequent steps, which trades sensitivity in low-depth genes for speed.
- Diversity increases 6 weeks after antibiotic treatment may reverse by 6 months, so single-post-treatment time points can overstate persistent shifts.

## Evidence

- [intro] Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2.: "Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2."
- [readme] The expected output from the nucleotide_diversity_code.R script is a CSV file called "Nucleotide_diversity.csv".: "The expected output from the nucleotide_diversity_code.R script is a CSV file called "Nucleotide_diversity.csv"."
- [readme] The values in genome_length.csv can be either genome lengths or covbases for each gene and sample, depending on the specific situation.: "The values in genome_length.csv can be either genome lengths or covbases for each gene and sample, depending on the specific situation."
- [readme] with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100: "with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100"
- [abstract] The nucleotide diversity of some genes is increased 6 weeks after antibiotic treatment, potentially indicating the role of these genes in specific antibiotic treatments.: "The nucleotide diversity of some genes is increased 6 weeks after antibiotic treatment, potentially indicating the role of these genes in specific antibiotic treatments."
