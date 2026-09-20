---
name: metagenome-functional-profiling
description: Use when the input is an ASV/OTU abundance table plus study marker-gene
  sequences (e.g., 16S rDNA FASTA) and the goal is predicting functional content rather
  than sequencing a shotgun metagenome directly.
when_to_use_negative:
- The input is already a functional profile (e.g., an EC or pathway table) rather
  than marker-gene abundances — no prediction is needed
- The community was profiled with direct shotgun metagenome sequencing, so functions
  can be observed rather than predicted from marker genes
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: PICRUSt2
  role: Main pipeline that predicts gene-family and MetaCyc pathway abundances from
    the ASV abundance table and study sequences
  repo: github.com/picrust/picrust2
- name: EPA-ng
  role: Read/sequence placement engine underlying PICRUSt2's placement of study sequences
    into reference phylogenies
  repo: github.com/picrust/picrust2
- name: MinPath
  role: Pathway inference step bundled with PICRUSt2 (GPL-licensed source included
    in the repo) for MetaCyc pathway reconstruction
  repo: github.com/picrust/picrust2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-020-0548-6
    title: PICRUSt2 for prediction of metagenome functions
  source_content_hash: sha256:b296e36721aa59b9ce65720049d34486763c2a3568ace22504b0ef010f4021ae
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - EPA-ng
  - MinPath
  - PICRUSt2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metagenome-functional-profiling

## Summary

Predict metagenome functional profiles from marker-gene (ASV) abundance data using PICRUSt2, producing inferred gene-family (EC) and MetaCyc pathway abundance tables from 16S-style sequence input.

## When to use

Use when the input is an ASV/OTU abundance table plus study marker-gene sequences (e.g., 16S rDNA FASTA) and the goal is predicting functional content rather than sequencing a shotgun metagenome directly.

## When NOT to use

- The input is already a functional profile (e.g., an EC or pathway table) rather than marker-gene abundances — no prediction is needed
- The community was profiled with direct shotgun metagenome sequencing, so functions can be observed rather than predicted from marker genes

## Inputs

- ASV/OTU abundance table (e.g., BIOM or TSV) with abundance columns per sample
- Study marker-gene sequences in FASTA format (the sequences corresponding to the ASV table)

## Outputs

- Predicted gene-family abundance table (EC numbers) per sample
- Inferred MetaCyc pathway abundance table per sample
- Pipeline output directory containing intermediate and final PICRUSt2 tables

## How to apply

Install PICRUSt2 following the wiki's Installation instructions (https://github.com/picrust/picrust2/wiki/Installation), then run the end-to-end wrapper picrust2_pipeline.py, passing the ASV abundance table via -i and the study sequences FASTA via -s, writing all outputs to a single directory with -o. The pipeline internally places the study sequences (via EPA-ng placement) and infers gene-family abundances (e.g., EC numbers) and MetaCyc pathway abundances; MinPath source is bundled in the GPL-3.0-licensed repository. Collect the predicted EC gene-family abundance table and the inferred MetaCyc pathway abundance table from the output directory, validating that both tables exist and contain an abundance column for every input sample. Consult the PICRUSt2 wiki and FAQ (via the PICRUSt users Google group) before troubleshooting, as the README directs questions there rather than to GitHub issues.

## Related tools

- **PICRUSt2** (Main pipeline that predicts gene-family and MetaCyc pathway abundances from the ASV abundance table and study sequences) — github.com/picrust/picrust2
- **EPA-ng** (Read/sequence placement engine underlying PICRUSt2's placement of study sequences into reference phylogenies) — github.com/picrust/picrust2
- **MinPath** (Pathway inference step bundled with PICRUSt2 (GPL-licensed source included in the repo) for MetaCyc pathway reconstruction) — github.com/picrust/picrust2

## Examples

```
picrust2_pipeline.py -i abundance_table.biom -s study_seqs.fna -o picrust2_out
```

## Evaluation signals

- The EC gene-family abundance output table and the MetaCyc pathway abundance output table both exist in the pipeline output directory
- Each output table contains an abundance column for every sample present in the input ASV abundance table
- Predicted abundance values are non-negative and populated (not empty) for the tutorial test samples
- Pathway identifiers in the inferred pathway table correspond to MetaCyc pathways
- The full run completes end-to-end via picrust2_pipeline.py on the tutorial's bundled test ASV table and study-sequence FASTA

## Limitations

- Functions are predicted from marker-gene data, not directly observed; results are inferred profiles rather than measured metagenomes
- PICRUSt2 is a prediction method (it predicts metagenome functions), so accuracy depends on the marker-gene input and reference data (e.g., Greengenes, MetaCyc); it is not a shotgun functional profiler
- Per the README, questions should go to the PICRUSt users Google group after reading the wiki FAQ; GitHub issues are reserved for actual run problems

## Evidence
