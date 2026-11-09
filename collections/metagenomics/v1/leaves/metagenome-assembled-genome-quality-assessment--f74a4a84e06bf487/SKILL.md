---
name: metagenome-assembled-genome-quality-assessment
description: Use when a collection of MAG FASTA files (e.g. 19,542 VMGC prokaryotic
  genomes from Zenodo record 10457006) must be scored for completeness/contamination
  and assigned MIMAG-revised quality tiers.
when_to_use_negative:
- Input is viral genome sequences — use CheckV instead (viral genomes were assessed
  at ≥50% completeness via CheckV, not CheckM2)
- Input is fungal/eukaryotic bins — the VMGC pipeline used BUSCO with thresholds C:>50%
  and D:<5% for fungi, not CheckM2
- Input genomes are already annotated with quality values in VMGC_prokaryote_MAG.info
  and the task is only to read the tiers, not recompute them
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: CheckM2
  role: Runs 'predict' to estimate per-genome Completeness, Contamination and the
    completeness model used for the 19,542 VMGC prokaryotic genomes
- name: GNU parallel 20201122
  role: Parallelizes CheckM2 'predict' over the large batch of MAG FASTA files
  repo: https://github.com/WatsonLab/single_and_multiple_binning
- name: GUNC
  role: Chimerism filtering over CheckM2 output to flag bins with discordant taxonomic
    gene assignments
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:904067742b83003e1748595042e08414ceb07ae381ad89b0605fd7ea3532a77f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WatsonLab/single_and_multiple_binning
  tools:
  - CheckM2
  - GNU parallel 20201122
  - GUNC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metagenome-assembled-genome-quality-assessment

## Summary

Assesses the quality of metagenome-assembled genomes (MAGs) by running CheckM2 v1.0.1 'predict' to estimate per-genome Completeness and Contamination, then classifying each genome into near-complete / high-quality / medium-quality tiers under criteria revised from the MIMAG standard, retaining only genomes with ≥50% completeness and <5% contamination.

## When to use

Use when a collection of MAG FASTA files (e.g. 19,542 VMGC prokaryotic genomes from Zenodo record 10457006) must be scored for completeness/contamination and assigned MIMAG-revised quality tiers.

## When NOT to use

- Input is viral genome sequences — use CheckV instead (viral genomes were assessed at ≥50% completeness via CheckV, not CheckM2)
- Input is fungal/eukaryotic bins — the VMGC pipeline used BUSCO with thresholds C:>50% and D:<5% for fungi, not CheckM2
- Input genomes are already annotated with quality values in VMGC_prokaryote_MAG.info and the task is only to read the tiers, not recompute them

## Inputs

- MAG FASTA files (VMGC_prokaryote_MAG.tar.gz, n=19,542; 18,570 MAGs and 972 isolated genomes) from https://zenodo.org/records/10457006
- Per-genome annotation table VMGC_prokaryote_MAG.info (Source/Quality/Clustering) for cross-checking assigned quality

## Outputs

- checkm2_quality_report.tsv with per-genome Completeness, Contamination and completeness model columns
- GUNC chimerism flags per genome
- Per-genome quality class assignments (medium-quality / high-quality / near-complete)
- Summary of tier counts for comparison with Fig. 1b

## How to apply

Download the prokaryotic MAG FASTA files from the VMGC Zenodo record 10457006 and run CheckM2 v1.0.1 'predict' over all FASTAs, parallelized with GNU parallel, to obtain per-genome Completeness, Contamination and the completeness model used. Apply GUNC chimerism filtering over the CheckM2 output to flag bins with discordant taxonomic gene assignments as potential chimeras. Then assign quality classes using criteria revised from the MIMAG standard: retain genomes with ≥50% completeness and <5% contamination, and bin genomes into near-complete / high-quality / medium-quality tiers. Aggregate the tier counts and compare them against the published Fig. 1b distribution (10,127 medium-quality, 8,397 high-quality, 1,017 near-complete) as a reproduction check. Write the per-genome results to checkm2_quality_report.tsv plus a class-count summary.

## Related tools

- **CheckM2** (Runs 'predict' to estimate per-genome Completeness, Contamination and the completeness model used for the 19,542 VMGC prokaryotic genomes)
- **GNU parallel 20201122** (Parallelizes CheckM2 'predict' over the large batch of MAG FASTA files) — https://github.com/WatsonLab/single_and_multiple_binning
- **GUNC** (Chimerism filtering over CheckM2 output to flag bins with discordant taxonomic gene assignments)

## Examples

```
find VMGC_prokaryote_MAG/*.fa | parallel -j 10 checkm2 predict --input {} --output results/{/.} --threads 20 --extension fa
```

## Evaluation signals

- Per-genome report contains Completeness, Contamination and completeness-model columns for all 19,542 genomes
- All retained genomes satisfy the ≥50% completeness and <5% contamination filter
- Tier counts reproduce the published distribution: 10,127 medium-quality, 8,397 high-quality, 1,017 near-complete
- GUNC-flagged potential chimeras are recorded and excluded or annotated before tier assignment
- checkm2_quality_report.tsv is produced alongside a summary of class counts

## Limitations

- Completeness of some organisms may be systematically biased — the article notes 'the completeness of A. indicium and A. nucleatus might be underestimated in the current CheckM2 algorithm'
- CheckM2-based estimates were corroborated with CheckM and BUSCO (based on 124 universal single-copy orthologs (USCOs) for prokaryotes), which are cited as giving similar results but may diverge
- The ≥50% completeness / <5% contamination cutoffs follow MIMAG-revised criteria and may exclude genomes a different standard would retain

## Evidence

- [abstract] 19,542 prokaryotic genomes classified as 10,127 medium-quality, 8,397 high-quality and 1,017 near-complete: "10,127, 8,397 and 1,017 were classified as medium-quality, high-quality and near-complete genomes, respectively"
- [abstract] Prokaryotic genome quality filtering thresholds: "yielded 18,570 prokaryotic genomes with ≥50% completeness and <5% contamination"
- [abstract] MIMAG-revised quality criteria: "following the criteria revised from the MIMAG (Minimum Information about a Metagenome-Assembled Genome) standard17"
- [abstract] CheckM2 limitation on underestimated completeness: "the completeness of A. indicium and A. nucleatus might be underestimated in the current CheckM2 algorithm31"
- [readme] Data source for MAG FASTA files: "The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006"
