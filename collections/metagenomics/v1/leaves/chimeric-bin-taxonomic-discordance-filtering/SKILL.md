---
name: chimeric-bin-taxonomic-discordance-filtering
description: Use when a MAG catalogue (e.g., 19,542 VMGC prokaryotic FASTA genomes)
  has per-genome CheckM2 Completeness/Contamination estimates and bins must be screened
  for taxonomic discordance (chimerism) prior to quality classification.
when_to_use_negative:
- Input is viral contigs — the VMGC pipeline instead uses CheckV to classify the 14,224
  viral sequences as complete/high/medium quality
- Input is fungal/eukaryotic bins — the VMGC pipeline applies BUSCO-based thresholds
  (C:>50%, D:<5%) rather than CheckM2+GUNC
- You only need taxonomy assignment of nonredundant species bins (786 SGBs) — use
  the Kraken2/Bracken customized-database workflow instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CheckM2
  role: Produces per-genome Completeness/Contamination estimates and the completeness
    model that are the input to GUNC chimerism filtering
- name: GUNC
  role: Chimerism filter applied over CheckM2 output to flag bins with discordant
    taxonomic gene assignments as potential chimeras
- name: GNU parallel
  role: Parallelizes CheckM2 predict across the 19,542 MAG FASTA files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:d621bdbcecf3798d541372a67d9c9c85f06587f0af4364de13c0797ecee91737
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CheckM2
  - GNU parallel
  - GUNC
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Chimeric bin taxonomic-discordance filtering

## Summary

Flags metagenome-assembled bins whose genes show discordant taxonomic assignments (potential chimeras) by running GUNC chimerism filtering over CheckM2 output, before assigning MIMAG-revised quality classes to the 19,542 VMGC prokaryotic genomes.

## When to use

Use when a MAG catalogue (e.g., 19,542 VMGC prokaryotic FASTA genomes) has per-genome CheckM2 Completeness/Contamination estimates and bins must be screened for taxonomic discordance (chimerism) prior to quality classification.

## When NOT to use

- Input is viral contigs — the VMGC pipeline instead uses CheckV to classify the 14,224 viral sequences as complete/high/medium quality
- Input is fungal/eukaryotic bins — the VMGC pipeline applies BUSCO-based thresholds (C:>50%, D:<5%) rather than CheckM2+GUNC
- You only need taxonomy assignment of nonredundant species bins (786 SGBs) — use the Kraken2/Bracken customized-database workflow instead

## Inputs

- Prokaryotic MAG FASTA files (VMGC_prokaryote_MAG.tar.gz, n=19,542, Zenodo record 10457006)
- CheckM2 v1.0.1 quality report (per-genome Completeness, Contamination, completeness model)

## Outputs

- GUNC chimerism flag per MAG (discordant taxonomic gene assignments marked as potential chimeras)
- checkm2_quality_report.tsv — per-genome quality table with Completeness, Contamination and chimerism status
- Summary of quality-class counts (10,127 medium-quality, 8,397 high-quality, 1,017 near-complete)

## How to apply

Run CheckM2 v1.0.1 'predict' over all prokaryotic MAG FASTA files (the VMGC MAGs downloaded from Zenodo record 10457006, n=18,570 MAGs plus 972 isolated genomes), parallelized with GNU parallel, to obtain per-genome Completeness, Contamination and the completeness model used. Feed the CheckM2 output into GUNC chimerism filtering so bins whose taxonomic gene assignments are discordant are flagged as potential chimeras rather than silently retained. Retain only genomes passing the revised-MIMAG screen of ≥50% completeness and <5% contamination, then tier the survivors into near-complete, high-quality and medium-quality classes. Rationale: chimera detection prevents a composite bin (e.g., bacterial plus host or kit contaminant contigs) from earning an inflated completeness score; the VMGC paper notes completeness for some taxa (A. indicium, A. nucleatus) can be underestimated by CheckM2, so GUNC discordance is an independent sanity check on bin integrity.

## Related tools

- **CheckM2** (Produces per-genome Completeness/Contamination estimates and the completeness model that are the input to GUNC chimerism filtering)
- **GUNC** (Chimerism filter applied over CheckM2 output to flag bins with discordant taxonomic gene assignments as potential chimeras)
- **GNU parallel** (Parallelizes CheckM2 predict across the 19,542 MAG FASTA files)

## Examples

```
checkm2 predict --input VMGC_MAGs/*.fasta --threads 50 --output-directory checkm2_results  # then run GUNC chimerism filtering on checkm2_results/quality_report.tsv
```

## Evaluation signals

- After GUNC filtering and the ≥50% completeness / <5% contamination screen, class counts should reproduce the reported Fig. 1b distribution: 10,127 medium-quality, 8,397 high-quality, 1,017 near-complete
- No genome below 50% completeness or with ≥5% contamination appears in the retained quality-class table
- checkm2_quality_report.tsv contains one row per input MAG (18,570 MAGs + 972 isolated genomes = 19,542) with Completeness, Contamination and chimerism columns populated
- Bins flagged by GUNC as chimeric are excluded (or annotated) before MIMAG-revised tier assignment, not counted among the 19,542 classified genomes

## Limitations

- CheckM2 completeness can be underestimated for some taxa — the paper notes 'the completeness of A. indicium and A. nucleatus might be underestimated in the current CheckM2 algorithm'
- Public vaginal metagenomic datasets may carry contamination from experimental operations or kits, which can contaminate bins and confound completeness/contamination estimates
- Similar CheckM2-based assessments were cross-checked with CheckM and BUSCO (124 USCOs for prokaryotes), so single-algorithm results should not be treated as definitive

## Evidence

- [other] Apply GUNC chimerism filtering over the CheckM2 output to flag bins with discordant taxonomic gene assignments as potential chimeras.: [evidence span withheld — anchor status: not_found]
- [abstract] yielded 18,570 prokaryotic genomes with ≥50% completeness and <5% contamination: "yielded 18,570 prokaryotic genomes with ≥50% completeness and <5% contamination"
- [abstract] following the criteria revised from the MIMAG (Minimum Information about a Metagenome-Assembled Genome) standard: "following the criteria revised from the MIMAG (Minimum Information about a Metagenome-Assembled Genome) standard17"
- [abstract] the completeness of A. indicium and A. nucleatus might be underestimated in the current CheckM2 algorithm: "the completeness of A. indicium and A. nucleatus might be underestimated in the current CheckM2 algorithm31"
- [discussion] Another concern is contamination from experimental operations or kits in public vaginal metagenome datasets: "Another concern is contamination from experimental operations or kits in public vaginal metagenome datasets used for constructing the VMGC"
