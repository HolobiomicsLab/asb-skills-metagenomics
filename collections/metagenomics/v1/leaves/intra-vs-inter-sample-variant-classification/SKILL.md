---
name: intra-vs-inter-sample-variant-classification
description: Use when mapping-based SNV calls from inStrain profiles across multiple
  samples must be partitioned into within-host vs between-host variants and analyzed
  for selection (dN/dS) at the gene level.
when_to_use_negative:
- 'Genome-level strain sharing rather than SNV-level diversity: use ''inStrain compare''
  for ConANI/popANI strain comparisons instead'
- Reference-free variant discovery without a mapped reference .fasta and gene annotations
  — this skill requires bams mapped to a specific genome plus Prodigal ORF calls
- Read filtering or coverage breadth estimation as the primary goal (e.g. use CoverM
  for coverage calculation instead)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0194
tools:
- name: inStrain
  role: Profiles microdiversity and calls SNVs from sorted .bam mappings, with read-pair
    filters and default SNP thresholds (min_cov 5, min_freq 0.05, FDR 1e-6); profile_genes
    gives per-gene SNP type
  repo: github.com/MrOlm/instrain
- name: Samtools
  role: Required by inStrain for interacting with .bam and .sam files and for coverage
    calculation per position
- name: Prodigal
  role: Annotates open reading frames (metagenome mode) providing the gene calls needed
    for per-gene SNP typing and synonymous/nonsynonymous classification
- name: Pandas
  role: Dataframe handling for extracting and classifying biallelic SNV positions
    across samples as intra- vs inter-sample
- name: SciPy
  role: Statistical support for the dN/dS computation and downstream tests
- name: Numpy
  role: Numeric computation for per-gene dN/dS and site counts
- name: Matplotlib
  role: Plotting per-gene dN/dS and nonsynonymous change distributions
- name: Seaborn
  role: Statistical visualization styling for the per-gene dN/dS and variant-class
    figures
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:720c1708ffd4f1d63caafb39622e3f121ff65d39445ddfb97949f8a701a1e6dc
schema_version: 0.2.0
metadata:
  license_tier: unknown
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# intra-vs-inter-sample-variant-classification

## Summary

Classifies metagenomic SNVs as intra-sample (biallelic within a single sample, e.g. an infant) versus inter-sample (substitutions between samples), annotates them as synonymous/nonsynonymous from Prodigal gene calls, and computes per-gene dN/dS to test whether diversity is evolved in situ or shared between hosts.

## When to use

Use when mapping-based SNV calls from inStrain profiles across multiple samples must be partitioned into within-host vs between-host variants and analyzed for selection (dN/dS) at the gene level.

## When NOT to use

- Genome-level strain sharing rather than SNV-level diversity: use 'inStrain compare' for ConANI/popANI strain comparisons instead
- Reference-free variant discovery without a mapped reference .fasta and gene annotations — this skill requires bams mapped to a specific genome plus Prodigal ORF calls
- Read filtering or coverage breadth estimation as the primary goal (e.g. use CoverM for coverage calculation instead)

## Inputs

- Sorted .bam files of reads from each sample (e.g. infant metagenomes) mapped to a reference genome .fasta (e.g. phage 482_10.ph)
- Reference genome .fasta for the population of interest
- Prodigal gene annotations (.fna gene calls / ORF predictions) for the reference genome
- inStrain profile and profile_genes report tables (SNV, gene coverage, microdiversity)

## Outputs

- Table of SNV positions classified as intra-sample (biallelic within one sample) or inter-sample (between samples)
- Per-variant synonymous/nonsynonymous labels derived from Prodigal gene annotations
- Per-gene dN/dS values computed as [(non-synonymous substitutions / non-synonymous sites) / (synonymous substitutions / synonymous sites)]
- Per-gene presence/deletion frequency across samples from gene-level coverage
- Plots of per-gene dN/dS and within- vs between-sample nonsynonymous change distributions (e.g. tail fiber gene highlighted)

## How to apply

Run 'inStrain profile' on each sample's sorted .bam against the reference .fasta (e.g. E. faecalis phage 482_10.ph) using default read-pair filters (proper orientation, expected insert size, minimum read-pair ANI 0.95) and SNP thresholds (min_cov 5, min_freq 0.05, FDR 1e-6), then run 'inStrain profile_genes' with the Prodigal gene calls to get per-gene coverage, microdiversity, and SNP type. Extract biallelic SNV positions across samples with Pandas and label each position as intra-sample (biallelic within one sample) or inter-sample (substitution between samples). Annotate each variant as synonymous or nonsynonymous from the Prodigal annotations and compute per-gene dN/dS as [(non-synonymous substitutions / non-synonymous sites) / (synonymous substitutions / synonymous sites)] using Numpy/SciPy. Also compute per-gene presence/deletion frequency across samples from gene-level coverage. Visualize per-gene dN/dS and the within- vs between-sample nonsynonymous change distribution (highlighting genes of interest such as tail fibers) with Matplotlib/Seaborn.

## Related tools

- **inStrain** (Profiles microdiversity and calls SNVs from sorted .bam mappings, with read-pair filters and default SNP thresholds (min_cov 5, min_freq 0.05, FDR 1e-6); profile_genes gives per-gene SNP type) — github.com/MrOlm/instrain
- **Samtools** (Required by inStrain for interacting with .bam and .sam files and for coverage calculation per position)
- **Prodigal** (Annotates open reading frames (metagenome mode) providing the gene calls needed for per-gene SNP typing and synonymous/nonsynonymous classification)
- **Pandas** (Dataframe handling for extracting and classifying biallelic SNV positions across samples as intra- vs inter-sample)
- **SciPy** (Statistical support for the dN/dS computation and downstream tests)
- **Numpy** (Numeric computation for per-gene dN/dS and site counts)
- **Matplotlib** (Plotting per-gene dN/dS and nonsynonymous change distributions)
- **Seaborn** (Statistical visualization styling for the per-gene dN/dS and variant-class figures)

## Examples

```
inStrain profile mapping.bam 482_10.ph.fasta -o inStrain_profile1
```

## Evaluation signals

- Nonsynonymous changes in the tail fiber gene are purged within infants yet selected for between infants, reproducing the reported Figure 6 pattern
- All called SNVs satisfy the inStrain SNP-calling thresholds: >= 5 reads mapping to the position, > 5% of reads carrying the variant base, and variant reads exceeding the null model at FDR 1e-6
- Per-gene coverage, microdiversity, and SNP type values match the inStrain profile_genes report tables
- Every SNV position is assigned to exactly one class (intra-sample biallelic or inter-sample substitution) with no unclassified positions
- Per-gene presence/deletion frequencies derived from gene-level coverage agree with the gene coverage table across all infant samples

## Limitations

- It is difficult to distinguish microbiome diversity that is evolved in situ from that introduced by immigration (inoculation from diverse hospital-associated sources)
- Variant sites near the ends of genome fragments can be missed with inStrain
- All un-paired reads are removed by default and filters are applied to read pairs, so single-end or improperly paired mappings yield no SNV calls

## Evidence

- [readme] inStrain profile quick start command: [evidence span withheld — anchor status: external_doc]
