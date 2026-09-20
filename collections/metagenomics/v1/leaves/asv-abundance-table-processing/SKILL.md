---
name: asv-abundance-table-processing
description: Use when the input is a marker-gene (16S rDNA) ASV abundance table with
  its corresponding sequence FASTA and the goal is to predict metagenome functional
  profiles from that marker-gene input.
when_to_use_negative:
- The input is already shotgun metagenomic data with measured functional profiles,
  so functional prediction from marker-gene sequences is unnecessary.
- You only need taxonomic classification of amplicon reads rather than functional
  prediction from an abundance table.
- The ASV abundance table lacks a corresponding sequence FASTA, since picrust2_pipeline.py
  requires both abundance (-i) and study-sequence (-s) inputs.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: PICRUSt2
  role: Core pipeline invoked via picrust2_pipeline.py to predict metagenome functional
    profiles from the ASV abundance table and study sequences
  repo: https://github.com/picrust/picrust2
- name: EPA-ng
  role: Massively parallel evolutionary placement of the study sequences used within
    the PICRUSt2 pipeline
- name: MinPath
  role: Pathway inference step; its source code is distributed within the PICRUSt2
    repository under the GNU General Public License
  repo: https://github.com/picrust/picrust2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-020-0548-6
    title: PICRUSt2 for prediction of metagenome functions
  source_content_hash: sha256:78eab07c46f6466615a2ce7c084a5d6a977d47f25f1326eacac783a9a82b4d86
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
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# asv-abundance-table-processing

## Summary

Run the PICRUSt2 end-to-end prediction pipeline on an ASV abundance table plus a study-sequence FASTA to produce predicted gene-family (EC) and MetaCyc pathway abundance tables for each sample, as documented in the PICRUSt2 wiki tutorials.

## When to use

Use when the input is a marker-gene (16S rDNA) ASV abundance table with its corresponding sequence FASTA and the goal is to predict metagenome functional profiles from that marker-gene input.

## When NOT to use

- The input is already shotgun metagenomic data with measured functional profiles, so functional prediction from marker-gene sequences is unnecessary.
- You only need taxonomic classification of amplicon reads rather than functional prediction from an abundance table.
- The ASV abundance table lacks a corresponding sequence FASTA, since picrust2_pipeline.py requires both abundance (-i) and study-sequence (-s) inputs.

## Inputs

- ASV abundance table (amplicon feature table, e.g. from DADA2-corrected 16S rDNA sequences)
- Study-sequence FASTA file corresponding to the ASVs

## Outputs

- Predicted gene-family (EC) abundance table
- Inferred MetaCyc pathway abundance table
- Pipeline output directory containing all intermediate and final results

## How to apply

First install PICRUSt2 following the wiki installation instructions, then take the tutorial's bundled test ASV abundance table and matching study-sequence FASTA as inputs. Run picrust2_pipeline.py, passing the abundance table with -i and the study sequences with -s, writing all results to a single output directory. The pipeline internally relies on EPA-ng for evolutionary placement of study sequences and MinPath (distributed under the GNU GPL) for pathway inference, producing a predicted gene-family (EC) abundance table and an inferred MetaCyc pathway abundance table. Validate that both output tables exist in the output directory and contain abundance columns for each test sample before proceeding to downstream analysis.

## Related tools

- **PICRUSt2** (Core pipeline invoked via picrust2_pipeline.py to predict metagenome functional profiles from the ASV abundance table and study sequences) — https://github.com/picrust/picrust2
- **EPA-ng** (Massively parallel evolutionary placement of the study sequences used within the PICRUSt2 pipeline)
- **MinPath** (Pathway inference step; its source code is distributed within the PICRUSt2 repository under the GNU General Public License) — https://github.com/picrust/picrust2

## Examples

```
picrust2_pipeline.py -i tutorial_asv_table.biom -s tutorial_study_seqs.fna -o picrust2_out
```

## Evaluation signals

- The EC gene-family abundance output table exists in the pipeline output directory.
- The MetaCyc pathway abundance output table exists in the pipeline output directory.
- Both output tables contain abundance columns for each test sample present in the input abundance table.
- The run completed without errors after following the wiki installation instructions, indicating a valid installation and pipeline execution.

## Limitations

- Questions about PICRUSt2 should be asked on the PICRUSt users Google group rather than as GitHub issues, and the FAQ should be read first; only genuine runtime issues belong in GitHub issues.
- The source documentation is spread across the PICRUSt2 wiki, which must be read before use; the repository README itself contains no abstract body or changelog.
- The MetaCyc pathway predictions depend on MinPath inference and EPA-ng sequence placement, so errors in the input FASTA or abundance table propagate into the pathway output.

## Evidence

- [readme] Please see the PICRUSt2 wiki for the documentation and tutorials.: "Please see the [PICRUSt2 wiki](https://github.com/picrust/picrust2/wiki) for the documentation and tutorials."
- [readme] There is a lot of information within this Wiki about how to install and run PICRUSt2 as well as how it works.: "There is a lot of information within this Wiki about how to install and run PICRUSt2 as well as how it works"
- [methods] Installation instructions are found on the wiki installation page.: "Installation instructions are found [here](https://github.com/picrust/picrust2/wiki/Installation)."
- [intro] The repository contains the source code for MinPath, distributed under the GNU GPL.: "this repository contains the source code for MinPath, which is distributed under the GNU General Public License"
- [supplementary] Piphillin predicts metagenomic composition and dynamics from DADA2-corrected 16S rDNA sequences.: "Piphillin predicts metagenomic composition and dynamics from DADA2-corrected 16S rDNA sequences"
