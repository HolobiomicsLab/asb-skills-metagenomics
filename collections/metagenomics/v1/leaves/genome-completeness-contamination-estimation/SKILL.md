---
name: genome-completeness-contamination-estimation
description: Use when you have a set of metagenome-assembled genomes (FASTA files)
  and need per-genome completeness/contamination estimates plus a defensible quality-tier
  classification (e.g. medium/high/near-complete) before downstream analyses.
when_to_use_negative:
- Genomes are viral — the VMGC used CheckV (e.g. >=50% completeness, 10.3% complete
  viral genomes) rather than CheckM2/MIMAG criteria
- Genomes are fungal — the study applied BUSCO fungi thresholds (C:>50%, D:<5%) instead
  of CheckM2-based prokaryotic quality classes
- You only need taxonomic or abundance profiling of the species-level genome bins
  rather than quality estimation of the underlying bins
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: CheckM2
  role: Runs 'predict' to estimate per-genome Completeness, Contamination and the
    completeness model used for prokaryotic MAGs
- name: GUNC
  role: Chimerism filtering over the CheckM2 output, flagging bins with discordant
    taxonomic gene assignments as potential chimeras
- name: GNU parallel 20201122
  role: Parallelizes the CheckM2 'predict' runs over the 19,542 MAG FASTA files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:0f5d8b134bd18880d44e571c0dc7dc022bd68bf1a634aa5a9b768e26fcf2eb4e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CheckM2
  - GNU parallel 20201122
  - GUNC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# genome-completeness-contamination-estimation

## Summary

Estimate completeness, contamination and the completeness model for MAGs with CheckM2 v1.0.1 'predict', flag potential chimeras with GUNC chimerism filtering, and assign quality classes using criteria revised from the MIMAG standard. Applied in the VMGC study to classify all 19,542 prokaryotic genomes.

## When to use

Use when you have a set of metagenome-assembled genomes (FASTA files) and need per-genome completeness/contamination estimates plus a defensible quality-tier classification (e.g. medium/high/near-complete) before downstream analyses.

## When NOT to use

- Genomes are viral — the VMGC used CheckV (e.g. >=50% completeness, 10.3% complete viral genomes) rather than CheckM2/MIMAG criteria
- Genomes are fungal — the study applied BUSCO fungi thresholds (C:>50%, D:<5%) instead of CheckM2-based prokaryotic quality classes
- You only need taxonomic or abundance profiling of the species-level genome bins rather than quality estimation of the underlying bins

## Inputs

- Prokaryotic MAG FASTA files (n=19,542; 18,570 MAGs and 972 isolated genomes) from VMGC Zenodo record 10457006 (VMGC_prokaryote_MAG.tar.gz)
- Per-genome CheckM2 output table (Completeness, Contamination, completeness model)
- GUNC chimerism assessment output over the CheckM2 results

## Outputs

- Per-genome quality table (checkm2_quality_report.tsv) with Completeness, Contamination, completeness model and assigned quality class
- Summary of quality class counts (near-complete / high-quality / medium-quality) comparable to Fig. 1b
- List of bins flagged as potential chimeras by GUNC

## How to apply

Download the prokaryotic MAG FASTA files (n=19,542, VMGC_prokaryote_MAG.tar.gz from Zenodo record 10457006) and run CheckM2 v1.0.1 'predict' over all FASTA files, parallelized with GNU parallel, to obtain per-genome Completeness, Contamination and the completeness model used. Next, apply GUNC chimerism filtering over the CheckM2 output to flag bins whose discordant taxonomic gene assignments indicate potential chimeras. Then assign a quality class per genome using criteria revised from the MIMAG standard: retain genomes with >=50% completeness and <5% contamination, and bin the rest into near-complete / high-quality / medium-quality tiers. Aggregate the class counts and compare against the reference distribution (10,127 medium-quality, 8,397 high-quality, 1,017 near-complete) to validate the classification.

## Related tools

- **CheckM2** (Runs 'predict' to estimate per-genome Completeness, Contamination and the completeness model used for prokaryotic MAGs)
- **GUNC** (Chimerism filtering over the CheckM2 output, flagging bins with discordant taxonomic gene assignments as potential chimeras)
- **GNU parallel 20201122** (Parallelizes the CheckM2 'predict' runs over the 19,542 MAG FASTA files)

## Examples

```
find VMGC_prokaryote_MAG -name '*.fa' | parallel -j 10 checkm2 predict --input {} --output checkm2_out/{/.} -x fa
```

## Evaluation signals

- Class counts reproduce the reported distribution: 10,127 medium-quality, 8,397 high-quality and 1,017 near-complete of 19,542 prokaryotic genomes
- Retained genomes all satisfy >=50% completeness and <5% contamination (the 18,570 prokaryotic genomes passing the screen)
- checkm2_quality_report.tsv has one row per input FASTA with non-empty Completeness, Contamination and completeness-model columns
- Bins flagged by GUNC chimerism are excluded or annotated before quality-class assignment
- Cross-checks with other algorithms such as CheckM and BUSCO (124 USCOs for prokaryotes) give similar assessment results

## Limitations

- CheckM2 may underestimate completeness for certain taxa — the article notes that 'the completeness of A. indicium and A. nucleatus might be underestimated in the current CheckM2 algorithm'
- Quality tiers follow criteria revised from the MIMAG standard, so counts are not directly comparable to studies using the original MIMAG cutoffs
- Genes lacking similarity to CheckM2 reference models (e.g. proteins without recognizable homologs) can bias completeness estimates for novel lineages

## Evidence

- [abstract] 10,127, 8,397 and 1,017 were classified as medium-quality, high-quality and near-complete genomes, respectively: "10,127, 8,397 and 1,017 were classified as medium-quality, high-quality and near-complete genomes, respectively"
- [abstract] yielded 18,570 prokaryotic genomes with >=50% completeness and <5% contamination: "yielded 18,570 prokaryotic genomes with ≥50% completeness and <5% contamination"
- [abstract] following the criteria revised from the MIMAG (Minimum Information about a Metagenome-Assembled Genome) standard17: "following the criteria revised from the MIMAG (Minimum Information about a Metagenome-Assembled Genome) standard17"
- [readme] Full prokaryotic genomes (n=19,542): "Full prokaryotic genomes (n=19,542)  | 8.1 GB | VMGC_prokaryote_MAG.tar.gz"
- [abstract] the completeness of A. indicium and A. nucleatus might be underestimated in the current CheckM2 algorithm31: "the completeness of A. indicium and A. nucleatus might be underestimated in the current CheckM2 algorithm31"
