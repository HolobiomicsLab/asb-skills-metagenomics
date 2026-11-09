---
name: subspecies-detection-thresholding
description: Use when metagenomic reads have been mapped to concatenated representative
  sub-species FASTAs and you must decide which sub-species count as detected per sample
  before comparing metrics (e.g. nucleotide diversity) across groups such as birth
  mode.
when_to_use_negative:
- Reads have not yet been mapped to representative sub-species genomes (no .bam available)
  — profile with inStrain first
- The reference genome set has not been de-replicated near 98% ANI — multi-mapped
  reads will distort coverage and diversity estimates
- The question is strain sharing or ANI comparison between samples rather than per-sample
  presence/abundance of sub-species
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: inStrain
  role: profiles .bam files for coverage, clonality, and SNV-based nucleotide diversity;
    genome_wide aggregates scaffold metrics to sub-species level; unmaskedBreadth
    supports the >= 0.5 detection threshold
  repo: github.com/MrOlm/instrain
- name: Samtools
  role: used by inStrain to interact with .bam/.sam files and compute per-position
    coverage on filtered reads
- name: Pandas
  role: filters detected sub-species at the >= 0.5 unmaskedBreadth threshold, excludes
    likely contaminants, and groups nucleotide diversity by birth mode
- name: SciPy
  role: statistical test of the birth-mode difference in Klebsiella nucleotide diversity
    after thresholding
- name: Matplotlib
  role: renders the diversity comparison plot after threshold-based filtering
- name: Seaborn
  role: renders the diversity comparison plot after threshold-based filtering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:66f8bbb5809642fe7ce8b1bce05aa7cabbc4a36e94e57a40a50267acaee7b544
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# subspecies-detection-thresholding

## Summary

Applies breadth-based detection thresholds and contaminant flags to inStrain sub-species profiles, so that only genomes genuinely present (>= 0.5 unmaskedBreadth) in an infant fecal metagenome contribute to microdiversity statistics such as Klebsiella nucleotide diversity.

## When to use

Use when metagenomic reads have been mapped to concatenated representative sub-species FASTAs and you must decide which sub-species count as detected per sample before comparing metrics (e.g. nucleotide diversity) across groups such as birth mode.

## When NOT to use

- Reads have not yet been mapped to representative sub-species genomes (no .bam available) — profile with inStrain first
- The reference genome set has not been de-replicated near 98% ANI — multi-mapped reads will distort coverage and diversity estimates
- The question is strain sharing or ANI comparison between samples rather than per-sample presence/abundance of sub-species

## Inputs

- coReads .bam files mapping infant fecal metagenomic reads to concatenated representative sub-species .fasta files
- scaffold-to-bin mapping or individual sub-species .fasta list for inStrain genome_wide
- negative extraction control sample profiles and per-campaign (e.g. Sloan2, NIH4) detection tables for contaminant flagging

## Outputs

- inStrain profile output folder with per-scaffold coverage, clonality, and SNV-based nucleotide diversity
- genome_wide table of sub-species-level microdiversity metrics filtered to >= 0.5 unmaskedBreadth and de-contaminated
- per-birth-mode nucleotide diversity values and comparison figure (e.g. Klebsiella in cesarean vs vaginally born infants)

## How to apply

First run inStrain profile on each coReads .bam against the concatenated representative sub-species .fasta using default read filters (min read-pair ANI -l 0.95, minimum 5 reads per position, >5% variant-base frequency, SNV FDR 1e-6), then run inStrain genome_wide to aggregate scaffold-level results into genome-level microdiversity metrics. Define detection of a sub-species as that genome being present with >= 0.5 unmaskedBreadth, filtering the genome_wide output table in pandas accordingly. Exclude sub-species flagged as likely contaminants — those detected in the negative extraction control or significantly enriched in one of the six sampling campaigns (e.g. Sloan2, NIH4). Extract nucleotide diversity for the retained sub-species (e.g. Klebsiella) with pandas, group by birth mode (cesarean vs vaginal), and test with scipy before plotting with matplotlib/seaborn. Rationale: filtering reads and requiring breadth-based presence prevents erroneously mapped or low-coverage genomes from inflating diversity values.

## Related tools

- **inStrain** (profiles .bam files for coverage, clonality, and SNV-based nucleotide diversity; genome_wide aggregates scaffold metrics to sub-species level; unmaskedBreadth supports the >= 0.5 detection threshold) — github.com/MrOlm/instrain
- **Samtools** (used by inStrain to interact with .bam/.sam files and compute per-position coverage on filtered reads)
- **Pandas** (filters detected sub-species at the >= 0.5 unmaskedBreadth threshold, excludes likely contaminants, and groups nucleotide diversity by birth mode)
- **SciPy** (statistical test of the birth-mode difference in Klebsiella nucleotide diversity after thresholding)
- **Matplotlib** (renders the diversity comparison plot after threshold-based filtering)
- **Seaborn** (renders the diversity comparison plot after threshold-based filtering)

## Examples

```
inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1
```

## Evaluation signals

- Every retained sub-species in the filtered genome_wide table has unmaskedBreadth >= 0.5
- No retained sub-species is detected in the negative extraction control or enriched in a single sampling campaign (Sloan2, NIH4 exclusion checks pass)
- Detection calls and diversity values are stable when samples from the Sloan2 and NIH4 campaigns are excluded, per the supplementary sensitivity analysis
- Downstream figure reproduces Figure 4 / Supplemental Figure S6: Klebsiella shows higher nucleotide diversity in C-section-born infants than vaginally born infants
- inStrain profile outputs contain per-scaffold coverage, clonality, and SNV metrics computed with the stated defaults (min read-pair ANI 0.95, min_cov 5, min_freq 0.05, FDR 1e-6)

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain
- It is difficult to distinguish microbiome diversity that is evolved in situ from that introduced by immigration, so high nucleotide diversity cannot be causally attributed to within-host evolution
- Genome de-replication at 98% ANI is needed to reduce multi-mapped reads; skipping it undermines coverage- and breadth-based detection

## Evidence
