---
name: genome-wide-scaffold-aggregation
description: Use when you have inStrain profile output per sample (computed against
  concatenated representative sub-species .fasta references) and need per-sub-species
  genome-level microdiversity metrics (e.g. nucleotide diversity of Klebsiella) for
  group-level comparison such as birth mode.
when_to_use_negative:
- You only need strain-to-strain relatedness between samples (ConANI/popANI) — use
  inStrain compare on two profile objects instead of genome_wide aggregation
- No inStrain profile output exists yet for the .bam/.fasta pair — aggregation requires
  per-scaffold profile results from `inStrain profile` as input
- You only need raw per-scaffold or per-gene coverage — a fast coverage tool like
  CoverM computes this without the SNV/diversity aggregation step
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: inStrain
  role: 'Core tool: `profile` computes scaffold-level coverage, clonality, and SNV-based
    nucleotide diversity from filtered reads; `genome_wide` aggregates scaffolds into
    sub-species-level metrics'
  repo: github.com/MrOlm/instrain
- name: Samtools
  role: Underlying engine inStrain requires for interacting with .bam/.sam files and
    calculating per-position coverage
- name: Pandas
  role: Filter aggregated sub-species rows by the >= 0.5 unmaskedBreadth detection
    threshold, remove likely contaminants, and group diversity values by birth mode
- name: SciPy
  role: Statistical test of the birth-mode difference in aggregated Klebsiella nucleotide
    diversity
- name: Matplotlib
  role: Render the diversity comparison plot from the filtered genome-level table
- name: Seaborn
  role: Statistical plotting of the C-section vs vaginal nucleotide diversity comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:3e797e12388c5cbee4dbfefabff3b01822699ae7e458bdec4235a3836fca5add
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matplotlib
  - Numpy
  - Pandas
  - Samtools
  - SciPy
  - Seaborn
  - inStrain
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# genome-wide-scaffold-aggregation

## Summary

Aggregate scaffold-level inStrain profile results (coverage, clonality, SNV-based nucleotide diversity) into genome-level (sub-species-level) microdiversity metrics using the inStrain genome_wide module, so that individual sub-species populations can be detected, filtered, and statistically compared across sample groups.

## When to use

Use when you have inStrain profile output per sample (computed against concatenated representative sub-species .fasta references) and need per-sub-species genome-level microdiversity metrics (e.g. nucleotide diversity of Klebsiella) for group-level comparison such as birth mode.

## When NOT to use

- You only need strain-to-strain relatedness between samples (ConANI/popANI) — use inStrain compare on two profile objects instead of genome_wide aggregation
- No inStrain profile output exists yet for the .bam/.fasta pair — aggregation requires per-scaffold profile results from `inStrain profile` as input
- You only need raw per-scaffold or per-gene coverage — a fast coverage tool like CoverM computes this without the SNV/diversity aggregation step

## Inputs

- Per-sample coReads .bam files sorted against the concatenated representative sub-species .fasta
- Concatenated representative sub-species .fasta reference (e.g. UHGG-derived genomes concatenated into a single .fasta file)
- inStrain profile output folder(s) containing scaffold-level metrics
- Scaffold-to-bin (scaffold_to_bin) mapping table or list of individual sub-species .fasta files
- Sample metadata table (e.g. birth mode: cesarean vs vaginal) for grouping and statistics

## Outputs

- Genome-level (sub-species-level) inStrain genome_wide metrics table with per-genome coverage, unmaskedBreadth, clonality, and nucleotide diversity
- Filtered sub-species table (unmaskedBreadth >= 0.5, contaminants removed) ready for statistical testing
- Group-level diversity comparison figure (e.g. Klebsiella nucleotide diversity by birth mode) with scipy test results

## How to apply

First run `inStrain profile` on each coReads .bam against the concatenated representative sub-species .fasta with default read filters (min read-pair ANI -l 0.95, min_cov 5, min_freq 0.05, SNP FDR 1e-6) to produce per-scaffold coverage, clonality, and SNV-based nucleotide diversity. Then run `inStrain genome_wide -i <inStrain_folder> -s <scaffold_to_bin or individual .fasta list>` to roll scaffold-level results up into genome-level (sub-species-level) microdiversity metrics using a scaffold-to-bin mapping. Filter detected sub-species in pandas using the >= 0.5 unmaskedBreadth detection threshold, and drop sub-species marked 'likely contaminant' (detected in the negative extraction control or significantly enriched in one of the six sampling campaigns). Finally, group the genome-level nucleotide diversity values by sample metadata (e.g. cesarean vs vaginal birth mode) with pandas, test with a scipy statistical test, and render the comparison with matplotlib/seaborn. Correct application is judged by whether the aggregated table covers each detected sub-species at >= 0.5 unmaskedBreadth and reproduces the expected group-level pattern (here, higher Klebsiella nucleotide diversity in C-section-born infants).

## Related tools

- **inStrain** (Core tool: `profile` computes scaffold-level coverage, clonality, and SNV-based nucleotide diversity from filtered reads; `genome_wide` aggregates scaffolds into sub-species-level metrics) — github.com/MrOlm/instrain
- **Samtools** (Underlying engine inStrain requires for interacting with .bam/.sam files and calculating per-position coverage)
- **Pandas** (Filter aggregated sub-species rows by the >= 0.5 unmaskedBreadth detection threshold, remove likely contaminants, and group diversity values by birth mode)
- **SciPy** (Statistical test of the birth-mode difference in aggregated Klebsiella nucleotide diversity)
- **Matplotlib** (Render the diversity comparison plot from the filtered genome-level table)
- **Seaborn** (Statistical plotting of the C-section vs vaginal nucleotide diversity comparison)

## Examples

```
inStrain genome_wide -i inStrain_profile_folder -s scaffold_to_bin.tsv
```

## Evaluation signals

- Every sub-species retained in the final table has unmaskedBreadth >= 0.5 in at least the samples where it is called detected
- Genome-level metrics in the genome_wide output are consistent with (i.e. aggregate of) the scaffold-level values in the corresponding inStrain profile folder
- No sub-species flagged 'likely contaminant' (present in negative extraction control or enriched in one sampling campaign, e.g. Sloan2 or NIH4) remains in the analyzed set
- Validation figure shows Klebsiella with higher nucleotide diversity in C-section-born infants than in vaginally born infants, reproducing the Figure 4 / Supplemental Figure S6 result
- The birth-mode association remains significant when excluding samples from individual campaigns (Sloan2 and NIH4), confirming robustness of the aggregation-to-statistics pipeline

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain, so genome-level diversity may undercount edge SNVs
- Genomes should be de-replicated at 98% ANI before mapping to reduce multi-mapped reads that would distort aggregated coverage and diversity
- It is difficult to distinguish microdiversity evolved in situ from diversity introduced by immigration, so the aggregated metric does not by itself identify the source of diversity

## Evidence

- [full_text] Run inStrain genome_wide -i <inStrain_folder> -s <scaffold_to_bin or individual .fasta list> to aggregate scaffold-level results into genome-level (sub-species-level) microdiversity metrics.: [evidence span withheld — anchor status: not_found]
- [readme] $ inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1: [evidence span withheld — anchor status: external_doc]
