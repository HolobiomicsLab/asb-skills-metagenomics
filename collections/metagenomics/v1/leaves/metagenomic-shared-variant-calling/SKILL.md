---
name: metagenomic-shared-variant-calling
description: Use when you have shotgun metagenomic BAM alignments from many samples
  and need to identify SNPs shared across the population (e.g., strain-tracking or
  ARG diversity analysis), not per-sample de novo calling.
when_to_use_negative:
- You have a single sample or very few samples — minSample = 5% of total sample size
  and shared-variant detection require a population of many metagenomic samples
- The input is single-cell mitochondrial data for clonal substructure — use the parent
  tool MQuad, since MetaQuad is specifically tailored to shotgun metagenomic data
- You need per-sample genotype calls rather than population-shared informative variants
  — comparator callers like BCFtools or GATK HaplotypeCaller address that goal
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3168
tools:
- name: MetaQuad
  role: Core density-clustering caller that scores cellSNP pile-ups and flags shared
    informative variants (num_of_clusters >= 2)
  repo: https://github.com/holab-hku/MetaQuad
- name: cellsnp-lite
  role: Piles up bi-allelic SNP allele/depth counts from BAMs into the cellSNP folder
    consumed by MetaQuad (minMAF = 0.02, minCOUNT = 100)
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: bwa
  role: Aligns filtered shotgun metagenomic reads to the reference database to create
    .filtered.sorted.bam files
- name: bowtie2
  role: Alternative aligner to bwa for producing .filtered.sorted.bam input files
- name: samtools
  role: Computes per-BAM coverage tables ('samtools coverage') used as genome-length
    normalization factors for downstream nucleotide diversity
- name: MQuad
  role: Parent tool whose binomial-mixture/BIC framework MetaQuad extends for metagenomic
    data
  repo: https://github.com/single-cell-genetics/MQuad
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:87be26b6cc5202a31bf55df31413380f36607b40e3f507ca012fb5c0aca790be
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
  - samtools
  - vcfR
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 21
---

# metagenomic-shared-variant-calling

## Summary

Detect shared informative variants across a population of shotgun metagenomic samples using MetaQuad, a density/binomial-mixture clustering extension of MQuad. It distinguishes genuinely shared polymorphic sites from background noise after cellSNP-lite pile-up.

## When to use

Use when you have shotgun metagenomic BAM alignments from many samples and need to identify SNPs shared across the population (e.g., strain-tracking or ARG diversity analysis), not per-sample de novo calling.

## When NOT to use

- You have a single sample or very few samples — minSample = 5% of total sample size and shared-variant detection require a population of many metagenomic samples
- The input is single-cell mitochondrial data for clonal substructure — use the parent tool MQuad, since MetaQuad is specifically tailored to shotgun metagenomic data
- You need per-sample genotype calls rather than population-shared informative variants — comparator callers like BCFtools or GATK HaplotypeCaller address that goal

## Inputs

- Shotgun metagenomic reads aligned to a reference database (.filtered.sorted.bam from bwa or bowtie2)
- cellSNP-lite output folder (AD/DP sparse .mtx matrices and pile-up VCF) produced with minMAF = 0.02 and minCOUNT = 100
- Simulated or real ground-truth variant list for benchmarking (optional)

## Outputs

- BIC_params.csv with per-variant deltaBIC, num_of_clusters, and fitted binomial-mixture parameters
- Set of shared informative mutations defined by num_of_clusters >= 2
- Downstream per-gene Nucleotide_diversity.csv when combined with samtools coverage and nucleotide_diversity_code.R

## How to apply

First filter and align reads to a reference database with bwa or bowtie2, producing '.filtered.sorted.bam' files. Pile up variants with cellsnp-lite using the recommended minMAF = 0.02 and minCOUNT = 100, which yields the cellSNP output folder (AD/DP sparse matrices and VCF) that MetaQuad takes as input. Install MetaQuad via pip from GitHub and run 'metaquad -i cellSNP -o Metaquad' with minSample set to 5% of the total sample size (e.g., --minSample=5 for n=100, --minSample=50 for n=1000). Read the produced BIC_params.csv and retain variants with num_of_clusters >= 2 as informative shared mutations; the deltaBIC-based density clustering separates informative variants from background noise. Downstream, the cellSNP VCF, BIC_params.csv, and samtools-coverage-derived genome lengths can feed nucleotide-diversity computation per gene (e.g., ARGs), with Wilcoxon rank-sum tests comparing treatment time points.

## Related tools

- **MetaQuad** (Core density-clustering caller that scores cellSNP pile-ups and flags shared informative variants (num_of_clusters >= 2)) — https://github.com/holab-hku/MetaQuad
- **cellsnp-lite** (Piles up bi-allelic SNP allele/depth counts from BAMs into the cellSNP folder consumed by MetaQuad (minMAF = 0.02, minCOUNT = 100)) — https://github.com/single-cell-genetics/cellsnp-lite
- **bwa** (Aligns filtered shotgun metagenomic reads to the reference database to create .filtered.sorted.bam files)
- **bowtie2** (Alternative aligner to bwa for producing .filtered.sorted.bam input files)
- **samtools** (Computes per-BAM coverage tables ('samtools coverage') used as genome-length normalization factors for downstream nucleotide diversity)
- **MQuad** (Parent tool whose binomial-mixture/BIC framework MetaQuad extends for metagenomic data) — https://github.com/single-cell-genetics/MQuad

## Examples

```
metaquad -i cellSNP -o Metaquad --minSample=5
```

## Evaluation signals

- minSample is set to 5% of the total sample size (e.g., 5 for n=100; 50 for n=1000) — deviation breaks comparability with the reported benchmark
- Only variants with num_of_clusters >= 2 in BIC_params.csv are retained as informative shared mutations
- Benchmark check: MetaQuad reduces false positive SNPs versus BCFtools, GATK HaplotypeCaller, VarScan2, MetaSNV, and inStrain without greatly affecting the true positive rate, with F1 approaching 1 at high mean depth
- Scalability check on n=1000 at mean depth 11.8: recovery of 7598 of 7638 true positives with zero false positives
- Applied-acceptance example: 7 ARGs show significantly increased nucleotide diversity (Wilcoxon P < .05) between pre-treatment (00) and 6-weeks-post-LEVO (01) groups

## Limitations

- cellsnp-lite pile-up cost grows with sample size; the README recommends higher minCOUNT values to reduce computation time for subsequent steps
- Performance depends on mean depth: all callers, including MetaQuad, approach perfect F1 (~1) only as mean depth increases, so low-depth data will underperform
- MetaQuad is an extension of MQuad designed for shotgun metagenomic populations; it is not intended for single-cell mitochondrial clonal analysis

## Evidence

- [intro] Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2.: "Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2."
- [readme] Pile up variants using [cellsnp-lite](https://github.com/single-cell-genetics/cellsnp-lite). with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100.: "Pile up variants using [cellsnp-lite](https://github.com/single-cell-genetics/cellsnp-lite). <br/> with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100."
- [methods] metaquad  -i cellSNP -o Metaquad --minSample=5: "metaquad  -i cellSNP -o Metaquad --minSample=5"
- [abstract] Empirical comparisons with other state-of-the-art methods show that MetaQuad significantly reduces the number of false positive SNPs without greatly affecting the true positive rate.: "Empirical comparisons with other state-of-the-art methods show that MetaQuad significantly reduces the number of false positive SNPs without greatly affecting the true positive rate."
- [readme] A higher value is recommended to reduce computation time for subsequent steps.: "A higher value is recommended to reduce computation time for subsequent steps."
