---
name: metabolic-pathway-reconstruction
description: Use when the input is an ASV/OTU abundance table plus a study-sequence
  FASTA and the goal is a predicted metagenome functional profile (EC gene families
  and MetaCyc pathway abundances) without shotgun sequencing data.
when_to_use_negative:
- The dataset already consists of shotgun metagenome sequences — functional profiles
  can be measured directly rather than predicted from marker genes
- The input is a precomputed functional profile table rather than an ASV/OTU abundance
  table plus study sequences
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: PICRUSt2
  role: Core pipeline that predicts metagenome functional profiles from an ASV abundance
    table and study-sequence FASTA
  repo: https://github.com/picrust/picrust2
- name: EPA-ng
  role: Evolutionary placement of study sequences against reference trees within the
    pipeline
- name: MinPath
  role: Inference of minimal MetaCyc pathway sets; source code distributed in the
    PICRUSt2 repository under the GNU General Public License
  repo: https://github.com/picrust/picrust2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-020-0548-6
    title: PICRUSt2 for prediction of metagenome functions
  source_content_hash: sha256:2696b0c3b503f1a39fe3a4727f0736e12ced9096a88557f5c482d5f1162f6661
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/picrust/picrust2
  tools:
  - EPA-ng
  - MinPath
  - PICRUSt2
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# metabolic-pathway-reconstruction

## Summary

Predict metagenome functional profiles from marker-gene (ASV/OTU) sequence data using PICRUSt2, which combines sequence placement (EPA-ng), ancestral reconstruction, and MinPath to infer gene-family (EC) and MetaCyc pathway abundances.

## When to use

Use when the input is an ASV/OTU abundance table plus a study-sequence FASTA and the goal is a predicted metagenome functional profile (EC gene families and MetaCyc pathway abundances) without shotgun sequencing data.

## When NOT to use

- The dataset already consists of shotgun metagenome sequences — functional profiles can be measured directly rather than predicted from marker genes
- The input is a precomputed functional profile table rather than an ASV/OTU abundance table plus study sequences

## Inputs

- ASV abundance table (per-sample read/sequence counts)
- Study sequences FASTA (marker-gene sequences, e.g. 16S rRNA amplicons)

## Outputs

- Predicted gene-family (EC) abundance table with an abundance column per sample
- Inferred MetaCyc pathway abundance table with an abundance column per sample

## How to apply

Install PICRUSt2 following the wiki installation instructions, then load the ASV abundance table and the corresponding study-sequence FASTA. Run picrust2_pipeline.py with the abundance table via -i and the study sequences via -s, writing all results to a single output directory. The pipeline internally places study sequences against reference trees (EPA-ng), performs ancestral state reconstruction, and uses MinPath (GPL-licensed source bundled in the repository) to infer MetaCyc pathways. Collect the predicted gene-family (EC) abundance table and the inferred MetaCyc pathway abundance table from the output directory.

## Related tools

- **PICRUSt2** (Core pipeline that predicts metagenome functional profiles from an ASV abundance table and study-sequence FASTA) — https://github.com/picrust/picrust2
- **EPA-ng** (Evolutionary placement of study sequences against reference trees within the pipeline)
- **MinPath** (Inference of minimal MetaCyc pathway sets; source code distributed in the PICRUSt2 repository under the GNU General Public License) — https://github.com/picrust/picrust2

## Examples

```
picrust2_pipeline.py -i abundance_table.biom -s study_seqs.fna -o picrust2_out
```

## Evaluation signals

- The EC gene-family abundance table exists in the pipeline output directory
- The MetaCyc pathway abundance table exists in the pipeline output directory
- Both output tables contain an abundance column for every sample in the input ASV table
- Abundance values in the output tables are numeric and non-negative for all test samples
- The run reproduces the tutorial's test-data output when executed on the bundled test ASV table and FASTA

## Limitations

- PICRUSt2 documentation and tutorials are hosted on the GitHub wiki; users are directed to consult the wiki and the FAQ before troubleshooting
- Questions should be routed to the PICRUSt users Google group rather than GitHub issues; only genuine runtime problems belong on the issue tracker
- Predicted profiles are inferences from marker-gene data (via reference databases such as Greengenes, IMG, MetaCyc/KEGG lineage information), not directly measured metagenomes

## Evidence

- [readme] There is a lot of information within this Wiki about how to install and run PICRUSt2 as well as how it works: "Please see the [PICRUSt2 wiki](https://github.com/picrust/picrust2/wiki) for the documentation and tutorials. There is a lot of information within this Wiki about how to install and run PICRUSt2"
