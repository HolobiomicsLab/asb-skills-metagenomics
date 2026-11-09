---
name: dn-ds-calculation-from-metagenomic-snvs
description: Use when inStrain has called SNVs on read pairs mapped to a reference
  genome and you need per-gene synonymous/nonsynonymous classification and dN/dS to
  compare variation within versus between hosts.
when_to_use_negative:
- Coverage is too low to meet the inStrain SNP criteria (>=5 reads per position, >5%
  variant base, FDR 1e-6); minor-allele detection is coverage-dependent (Supplemental
  Figure S2), so dN/dS from sparse SNVs will be noise
- The goal is strain-level genome comparison (ConANI/popANI) rather than gene-level
  selection statistics; use 'inStrain compare' instead
- No annotated reference exists — synonymous/nonsynonymous assignment requires a reference
  .fasta with Prodigal open reading frame calls
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: inStrain
  role: Profiles each .bam against the reference .fasta and calls SNVs with gene localization
    and synonymous/non-synonymous identification ('profile' and 'profile_genes')
  repo: github.com/MrOlm/instrain
- name: Samtools
  role: Interacts with .bam/.sam files to filter read pairs and compute per-position
    coverage feeding the SNV calls
- name: Prodigal
  role: Annotates open reading frames (.fna gene calls, metagenome mode) used to assign
    SNVs to genes and classify them synonymous vs nonsynonymous
- name: Pandas
  role: Extracts biallelic SNV positions and classifies each variant as intra- vs
    inter-infant from the inStrain SNV tables
- name: Numpy
  role: Arithmetic for per-gene dN/dS from nonsynonymous/synonymous substitution and
    site counts
- name: SciPy
  role: Supports per-gene dN/dS computation over the SNV and gene-annotation tables
- name: Matplotlib
  role: Plots per-gene dN/dS and the within- vs between-infant nonsynonymous change
    distribution
- name: Seaborn
  role: Styling for the Figure-6-style plot highlighting the tail fiber gene
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:6439156f43048b10426133ffa6ec9c0ec541f0faa2c6ea93338ff48e41cf3000
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
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# dn-ds-calculation-from-metagenomic-snvs

## Summary

Computes per-gene dN/dS from biallelic metagenomic SNVs called by inStrain, labeling each variant synonymous or nonsynonymous against Prodigal gene calls to test whether diversity within hosts differs from variation between hosts (e.g., the E. faecalis phage 482_10.ph tail fiber gene, Figure 6).

## When to use

Use when inStrain has called SNVs on read pairs mapped to a reference genome and you need per-gene synonymous/nonsynonymous classification and dN/dS to compare variation within versus between hosts.

## When NOT to use

- Coverage is too low to meet the inStrain SNP criteria (>=5 reads per position, >5% variant base, FDR 1e-6); minor-allele detection is coverage-dependent (Supplemental Figure S2), so dN/dS from sparse SNVs will be noise
- The goal is strain-level genome comparison (ConANI/popANI) rather than gene-level selection statistics; use 'inStrain compare' instead
- No annotated reference exists — synonymous/nonsynonymous assignment requires a reference .fasta with Prodigal open reading frame calls

## Inputs

- Sorted .bam files of metagenomic read pairs mapped to the reference genome .fasta (E. faecalis phage 482_10.ph)
- Reference genome .fasta
- Prodigal gene calls (.fna) for the reference genome (metagenome mode)
- inStrain profile SNV table and inStrain profile_genes gene-level report

## Outputs

- Per-gene dN/dS values
- SNV table with biallelic positions classified intra- vs inter-infant and synonymous vs nonsynonymous
- Per-gene coverage, microdiversity, and SNP type from inStrain profile_genes
- Per-gene presence/deletion frequency across infant samples
- Matplotlib/Seaborn figure of per-gene dN/dS and within- vs between-infant nonsynonymous changes (tail fiber highlighted)

## How to apply

Run 'inStrain profile' on each infant sample's sorted .bam mapped to the phage 482_10.ph .fasta, keeping default read-pair filters (proper orientation, expected insert size, minimum read-pair ANI 0.95) and default SNP thresholds (minimum 5 reads per position, >5% variant base, FDR 1e-6) so only high-confidence biallelic SNVs enter the analysis. Run 'inStrain profile_genes' with the Prodigal gene calls (run in metagenome mode) to obtain per-gene coverage, microdiversity, and SNP type (synonymous vs nonsynonymous). In Pandas, extract biallelic SNV positions across infants and classify each as intra-infant (biallelic within a single infant) or inter-infant (substitution between infants); exclude genes with significant coverage differences across samples to avoid confounding. Compute per-gene dN/dS as [(non-synonymous substitutions / non-synonymous sites) / (synonymous substitutions / synonymous sites)] with Numpy/SciPy, labeling each variant from the Prodigal annotations, and derive per-gene presence/deletion frequency across infant samples from the gene-level coverage data. Plot per-gene dN/dS and the within- vs between-infant nonsynonymous change distribution (tail fiber gene highlighted) with Matplotlib/Seaborn. Validation: the output should reproduce the reported pattern that nonsynonymous tail fiber changes are purged within infants yet selected for between infants, with gene-level values matching the inStrain profile_genes report.

## Related tools

- **inStrain** (Profiles each .bam against the reference .fasta and calls SNVs with gene localization and synonymous/non-synonymous identification ('profile' and 'profile_genes')) — github.com/MrOlm/instrain
- **Samtools** (Interacts with .bam/.sam files to filter read pairs and compute per-position coverage feeding the SNV calls)
- **Prodigal** (Annotates open reading frames (.fna gene calls, metagenome mode) used to assign SNVs to genes and classify them synonymous vs nonsynonymous)
- **Pandas** (Extracts biallelic SNV positions and classifies each variant as intra- vs inter-infant from the inStrain SNV tables)
- **Numpy** (Arithmetic for per-gene dN/dS from nonsynonymous/synonymous substitution and site counts)
- **SciPy** (Supports per-gene dN/dS computation over the SNV and gene-annotation tables)
- **Matplotlib** (Plots per-gene dN/dS and the within- vs between-infant nonsynonymous change distribution)
- **Seaborn** (Styling for the Figure-6-style plot highlighting the tail fiber gene)

## Examples

```
inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1
```

## Evaluation signals

- Gene-level coverage and microdiversity values used for dN/dS match the inStrain profile_genes report
- Every included SNV is biallelic and passes the inStrain criteria: at least 5 reads at the position, >5% variant base, FDR 1e-6
- Each per-gene dN/dS equals [(non-synonymous substitutions / non-synonymous sites) / (synonymous substitutions / synonymous sites)] computed from Prodigal-annotated variants
- Reproduced figure shows nonsynonymous tail fiber changes depleted within infants but enriched between infants (Figure 6 pattern)
- Genes with significant coverage differences across infant samples are excluded from the comparison

## Limitations

- In situ evolution cannot be distinguished from immigration, so within- vs between-infant dN/dS differences may reflect inoculation from diverse hospital-associated sources rather than selection alone
- Variant sites near the ends of genome fragments can be missed with inStrain, potentially omitting SNVs near gene/genome edges
- The complete SNV list needed to redo the analysis is only available as Supplemental Table S3, not in the main text

## Evidence
