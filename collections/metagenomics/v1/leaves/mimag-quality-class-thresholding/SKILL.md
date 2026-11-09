---
name: mimag-quality-class-thresholding
description: Use when a set of MAG FASTA files needs completeness/contamination estimation
  and assignment to revised-MIMAG quality tiers (e.g. retaining genomes with >=50%
  completeness and <5% contamination).
when_to_use_negative:
- Input is raw metagenomic reads or unassembled contigs — this skill assumes binned
  MAG FASTA files already exist (see the single-/multi-coverage binning skill for
  the upstream step)
- Assessing viral genomes — the article uses CheckV for viral quality (10.3% complete,
  >=50% completeness filter), not CheckM2/MIMAG thresholds
- Input genomes are already annotated with a quality column (e.g. VMGC_prokaryote_MAG.info
  provides Source/Quality annotations), so re-running assessment is unnecessary
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0797
tools:
- name: CheckM2
  role: Estimates per-genome Completeness and Contamination via 'predict'; note the
    article warns completeness of A. indivicum and A. nucleatum may be underestimated
    by CheckM2
- name: GNU parallel 20201122
  role: Parallelizes CheckM2 prediction and GUNC filtering across the 19,542 genome
    FASTA files
- name: CheckM
  role: Alternative completeness/contamination estimator whose results were cross-checked
    against CheckM2 ('Similar assessment results were also revealed by other algorithms
    such as CheckM and BUSCO')
- name: BUSCO
  role: Alternative assessment based on 124 universal single-copy orthologs (USCOs)
    for prokaryotes, used to corroborate CheckM2 results
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:c3a1a6fa3cec38b5d351802fdf1e2a88c06102e6feeea6a7664aa8d0d414eb3c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BUSCO
  - CheckM
  - CheckM2
  - GNU parallel 20201122
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# mimag-quality-class-thresholding

## Summary

Classifies metagenome-assembled genomes into quality tiers (near-complete, high-quality, medium-quality) by combining CheckM2-estimated Completeness and Contamination with GUNC chimerism flags, using criteria revised from the MIMAG standard as applied to the 19,542 VMGC prokaryotic genomes.

## When to use

Use when a set of MAG FASTA files needs completeness/contamination estimation and assignment to revised-MIMAG quality tiers (e.g. retaining genomes with >=50% completeness and <5% contamination).

## When NOT to use

- Input is raw metagenomic reads or unassembled contigs — this skill assumes binned MAG FASTA files already exist (see the single-/multi-coverage binning skill for the upstream step)
- Assessing viral genomes — the article uses CheckV for viral quality (10.3% complete, >=50% completeness filter), not CheckM2/MIMAG thresholds
- Input genomes are already annotated with a quality column (e.g. VMGC_prokaryote_MAG.info provides Source/Quality annotations), so re-running assessment is unnecessary

## Inputs

- Prokaryotic MAG FASTA files (n=19,542 from VMGC Zenodo record 10457006, VMGC_prokaryote_MAG.tar.gz)
- CheckM2 v1.0.1 'predict' output (per-genome Completeness, Contamination, completeness model)
- GUNC chimerism output flagging bins with discordant taxonomic gene assignments

## Outputs

- Per-genome quality table (checkm2_quality_report.tsv with Completeness, Contamination, quality class)
- Summary of quality class counts (medium-quality / high-quality / near-complete tiers)
- Filtered set of quality-passing genomes (e.g. 18,570 VMGC prokaryotic genomes with >=50% completeness and <5% contamination)

## How to apply

Download the MAG FASTA files (here the 19,542 VMGC prokaryotic genomes, 18,570 MAGs plus 972 isolated genomes, from Zenodo record 10457006) and run CheckM2 v1.0.1 'predict' over all files, parallelized with GNU parallel, to obtain per-genome Completeness, Contamination and the completeness model used. Pass the CheckM2 output through GUNC chimerism filtering to flag bins with discordant taxonomic gene assignments as potential chimeras. Then apply the revised MIMAG criteria: retain genomes with >=50% completeness and <5% contamination, and bin the retained genomes into near-complete / high-quality / medium-quality tiers. Emit a per-genome quality table (checkm2_quality_report.tsv) and aggregate tier counts for comparison against the reference distribution (10,127 medium-quality, 8,397 high-quality, 1,017 near-complete).

## Related tools

- **CheckM2** (Estimates per-genome Completeness and Contamination via 'predict'; note the article warns completeness of A. indivicum and A. nucleatum may be underestimated by CheckM2)
- **GNU parallel 20201122** (Parallelizes CheckM2 prediction and GUNC filtering across the 19,542 genome FASTA files)
- **CheckM** (Alternative completeness/contamination estimator whose results were cross-checked against CheckM2 ('Similar assessment results were also revealed by other algorithms such as CheckM and BUSCO'))
- **BUSCO** (Alternative assessment based on 124 universal single-copy orthologs (USCOs) for prokaryotes, used to corroborate CheckM2 results)

## Examples

```
find VMGC_prokaryote_MAG/*.fasta | parallel -j 20 checkm2 predict --input {} --output checkm2_out/{/.} --threads 10
```

## Evaluation signals

- Total genomes entering the pipeline equals 19,542 (18,570 MAGs + 972 isolated genomes) in the per-genome table
- Every retained genome satisfies >=50% completeness and <5% contamination; genomes failing are excluded from tier assignment
- Aggregate class counts reproduce the reported Fig. 1b distribution: 10,127 medium-quality, 8,397 high-quality, 1,017 near-complete
- checkm2_quality_report.tsv contains one row per genome with Completeness, Contamination, quality class, and CheckM2 completeness model columns
- Cross-validation with CheckM or BUSCO (124 USCOs for prokaryotes) yields similar assessment results

## Limitations

- CheckM2 may underestimate completeness for some taxa — the article notes 'the completeness of A. indicium and A. nucleatus might be underestimated in the current CheckM2 algorithm'
- GUNC flags discordant taxonomic gene assignments as potential chimeras, but kit/experimental contamination in the source public datasets remains a stated concern
- Tier boundaries are 'revised from the MIMAG standard', so class counts may differ from outputs classified strictly under the original MIMAG criteria

## Evidence

- [abstract] 19,542 prokaryotic genomes classified as 10,127 medium-quality, 8,397 high-quality and 1,017 near-complete: "10,127, 8,397 and 1,017 were classified as medium-quality, high-quality and near-complete genomes, respectively"
- [abstract] Prokaryotic genome quality filtering thresholds: "yielded 18,570 prokaryotic genomes with ≥50% completeness and <5% contamination"
- [abstract] MIMAG revision basis: "following the criteria revised from the MIMAG (Minimum Information about a Metagenome-Assembled Genome) standard"
- [abstract] CheckM2 completeness caveat: "the completeness of A. indicium and A. nucleatus might be underestimated in the current CheckM2 algorithm"
- [abstract] Cross-check with alternative estimators: "Similar assessment results were also revealed by other algorithms such as CheckM and BUSCO"
- [abstract] CheckM2 corroboration via 124 USCOs: "BUSCO (based on 124 universal single-copy orthologs (USCOs) for prokaryotes32)"
- [readme] VMGC data repository containing the 19,542 genome FASTA set: "The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006"
- [readme] GNU parallel used for parallelized per-file operations: "find contigs/*.fasta | parallel -k -j 10 mash dist mash.sketch.msh {} \| sort -nk3 \| head -n 20"
