---
name: gene-level-microdiversity-profiling
description: Use when mapping metagenomic reads of a co-occurring strain (e.g. phage
  482_10.ph) back to its own reference and you need per-gene coverage, SNP types,
  and dN/dS to test whether within-individual diversity overlaps between-individual
  variation.
when_to_use_negative:
- Strains are not co-occurring within samples (single-strain, low-coverage mappings)
  — inStrain's strain-level population model requires sufficient within-sample microdiversity
  signal
- The goal is only coarse genome coverage/breadth estimation, which CoverM can compute
  far faster without SNV calling
- Variant sites near the ends of short genome fragments are the targets, since inStrain
  can miss these
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: inStrain
  role: Profiles coverage, microdiversity, SNVs, and gene-level SNP types from the
    .bam mappings ('profile' and 'profile_genes' commands)
  repo: https://github.com/MrOlm/instrain
- name: Samtools
  role: Required by inStrain for interacting with .bam and .sam files
- name: Prodigal
  role: Annotates open reading frames; its gene calls (.fna) feed into 'inStrain profile_genes'
    for per-gene SNP localization
- name: Pandas
  role: Extracts and classifies biallelic SNV positions as intra- vs inter-infant
- name: SciPy
  role: Supports dN/dS computation alongside Numpy
- name: Numpy
  role: Computes dN/dS from counts of nonsynonymous/synonymous substitutions and sites
- name: Matplotlib
  role: Plots per-gene dN/dS and the within- vs between-infant nonsynonymous change
    distribution
- name: Seaborn
  role: Used with Matplotlib for the diversity plotting
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:207d096a2722182ef5de6acc1dbd9773b26a4a3001487f994d8e671d9bfcb35a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MrOlm/instrain
  tools:
  - Matplotlib
  - Numpy
  - Pandas
  - Prodigal
  - Samtools
  - SciPy
  - Seaborn
  - inStrain
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# gene-level-microdiversity-profiling

## Summary

Profiles population microdiversity within individual genes by running inStrain's SNP calling and 'profile_genes' module on read mappings, then classifying variants as intra- vs inter-host (here intra- vs inter-infant) and computing per-gene dN/dS to reveal loci under divergent selection.

## When to use

Use when mapping metagenomic reads of a co-occurring strain (e.g. phage 482_10.ph) back to its own reference and you need per-gene coverage, SNP types, and dN/dS to test whether within-individual diversity overlaps between-individual variation.

## When NOT to use

- Strains are not co-occurring within samples (single-strain, low-coverage mappings) — inStrain's strain-level population model requires sufficient within-sample microdiversity signal
- The goal is only coarse genome coverage/breadth estimation, which CoverM can compute far faster without SNV calling
- Variant sites near the ends of short genome fragments are the targets, since inStrain can miss these

## Inputs

- Sorted .bam file per infant sample mapping reads to the phage 482_10.ph reference .fasta
- Reference genome .fasta for the phage
- Prodigal gene calls (.fna) for the phage genome
- Phage gene annotations for synonym/nonsynonym classification

## Outputs

- inStrain profile output directory with genome-, scaffold-, and gene-level coverage and nucleotide diversity
- inStrain profile_genes report of per-gene coverage, microdiversity, and SNP type
- Table of biallelic SNVs classified as intra-infant vs inter-infant and synonymous vs nonsynonymous
- Per-gene dN/dS values and gene presence/deletion frequencies across samples
- Figure of per-gene dN/dS and within- vs between-infant nonsynonymous changes (tail fiber gene highlighted)

## How to apply

Run 'inStrain profile' on each infant sample's sorted .bam against the phage 482_10.ph .fasta with default read-pair filters (proper orientation, expected insert size, minimum read-pair ANI 0.95) and default SNP thresholds (min_cov 5, min_freq 0.05, FDR 1e-6). Then run 'inStrain profile_genes' with Prodigal gene calls to obtain per-gene coverage, microdiversity, and SNP type. Extract biallelic SNV positions across infants with Pandas and classify each as intra-infant (biallelic within a single infant) or inter-infant (substitution between infants). Label variants as synonymous or nonsynonymous from the Prodigal annotations and compute per-gene dN/dS as [(non-synonymous substitutions / non-synonymous sites) / (synonymous substitutions / synonymous sites)] with SciPy/Numpy, plus per-gene presence/deletion frequency from gene-level coverage. Plot per-gene dN/dS and the within- vs between-infant nonsynonymous change distribution (highlighting the tail fiber gene) with Matplotlib/Seaborn.

## Related tools

- **inStrain** (Profiles coverage, microdiversity, SNVs, and gene-level SNP types from the .bam mappings ('profile' and 'profile_genes' commands)) — https://github.com/MrOlm/instrain
- **Samtools** (Required by inStrain for interacting with .bam and .sam files)
- **Prodigal** (Annotates open reading frames; its gene calls (.fna) feed into 'inStrain profile_genes' for per-gene SNP localization)
- **Pandas** (Extracts and classifies biallelic SNV positions as intra- vs inter-infant)
- **SciPy** (Supports dN/dS computation alongside Numpy)
- **Numpy** (Computes dN/dS from counts of nonsynonymous/synonymous substitutions and sites)
- **Matplotlib** (Plots per-gene dN/dS and the within- vs between-infant nonsynonymous change distribution)
- **Seaborn** (Used with Matplotlib for the diversity plotting)

## Examples

```
inStrain profile mapping.bam 482_10.ph.fasta -o inStrain_profile1; inStrain profile_genes inStrain_profile1 genes.fna
```

## Evaluation signals

- The output reproduces the reported pattern that nonsynonymous changes in the tail fiber gene are purged within infants yet selected for between infants
- Gene-level coverage and diversity values match the inStrain profile_genes report
- SNV counts pass the default filters: minimum 5 reads mapping to the position, more than 5% of reads with the variant base, and variant reads exceeding the null model at FDR 1e-6
- Read pairs retained satisfy proper orientation, expected insert size, and minimum read-pair ANI of 0.95
- dN/dS values reproduce the formula [(non-synonymous substitutions / non-synonymous sites) / (synonymous substitutions / synonymous sites)] per gene

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain
- It is difficult to distinguish microbiome diversity that is evolved in situ from that introduced by immigration (the authors suggest the shared variants may reflect inoculation from diverse hospital-associated sources)
- Read filtering must be careful to reduce the probability of reads being erroneously mapped, which could inflate apparent SNVs

## Evidence

- [discussion] profile_genes workflow: [evidence span withheld — anchor status: not_found]
- [readme] README quick start: [evidence span withheld — anchor status: external_doc]
