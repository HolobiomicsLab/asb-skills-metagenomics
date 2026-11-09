---
name: picrust2-pipeline-execution
description: Use when the input is an ASV/OTU abundance table plus marker-gene study
  sequences (FASTA) and the goal is to infer functional metagenome content (EC gene
  families and MetaCyc pathways) without shotgun sequencing.
when_to_use_negative:
- Input is already shotgun metagenomic data with directly measured functional profiles
  — functional prediction from marker genes is unnecessary
- Only taxonomic relative abundances without the underlying study sequences are available,
  since picrust2_pipeline.py requires the sequence FASTA (-s) for placement
edam_operation: http://edamontology.org/operation_0327
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0621
tools:
- name: PICRUSt2
  role: Main pipeline (picrust2_pipeline.py) that predicts metagenome functions from
    the abundance table and study sequences
  repo: https://github.com/picrust/picrust2
- name: EPA-ng
  role: Evolutionary placement of study marker-gene sequences used within the prediction
    pipeline
- name: MinPath
  role: Pathway inference/reconciliation step distributed with the PICRUSt2 source
    code
  repo: https://github.com/picrust/picrust2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-020-0548-6
    title: PICRUSt2 for prediction of metagenome functions
  source_content_hash: sha256:4716c1d9c4412fe5b4bc31d798365da4dca1bbc6b2aff0565cc83484a68f84ea
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/picrust/picrust2
  tools:
  - EPA-ng
  - MinPath
  - PICRUSt2
  edam_operation: http://edamontology.org/operation_0327
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# picrust2-pipeline-execution

## Summary

Executes the PICRUSt2 end-to-end pipeline to predict metagenome functional profiles from marker-gene (16S ASV) abundance tables and study sequences, producing predicted gene-family (EC) and MetaCyc pathway abundance tables.

## When to use

Use when the input is an ASV/OTU abundance table plus marker-gene study sequences (FASTA) and the goal is to infer functional metagenome content (EC gene families and MetaCyc pathways) without shotgun sequencing.

## When NOT to use

- Input is already shotgun metagenomic data with directly measured functional profiles — functional prediction from marker genes is unnecessary
- Only taxonomic relative abundances without the underlying study sequences are available, since picrust2_pipeline.py requires the sequence FASTA (-s) for placement

## Inputs

- ASV abundance table (e.g. the tutorial's bundled test ASV abundance table)
- Marker-gene study sequence FASTA corresponding to the abundance table features

## Outputs

- Predicted gene-family (EC) abundance table with an abundance column per sample
- Inferred MetaCyc pathway abundance table with an abundance column per sample
- Pipeline output directory containing all intermediate and final prediction results

## How to apply

First install PICRUSt2 following the wiki installation instructions (github.com/picrust/picrust2/wiki/Installation). Load the ASV abundance table and the corresponding study-sequence FASTA; sequences are placed into a reference tree using EPA-ng and function predictions are reconciled with MinPath, whose source is bundled and GPL-licensed. Run picrust2_pipeline.py with the abundance table as the -i argument and the study sequences as the -s argument, directing all results to a single output directory with -o. Collect the predicted gene-family (EC) abundance table and the inferred MetaCyc pathway abundance table from the output directory. Validate that both tables exist and contain an abundance column for every sample in the input table.

## Related tools

- **PICRUSt2** (Main pipeline (picrust2_pipeline.py) that predicts metagenome functions from the abundance table and study sequences) — https://github.com/picrust/picrust2
- **EPA-ng** (Evolutionary placement of study marker-gene sequences used within the prediction pipeline)
- **MinPath** (Pathway inference/reconciliation step distributed with the PICRUSt2 source code) — https://github.com/picrust/picrust2

## Examples

```
picrust2_pipeline.py -i test_asv_table.biom -s test_study_seqs.fna -o picrust2_out
```

## Evaluation signals

- EC gene-family abundance table exists in the pipeline output directory
- MetaCyc pathway abundance table exists in the pipeline output directory
- Both output tables contain abundance columns for each test sample present in the input ASV table
- The run completes end-to-end using the wiki-documented workflow (install, run picrust2_pipeline.py, collect outputs)

## Limitations

- Troubleshooting and usage questions are routed through the PICRUSt users Google group and the wiki FAQ rather than GitHub issues; the README asks users to read the wiki before asking questions
- No abstract body or discussion content was present in the source document, so precision/accuracy guidance beyond the documented workflow is not captured here

## Evidence

- [other] Run picrust2_pipeline.py with the abundance table (-i) and study sequences (-s) as inputs, writing results to a single output directory.: [evidence span withheld — anchor status: not_found]
- [other] confirm that the EC and pathway abundance output tables exist and contain abundance columns for each test sample: [evidence span withheld — anchor status: not_found]
