---
name: microdiversity-profiling-of-mapped-reads
description: Use when you have coReads .bam files mapped to representative sub-species
  .fasta references and need per-genome SNV calling, nucleotide diversity, clonality,
  and breadth for population-genome comparisons across sample groups.
when_to_use_negative:
- You only need read coverage/relative abundance without microdiversity — CoverM (coverm
  genome/contig) is a faster coverage-only alternative
- Your goal is dereplication of a genome set rather than profiling mapped-read microdiversity
  — use dRep dereplicate instead
- Reads are mapped to many highly similar (>98% ANI) references without dereplication,
  risking multi-mapped reads inflating diversity estimates
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: inStrain
  role: 'Profiles microdiversity: read filtering of .bam files, per-position coverage
    via Samtools, SNV calling, nucleotide diversity, clonality, and genome_wide aggregation'
  repo: https://github.com/MrOlm/instrain
- name: Samtools
  role: inStrain dependency for interacting with .bam and .sam files and calculating
    position coverage
- name: Pandas
  role: Filter sub-species by >= 0.5 unmaskedBreadth, exclude likely contaminants,
    and group nucleotide diversity by birth mode
- name: SciPy
  role: Statistical test of the birth-mode difference in Klebsiella nucleotide diversity
- name: Matplotlib
  role: Render the diversity comparison plot
- name: Seaborn
  role: Render the diversity comparison plot
- name: Numpy
  role: Numerical handling of diversity metrics during filtering and plotting
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:f7f587ed7545b24f01e004a0335d252873bad6aaaa12a13bb43cb6b02a1d3478
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MrOlm/instrain
  tools:
  - Matplotlib
  - Numpy
  - Pandas
  - Samtools
  - SciPy
  - Seaborn
  - inStrain
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# microdiversity-profiling-of-mapped-reads

## Summary

Profile strain-level microdiversity (SNVs, nucleotide diversity, clonality, coverage breadth) from metagenomic read mappings to reference genomes using inStrain, then aggregate scaffold-level metrics to genome level and test group differences (e.g., Klebsiella nucleotide diversity by birth mode).

## When to use

Use when you have coReads .bam files mapped to representative sub-species .fasta references and need per-genome SNV calling, nucleotide diversity, clonality, and breadth for population-genome comparisons across sample groups.

## When NOT to use

- You only need read coverage/relative abundance without microdiversity — CoverM (coverm genome/contig) is a faster coverage-only alternative
- Your goal is dereplication of a genome set rather than profiling mapped-read microdiversity — use dRep dereplicate instead
- Reads are mapped to many highly similar (>98% ANI) references without dereplication, risking multi-mapped reads inflating diversity estimates

## Inputs

- coReads .bam files (paired reads mapped to concatenated representative sub-species .fasta)
- Concatenated representative sub-species .fasta reference file
- Scaffold-to-bin mapping file or list of individual .fasta files for genome_wide aggregation
- Sample metadata table (e.g., birth mode: cesarean vs vaginal)
- Negative extraction control profiles and sampling campaign labels for contaminant filtering

## Outputs

- inStrain profile output folder with per-scaffold coverage, clonality, SNV calls, and nucleotide diversity
- Genome-level (sub-species-level) microdiversity table from inStrain genome_wide
- Filtered nucleotide diversity values for target sub-species (e.g., Klebsiella) grouped by birth mode
- Statistical test result and matplotlib/seaborn diversity comparison plot (e.g., Figure 4 / Supplemental Figure S6 style)

## How to apply

Run inStrain profile on each coReads .bam against the concatenated representative sub-species .fasta with default read filters (min read-pair ANI -l 0.95, min_cov 5, min_freq 0.05, SNP FDR 1e-6) to compute per-scaffold coverage, clonality, and SNV-based nucleotide diversity; inStrain relies on Samtools for .bam/.sam interaction. Then run inStrain genome_wide (-i <inStrain_folder> -s <scaffold_to_bin>) to aggregate scaffold-level results into sub-species-level microdiversity metrics. In pandas, filter detected sub-species to those with >= 0.5 unmaskedBreadth and exclude 'likely contaminant' genomes (detected in the negative extraction control or significantly enriched in one sampling campaign). Extract nucleotide diversity values for the target genus (e.g., Klebsiella), group by the condition of interest (cesarean vs vaginal delivery) with pandas, and test the difference with a SciPy statistical test before rendering the comparison with matplotlib/seaborn.

## Related tools

- **inStrain** (Profiles microdiversity: read filtering of .bam files, per-position coverage via Samtools, SNV calling, nucleotide diversity, clonality, and genome_wide aggregation) — https://github.com/MrOlm/instrain
- **Samtools** (inStrain dependency for interacting with .bam and .sam files and calculating position coverage)
- **Pandas** (Filter sub-species by >= 0.5 unmaskedBreadth, exclude likely contaminants, and group nucleotide diversity by birth mode)
- **SciPy** (Statistical test of the birth-mode difference in Klebsiella nucleotide diversity)
- **Matplotlib** (Render the diversity comparison plot)
- **Seaborn** (Render the diversity comparison plot)
- **Numpy** (Numerical handling of diversity metrics during filtering and plotting)

## Examples

```
inStrain profile coReads_sample.bam subspecies_representatives.fasta -o inStrain_profile_sample; inStrain genome_wide -i inStrain_profile_sample -s scaffold_to_bin.tsv
```

## Evaluation signals

- Reproduced figure shows Klebsiella with higher nucleotide diversity in C-section-born infants than in vaginally born infants (Figure 4 / Supplemental Figure S6 result)
- All analyzed sub-species pass the >= 0.5 unmaskedBreadth detection threshold; sub-species detected in the negative extraction control or enriched in one sampling campaign are excluded as 'likely contaminant'
- inStrain profile ran with default read filters confirmed: min read-pair ANI -l 0.95, min_cov 5, min_freq 0.05, SNP FDR 1e-6; unpaired reads removed and SNVs require >= 5 reads with > 5% variant base
- genome_wide output contains genome-level microdiversity metrics aggregating the per-scaffold profile results for every input inStrain folder

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain
- Genome de-replication at 98% ANI is needed to reduce multi-mapped reads
- It is difficult to distinguish microbiome diversity that evolved in situ from diversity introduced by immigration, complicating interpretation of birth-mode diversity differences
- The association between Klebsiella and birth mode was re-evaluated when excluding samples from the Sloan2 and NIH4 campaigns (potential campaign confounding)

## Evidence

- [readme] CLI usage: [evidence span withheld — anchor status: external_doc]
