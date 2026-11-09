---
name: gsc-mag-quality-criteria-application
description: Use when you have a CheckM2 quality_report.tsv for metagenomic bins and
  need to assign each bin a GSC-based quality label (near complete/high/medium/low)
  and summarize counts per category.
when_to_use_negative:
- The input is raw metagenomic reads or an unassembled FASTA with no bins — binning
  (e.g. MetaBAT2) and CheckM2 assessment must happen first
- You need MAG quality estimates themselves rather than categorization — run CheckM2
  to produce completeness/contamination before applying these thresholds
- The input already consists of a completed quality-category table; re-running the
  categorization would only duplicate existing labels
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: CheckM2
  role: Produces the per-bin completeness and contamination estimates consumed by
    the categorization thresholds
  repo: https://github.com/chklovski/CheckM2
- name: Python
  role: Implementation language for the categorization script (pandas parsing, matplotlib
    plotting)
- name: pandas
  role: Loads and parses the CheckM2 quality_report.tsv table
- name: matplotlib
  role: Renders the 2D completeness/contamination scatter plot (checkm-plot.pdf)
- name: Mapler
  role: Pipeline whose checkm evaluation step generates checkm_report.txt and checkm-plot.pdf
    from bin quality categories
  repo: https://github.com/Nimauric/Mapler
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:7cb5928d7451456fd3bd58ad6b155d2cfe536446b41c885454f9f58f903b6149
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chklovski/CheckM2
  tools:
  - CheckM2
  - Mapler
  - Python
  - matplotlib
  - pandas
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# gsc-mag-quality-criteria-application

## Summary

Applies GSC-style MAG quality thresholds to CheckM2 completeness/contamination estimates to categorize metagenomic assembly bins into near complete, high quality, medium quality, and low quality classes, as implemented in the Mapler pipeline's checkm report step.

## When to use

Use when you have a CheckM2 quality_report.tsv for metagenomic bins and need to assign each bin a GSC-based quality label (near complete/high/medium/low) and summarize counts per category.

## When NOT to use

- The input is raw metagenomic reads or an unassembled FASTA with no bins — binning (e.g. MetaBAT2) and CheckM2 assessment must happen first
- You need MAG quality estimates themselves rather than categorization — run CheckM2 to produce completeness/contamination before applying these thresholds
- The input already consists of a completed quality-category table; re-running the categorization would only duplicate existing labels

## Inputs

- CheckM2 quality report table (quality_report.tsv) with per-bin completeness and contamination columns
- Optional contig-count information per bin to enforce the single-contig criterion for 'near complete'

## Outputs

- Text quality report (checkm_report.txt): counts of near complete, high quality, medium quality and low quality bins followed by per-bin details (completeness, contamination, category)
- 2D scatter plot of bin completeness vs contamination (checkm-plot.pdf) rendered with matplotlib
- Per-bin quality category assignment (exactly one of four labels per bin)

## How to apply

Load the CheckM2 output table (quality_report.tsv) with pandas and parse each bin's completeness and contamination estimates. Categorize every bin by marker-gene-based GSC MAG criteria: near complete (single contig, completeness >=99%, contamination <=1%), high quality (completeness >=90%, contamination <=5%), medium quality (completeness >=50%, contamination <=10%), and low quality for bins meeting none of these. Enforce a single-pass, first-match decision order so each bin receives exactly one label. Aggregate counts per category and compile per-bin details, then write them as a text report (checkm_report.txt, a count of near complete, high quality, medium quality and low quality bins followed by details on each bin) and optionally render the completeness/contamination 2D scatter (checkm-plot.pdf) with matplotlib. This mirrors Mapler's approach of classifying assembly bins in classical quality categories according to their marker gene content, where completeness and contamination come from CheckM2's machine-learning predictions rather than lineage-specific marker sets.

## Related tools

- **CheckM2** (Produces the per-bin completeness and contamination estimates consumed by the categorization thresholds) — https://github.com/chklovski/CheckM2
- **Python** (Implementation language for the categorization script (pandas parsing, matplotlib plotting))
- **pandas** (Loads and parses the CheckM2 quality_report.tsv table)
- **matplotlib** (Renders the 2D completeness/contamination scatter plot (checkm-plot.pdf))
- **Mapler** (Pipeline whose checkm evaluation step generates checkm_report.txt and checkm-plot.pdf from bin quality categories) — https://github.com/Nimauric/Mapler

## Examples

```
import pandas as pd; df = pd.read_csv('quality_report.tsv', sep='\t'); df['quality'] = df.apply(categorize, axis=1)
```

## Evaluation signals

- Every bin present in the input quality_report.tsv receives exactly one of the four quality labels (near complete, high quality, medium quality, low quality)
- The per-category counts sum to the total bin count in the quality_report.tsv
- checkm_report.txt contains both the category counts and per-bin details (completeness, contamination, category)
- Threshold checks: 'near complete' bins satisfy completeness >=99% and contamination <=1% (and single contig); high quality bins satisfy >=90% comp / <=5% cont; medium quality satisfy >=50% comp / <=10% cont; all remaining bins fall into low quality
- checkm-plot.pdf, if rendered, shows a 2D scatter with all bins positioned at their (contamination, completeness) coordinates from CheckM2

## Limitations

- CheckM2 completeness and contamination are machine-learning predictions (universally trained models applied regardless of taxonomic lineage), so threshold assignments inherit their estimation error
- The 'near complete' criterion requires the bin to be a single contig, which most multi-contig bins cannot satisfy regardless of their completeness estimate
- This step is a classification/reporting step on existing CheckM2 output; it cannot recover or re-estimate bin quality itself

## Evidence

- [methods] Mapler assesses bins with CheckM2 and categorizes them by completeness and contamination: near complete (single contig, ≥99% comp., ≤1% cont.), high quality (≥90% comp., ≤5% cont.), medium quality (≥50% comp., ≤10% cont.), and low quality for remaining bins.: [evidence span withheld — anchor status: not_found]
- [readme] In addition to classifying assembly bins in classical quality categories according to their marker gene content and taxonomic assignment, Mapler analyzes the alignment of reads on contigs.: "In addition to classifying assembly bins in classical quality categories according to their marker gene content and taxonomic assignment"
- [readme] checkm-plot.pdf # A 2D plot of bin completeness and contamination / checkm_report.txt # A count of near complete, high quality, medium quality and low quality bins, followed by details on each bins: "checkm-plot.pdf # A 2D plot of bin completeness and contamination"
- [readme] Unlike CheckM1, CheckM2 has universally trained machine learning models it applies regardless of taxonomic lineage to predict the completeness and contamination of genomic bins.: [evidence span withheld — anchor status: external_doc]
