---
name: phage-microdiversity-visualization
description: Use when metagenomic .bam mappings of multiple hosts/infants to the same
  phage or microbial genome are available and the goal is to classify biallelic SNV
  positions as within- vs between-host variants and visualize selection (dN/dS) per
  gene.
when_to_use_negative:
- Input is already a SNP table or feature table with no read-level mappings — inStrain
  profiling requires .bam read-pair data
- Question is about strain sharing/comparison between profiles (use 'inStrain compare'
  ConANI/popANI instead of gene-level dN/dS visualization)
- Only rapid coverage/breadth estimates are needed, e.g. subspecies detection at >=
  0.5 unmaskedBreadth — use CoverM or inStrain genome-level output rather than the
  full dN/dS workflow
edam_operation: http://edamontology.org/operation_0484
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: inStrain
  role: Profiles microdiversity and calls SNVs from filtered read pairs; 'inStrain
    profile_genes' reports per-gene coverage, microdiversity, and SNP type
  repo: https://github.com/MrOlm/instrain
- name: Samtools
  role: Interacts with .bam/.sam files for read alignment handling during profiling
- name: Prodigal
  role: Annotates open reading frames of the phage reference to provide gene calls
    for profile_genes and synonymous/nonsynonymous labeling
- name: Pandas
  role: Extracts and classifies biallelic SNV positions across infants as intra- vs
    inter-infant
- name: SciPy
  role: Supports dN/dS computation with Numpy
- name: Numpy
  role: Numeric computation of per-gene dN/dS ratios
- name: Matplotlib
  role: Plots per-gene dN/dS and within- vs between-infant nonsynonymous change distributions
- name: Seaborn
  role: Styling/visualization of the per-gene dN/dS and nonsynonymous change plots
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:8f933fd49b0d84e5effb6361b5d58900d3e23ecb072346c4304a28e1e25b9d30
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
  edam_operation: http://edamontology.org/operation_0484
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# phage-microdiversity-visualization

## Summary

Reproduces the intra- vs inter-infant variant analysis of E. faecalis bacteriophage 482_10.ph (Figure 6) by profiling per-infant microdiversity with inStrain, computing per-gene dN/dS from Prodigal gene calls, and visualizing within- vs between-infant nonsynonymous changes to show tail fiber gene selection patterns.

## When to use

Use when metagenomic .bam mappings of multiple hosts/infants to the same phage or microbial genome are available and the goal is to classify biallelic SNV positions as within- vs between-host variants and visualize selection (dN/dS) per gene.

## When NOT to use

- Input is already a SNP table or feature table with no read-level mappings — inStrain profiling requires .bam read-pair data
- Question is about strain sharing/comparison between profiles (use 'inStrain compare' ConANI/popANI instead of gene-level dN/dS visualization)
- Only rapid coverage/breadth estimates are needed, e.g. subspecies detection at >= 0.5 unmaskedBreadth — use CoverM or inStrain genome-level output rather than the full dN/dS workflow

## Inputs

- Sorted .bam file per infant sample mapping reads to the phage 482_10.ph reference .fasta
- Phage reference genome .fasta (482_10.ph)
- Prodigal gene calls (.fna) for the phage reference
- inStrain profile_genes report (per-gene coverage, microdiversity, SNP type)

## Outputs

- Table of biallelic SNV positions classified as intra-infant or inter-infant, with synonymous/nonsynonymous labels
- Per-gene dN/dS values
- Per-gene presence/deletion frequency across infant samples
- Figure of per-gene dN/dS and within- vs between-infant nonsynonymous change distribution (tail fiber gene highlighted)

## How to apply

Run 'inStrain profile' on each infant sample's sorted .bam against the phage 482_10.ph .fasta, keeping default read-pair filters (proper orientation, expected insert size, minimum read-pair ANI 0.95) and default SNP thresholds (min_cov 5, min_freq 0.05, FDR 1e-6). Then run 'inStrain profile_genes' with the Prodigal .fna gene calls to obtain per-gene coverage, microdiversity, and SNP type. Extract biallelic SNV positions across infants with Pandas and classify each as intra-infant (biallelic within a single infant) or inter-infant (substitution between infants); label variants synonymous/nonsynonymous from Prodigal annotations and compute per-gene dN/dS as [(non-synonymous substitutions / non-synonymous sites) / (synonymous substitutions / synonymous sites)] using Numpy/SciPy. Compute per-gene presence/deletion frequency across samples from gene-level coverage, then plot per-gene dN/dS and the within- vs between-infant nonsynonymous change distribution with Matplotlib/Seaborn, highlighting the tail fiber gene. The rationale is that loci diverse within an infant include variants also found between infants, consistent with inoculation from diverse hospital-associated sources.

## Related tools

- **inStrain** (Profiles microdiversity and calls SNVs from filtered read pairs; 'inStrain profile_genes' reports per-gene coverage, microdiversity, and SNP type) — https://github.com/MrOlm/instrain
- **Samtools** (Interacts with .bam/.sam files for read alignment handling during profiling)
- **Prodigal** (Annotates open reading frames of the phage reference to provide gene calls for profile_genes and synonymous/nonsynonymous labeling)
- **Pandas** (Extracts and classifies biallelic SNV positions across infants as intra- vs inter-infant)
- **SciPy** (Supports dN/dS computation with Numpy)
- **Numpy** (Numeric computation of per-gene dN/dS ratios)
- **Matplotlib** (Plots per-gene dN/dS and within- vs between-infant nonsynonymous change distributions)
- **Seaborn** (Styling/visualization of the per-gene dN/dS and nonsynonymous change plots)

## Examples

```
inStrain profile mapping.bam 482_10.ph.fasta -o inStrain_profile1 && inStrain profile_genes inStrain_profile1 gene_calls.fna
```

## Evaluation signals

- Reproduces the reported pattern: nonsynonymous changes in the tail fiber gene are purged within infants yet selected for between infants
- Gene-level coverage and diversity values match the inStrain profile_genes report
- Every called SNV satisfies the default thresholds: >= 5 reads at the position, > 5% of reads carrying the variant base, and null-model FDR of 1e-6
- All retained read pairs map in proper orientation within the expected insert size and meet the minimum read-pair ANI of 0.95
- dN/dS values reproduce the formula [(non-synonymous substitutions / non-synonymous sites) / (synonymous substitutions / synonymous sites)]

## Limitations

- It is difficult to distinguish microbiome diversity that is evolved in situ from that introduced by immigration, so intra- vs inter-infant overlap may reflect hospital inoculation rather than in situ evolution
- Variant sites near the ends of genome fragments can be missed with inStrain
- Un-paired reads are removed by default, and the Python implementation can be slow (changelog notes it should call the node2vec C++ library)

## Evidence

- [readme] README quick start: [evidence span withheld — anchor status: external_doc]
