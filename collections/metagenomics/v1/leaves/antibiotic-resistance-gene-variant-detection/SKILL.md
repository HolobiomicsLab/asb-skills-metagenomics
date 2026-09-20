---
name: antibiotic-resistance-gene-variant-detection
description: Use when you need to detect shared informative variants across a population
  of shotgun metagenomic samples aligned to a reference gene database (e.g., antibiotic
  resistance genes), and want to separate shared polymorphic sites from background
  noise.
when_to_use_negative:
- Input is single-cell mitochondrial data rather than shotgun metagenomic samples
  — MQuad (the parent tool) is tailored for that setting, not this one.
- The goal is calling variants within a single sample rather than shared informative
  variants across a population of samples (minSample filtering requires multiple samples).
- Reads cannot be aligned to a reference gene database (e.g., no suitable ARG reference),
  since the pipeline starts from bwa/bowtie2 alignment to a reference.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: MetaQuad
  role: Detects shared informative variants from the cellsnp-lite output using density-based
    BIC clustering, with minSample = 5% of total sample size and num_of_clusters >=
    2
  repo: https://github.com/holab-hku/MetaQuad
- name: cellsnp-lite
  role: Piles up per-sample variant allele counts (AD/DP) from aligned BAMs with recommended
    minMAF = 0.02 and minCOUNT = 100
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: bwa
  role: Aligns filtered shotgun metagenomic reads to the ARG reference database, producing
    .filtered.sorted.bam files
- name: bowtie2
  role: Alternative aligner to bwa for producing per-sample .filtered.sorted.bam files
- name: samtools
  role: Computes per-sample coverage statistics (samtools coverage ${i}.sorted.bam
    > ${i}.tab) for genome length and covbases calculations
- name: MQuad
  role: Parent tool that MetaQuad extends for shotgun metagenomic data; shares the
    BIC_params.csv output schema and density clustering approach
  repo: https://github.com/single-cell-genetics/MQuad
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:d48e7caa11c603f6ea137128c9907171d66aff90cb37f3f434cb68800916257a
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# antibiotic-resistance-gene-variant-detection

## Summary

A pipeline for detecting shared informative variants across antibiotic resistance genes (ARGs) in shotgun metagenomic samples, using cellsnp-lite pileup followed by MetaQuad's density-based BIC clustering. Applied to 121 H. pylori eradication therapy samples, it recovered 7591 informative variants across 529 ARGs in about 60 minutes.

## When to use

Use when you need to detect shared informative variants across a population of shotgun metagenomic samples aligned to a reference gene database (e.g., antibiotic resistance genes), and want to separate shared polymorphic sites from background noise.

## When NOT to use

- Input is single-cell mitochondrial data rather than shotgun metagenomic samples — MQuad (the parent tool) is tailored for that setting, not this one.
- The goal is calling variants within a single sample rather than shared informative variants across a population of samples (minSample filtering requires multiple samples).
- Reads cannot be aligned to a reference gene database (e.g., no suitable ARG reference), since the pipeline starts from bwa/bowtie2 alignment to a reference.

## Inputs

- Shotgun metagenomic FASTQ reads per sample (121 H. pylori eradication therapy samples)
- Antibiotic resistance gene (ARG) reference database for alignment
- Per-sample .filtered.sorted.bam alignment files (from bwa or bowtie2)
- cellsnp-lite output folder (cellSNP.cells_ARG.vcf.gz, AD/DP matrices) with minMAF = 0.02 and minCOUNT = 100

## Outputs

- BIC_params.csv with columns variant_names, num_of_clusters, mean_DP, mean_AD, number_DP, sorted by deltaBIC
- Set of shared informative variants (7591 variants across 529 ARGs in the reference study)
- passed_ad.mtx / passed_dp.mtx sparse AD/DP matrices of qualified variants for downstream analysis
- deltaBIC_cdf.pdf and top variants heatmap.pdf diagnostic plots

## How to apply

Install MetaQuad from GitHub ('pip install -U git+https://github.com/holab-hku/MetaQuad.git') and verify with 'metaquad -h'. Align the shotgun metagenomic reads (e.g., 121 samples) to the antibiotic resistance gene reference database with bwa or bowtie2, producing per-sample .filtered.sorted.bam files. Pile up variants per sample with cellsnp-lite using the recommended parameters minMAF = 0.02 and minCOUNT = 100, writing to the cellSNP output folder. Run MetaQuad on that folder with 'metaquad -i cellSNP -o Metaquad --minSample=6', where minSample = 5% of the 121 samples; MetaQuad uses a density clustering model (num_of_clusters >= 2) to distinguish informative variants from background noise. Apply the reported quality thresholds (mean AD 0.1, mean DP 0.5) to retain shared informative variants, then inspect BIC_params.csv (columns variant_names, num_of_clusters, mean_DP, mean_AD, number_DP) for the final variant set.

## Related tools

- **MetaQuad** (Detects shared informative variants from the cellsnp-lite output using density-based BIC clustering, with minSample = 5% of total sample size and num_of_clusters >= 2) — https://github.com/holab-hku/MetaQuad
- **cellsnp-lite** (Piles up per-sample variant allele counts (AD/DP) from aligned BAMs with recommended minMAF = 0.02 and minCOUNT = 100) — https://github.com/single-cell-genetics/cellsnp-lite
- **bwa** (Aligns filtered shotgun metagenomic reads to the ARG reference database, producing .filtered.sorted.bam files)
- **bowtie2** (Alternative aligner to bwa for producing per-sample .filtered.sorted.bam files)
- **samtools** (Computes per-sample coverage statistics (samtools coverage ${i}.sorted.bam > ${i}.tab) for genome length and covbases calculations)
- **MQuad** (Parent tool that MetaQuad extends for shotgun metagenomic data; shares the BIC_params.csv output schema and density clustering approach) — https://github.com/single-cell-genetics/MQuad

## Examples

```
metaquad -i cellSNP -o Metaquad --minSample=6
```

## Evaluation signals

- BIC_params.csv exists, is parseable, and its variant_names column yields ~7591 informative variants spanning 529 ARGs when applied to the 121-sample H. pylori ARG dataset
- Retained variants satisfy the quality thresholds mean AD 0.1 and mean DP 0.5, and have num_of_clusters >= 2
- minSample parameter equals 5% of the total sample size (e.g., --minSample=6 for 121 samples)
- Full detection completes within approximately 60 minutes of runtime
- deltaBIC_cdf.pdf is produced and shows the deltaBIC cutoff applied by MetaQuad to separate informative variants from background noise

## Limitations

- cellsnp-lite minMAF/minCOUNT thresholds trade sensitivity for speed: a higher value is recommended to reduce computation time for subsequent steps, which may miss low-frequency variants.
- The default --minDP 10 in the MQuad family of tools may be too stringent for low sequencing depth data; --minDP 5 or smaller is recommended for droplet/low-depth data.
- The method distinguishes shared variants from noise using a density clustering model, so variants present in only 1 or 2 samples (num_cells_minor_cpt-like cases) may be filtered out.
- cellsnp-lite is described as a faster version of cellSNP that is 'still at testing stage so might be unstable'.

## Evidence

- [abstract] MetaQuad utilizes a density-based clustering technique to effectively distinguish between shared variants and non-polymorphic sites using shotgun metagenomic data.: "MetaQuad utilizes a density-based clustering technique to effectively distinguish between shared variants and non-polymorphic sites using shotgun metagenomic data."
- [abstract] MetaQuad detected 7591 variants across 529 antibiotic resistance genes.: "MetaQuad detected 7591 variants across 529 antibiotic resistance genes."
- [intro] Pile up variants using cellsnp-lite with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100.: "Pile up variants using [cellsnp-lite](https://github.com/single-cell-genetics/cellsnp-lite). <br/> with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100."
- [methods] metaquad  -i cellSNP -o Metaquad --minSample=5: "metaquad  -i cellSNP -o Metaquad --minSample=5"
- [readme] It is recommended to add --minDP 5 or a smaller value to prevent errors during fitting. The default value is 10, which is suitable for Smart-seq2 data but might be too stringent for low sequencing depth data.: [evidence span withheld — anchor status: external_doc]
