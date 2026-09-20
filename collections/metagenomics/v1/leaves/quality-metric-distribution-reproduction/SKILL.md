---
name: quality-metric-distribution-reproduction
description: Use when a paper reports counts of genomes in quality tiers (e.g. medium-quality/high-quality/near-complete)
  and you must independently recompute and verify those counts from the deposited
  genome FASTAs.
when_to_use_negative:
- Input genomes are viral or fungal — the article uses CheckV for viral completeness
  and BUSCO (fungi thresholds C:>50%, D:<5%) for fungal bins, not CheckM2
- The deposited annotation file (e.g. VMGC_prokaryote_MAG.info) already provides the
  Quality labels and you only need to read them rather than recompute them
- Your goal is to re-derive quality classes from scratch under your own thresholds
  rather than reproduce the paper's revised-MIMAG criteria (>=50% completeness, <5%
  contamination)
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3697
tools:
- name: CheckM2
  role: Predicts per-genome Completeness, Contamination and completeness model for
    the 19,542 prokaryotic MAGs
  repo: https://zenodo.org/records/10457006
- name: GNU parallel 20201122
  role: Parallelizes the CheckM2 predict run across all MAG FASTA files
- name: GUNC
  role: Flags bins with discordant taxonomic gene assignments as potential chimeras
    before quality classification
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:14d44fb19faa5fc4e7572dd53c103c4f1ce90d2da57705c064791e7214298b37
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CheckM2
  - GNU parallel 20201122
  - GUNC
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# quality-metric-distribution-reproduction

## Summary

Reproduce a published genome-quality classification distribution (here, the VMGC's 19,542 prokaryotic genomes) by rerunning CheckM2 quality estimation over downloaded MAG FASTAs, applying revised-MIMAG quality tiers, and comparing per-class counts against the paper's reported figures.

## When to use

Use when a paper reports counts of genomes in quality tiers (e.g. medium-quality/high-quality/near-complete) and you must independently recompute and verify those counts from the deposited genome FASTAs.

## When NOT to use

- Input genomes are viral or fungal — the article uses CheckV for viral completeness and BUSCO (fungi thresholds C:>50%, D:<5%) for fungal bins, not CheckM2
- The deposited annotation file (e.g. VMGC_prokaryote_MAG.info) already provides the Quality labels and you only need to read them rather than recompute them
- Your goal is to re-derive quality classes from scratch under your own thresholds rather than reproduce the paper's revised-MIMAG criteria (>=50% completeness, <5% contamination)

## Inputs

- Prokaryotic MAG FASTA files (n=19,542) from VMGC Zenodo record 10457006 (VMGC_prokaryote_MAG.tar.gz)
- CheckM2 v1.0.1 installation and a DIAMOND/attribute database
- GUNC for chimerism filtering
- GNU parallel 20201122 for parallelization

## Outputs

- checkm2_quality_report.tsv (per-genome Completeness, Contamination, completeness model)
- GUNC chimerism flags per genome bin
- Quality-class assignment per genome (medium-quality / high-quality / near-complete)
- Summary of class counts for comparison with Fig. 1b (10,127 / 8,397 / 1,017)

## How to apply

Download the prokaryotic MAG FASTA files (n=19,542, VMGC_prokaryote_MAG.tar.gz) from Zenodo record 10457006. Run CheckM2 v1.0.1 'predict' over all FASTAs (parallelized with GNU parallel) to obtain per-genome Completeness, Contamination, and the completeness model used; additionally run GUNC chimerism filtering to flag bins with discordant taxonomic gene assignments as potential chimeras. Apply the paper's revised-MIMAG criteria: retain genomes with >=50% completeness and <5% contamination, then bin into medium-quality / high-quality / near-complete tiers. Aggregate class counts and compare against the reported distribution (10,127 medium-quality, 8,397 high-quality, 1,017 near-complete), writing a per-genome checkm2_quality_report.tsv and a class-count summary.

## Related tools

- **CheckM2** (Predicts per-genome Completeness, Contamination and completeness model for the 19,542 prokaryotic MAGs) — https://zenodo.org/records/10457006
- **GNU parallel 20201122** (Parallelizes the CheckM2 predict run across all MAG FASTA files)
- **GUNC** (Flags bins with discordant taxonomic gene assignments as potential chimeras before quality classification)

## Examples

```
find VMGC_prokaryote_MAG/*.fasta | parallel -j 16 checkm2 predict --input {} --output checkm2_out/{/.} --threads 8
```

## Evaluation signals

- Total genomes passing the >=50% completeness / <5% contamination filter equals 19,542 (18,570 MAGs + 972 isolated genomes)
- Per-class counts match the reported distribution: 10,127 medium-quality, 8,397 high-quality, 1,017 near-complete
- checkm2_quality_report.tsv has one row per input genome with Completeness in [0,100], Contamination >= 0, and a non-empty completeness model column
- Genomes flagged as potential chimeras by GUNC chimerism filtering are excluded or annotated before class assignment
- Class-count totals sum to the number of retained genomes (10,127 + 8,397 + 1,017 = 19,541; any off-by-one traced to filter boundary handling)

## Limitations

- CheckM2 completeness may be underestimated for some taxa — the paper notes 'the completeness of A. indicium and A. nucleatus might be underestimated in the current CheckM2 algorithm'
- Similar assessment results were only partially corroborated by CheckM and BUSCO (124 USCOs for prokaryotes), so alternative estimators may disagree
- Public vaginal metagenome datasets may carry contamination from experimental operations or kits, affecting contamination estimates

## Evidence

- [abstract] 10,127, 8,397 and 1,017 were classified as medium-quality, high-quality and near-complete genomes, respectively: "10,127, 8,397 and 1,017 were classified as medium-quality, high-quality and near-complete genomes, respectively"
- [abstract] yielded 18,570 prokaryotic genomes with >=50% completeness and <5% contamination: "yielded 18,570 prokaryotic genomes with ≥50% completeness and <5% contamination"
- [abstract] the completeness of A. indicium and A. nucleatus might be underestimated in the current CheckM2 algorithm: "the completeness of A. indicium and A. nucleatus might be underestimated in the current CheckM2 algorithm"
- [readme] Full prokaryotic genomes (n=19,542) | 8.1 GB | VMGC_prokaryote_MAG.tar.gz: "Full prokaryotic genomes (n=19,542)  | 8.1 GB | VMGC_prokaryote_MAG.tar.gz"
- [abstract] Similar assessment results were also revealed by other algorithms such as CheckM and BUSCO: "Similar assessment results were also revealed by other algorithms such as CheckM and BUSCO"
