---
name: gene-family-abundance-inference
description: Use when you have an ASV/OTU abundance table plus a FASTA of marker-gene
  study sequences and need inferred functional profiles (EC gene families, MetaCyc
  pathways) without shotgun metagenomics data.
when_to_use_negative:
- Input is already a shotgun metagenome functional profile rather than marker-gene
  sequence data — prediction is unnecessary.
- Only a species/composition table is available without the underlying study sequences,
  since the pipeline requires a FASTA of study sequences for placement.
- The goal is taxonomy-based functional profiling with habitat-specific references
  (e.g. Tax4Fun2/Piphillin-style approaches) rather than PICRUSt2's tree-placement
  workflow.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0605
tools:
- name: PICRUSt2
  role: Main pipeline that predicts metagenome functions from the ASV abundance table
    and study sequences via picrust2_pipeline.py
  repo: https://github.com/picrust/picrust2
- name: EPA-ng
  role: Performs massively parallel evolutionary placement of study genetic sequences
    onto the reference tree within the PICRUSt2 workflow
  repo: https://github.com/picrust/picrust2
- name: MinPath
  role: Infers MetaCyc pathways; source code is bundled in the picrust2 repository
    under the GNU General Public License
  repo: https://github.com/picrust/picrust2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-020-0548-6
    title: PICRUSt2 for prediction of metagenome functions
  source_content_hash: sha256:59c0abd175914f773eadd8a349d710924c573fe7d23764d29f8a77d1bbc79512
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/picrust/picrust2
  tools:
  - EPA-ng
  - MinPath
  - PICRUSt2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# gene-family-abundance-inference

## Summary

Predicts metagenome gene-family (e.g. EC) and pathway (MetaCyc) abundance profiles from 16S/ITS amplicon sequence data using PICRUSt2, which places study sequences into a reference tree and infers functional potential from ASV abundances.

## When to use

Use when you have an ASV/OTU abundance table plus a FASTA of marker-gene study sequences and need inferred functional profiles (EC gene families, MetaCyc pathways) without shotgun metagenomics data.

## When NOT to use

- Input is already a shotgun metagenome functional profile rather than marker-gene sequence data — prediction is unnecessary.
- Only a species/composition table is available without the underlying study sequences, since the pipeline requires a FASTA of study sequences for placement.
- The goal is taxonomy-based functional profiling with habitat-specific references (e.g. Tax4Fun2/Piphillin-style approaches) rather than PICRUSt2's tree-placement workflow.

## Inputs

- ASV/OTU abundance table (per-sample counts)
- FASTA file of marker-gene (e.g. 16S rRNA) study sequences

## Outputs

- Predicted gene-family (EC) abundance table with abundance columns per sample
- Inferred MetaCyc pathway abundance table with abundance columns per sample

## How to apply

Install PICRUSt2 following the documented installation instructions in the repository wiki. Load the study ASV abundance table and the corresponding study-sequence FASTA, then run picrust2_pipeline.py with the abundance table supplied via -i and the study sequences via -s, writing results to a single output directory. The pipeline internally uses EPA-ng to place sequences onto a reference phylogeny and MinPath to infer pathways; MinPath source is distributed under the GNU General Public License in the repository. Collect the predicted gene-family (EC) abundance table and the inferred MetaCyc pathway abundance table from the pipeline output directory for downstream analysis.

## Related tools

- **PICRUSt2** (Main pipeline that predicts metagenome functions from the ASV abundance table and study sequences via picrust2_pipeline.py) — https://github.com/picrust/picrust2
- **EPA-ng** (Performs massively parallel evolutionary placement of study genetic sequences onto the reference tree within the PICRUSt2 workflow) — https://github.com/picrust/picrust2
- **MinPath** (Infers MetaCyc pathways; source code is bundled in the picrust2 repository under the GNU General Public License) — https://github.com/picrust/picrust2

## Examples

```
picrust2_pipeline.py -i asv_table.biom -s study_seqs.fna -o picrust2_out
```

## Evaluation signals

- EC gene-family abundance output table exists in the pipeline output directory and contains an abundance column for every input sample.
- MetaCyc pathway abundance output table exists in the pipeline output directory and contains an abundance column for every input sample.
- Predicted pathway abundances are consistent with MinPath-inferred MetaCyc pathways derived from the placed sequences.
- Output abundance values are non-negative and correspond to the samples present in the input -i abundance table.
- Tutorial test data run reproduces EC and pathway tables end-to-end when executed with the bundled test ASV table and FASTA.

## Limitations

- Functional profiles are predictions from marker-gene data, not observations; accuracy depends on reference databases such as Greengenes, IMG, KEGG, MetaCyc and BioCyc used for inference.
- Questions about PICRUSt2 behavior should go to the PICRUSt users Google group (after reading the FAQ) rather than GitHub issues, and only genuine running problems belong in the issue tracker.
- The bundled MinPath source is distributed under the GNU General Public License, which imposes licensing obligations on redistribution.

## Evidence

- [readme] Please see the PICRUSt2 wiki for the documentation and tutorials. There is a lot of information within this Wiki about how to install and run PICRUSt2 as well as how it works.: "Please see the [PICRUSt2 wiki](https://github.com/picrust/picrust2/wiki) for the documentation and tutorials."
- [readme] Note that this repository contains the source code for MinPath, which is distributed under the GNU General Public License.: "Note that this repository contains the source code for MinPath, which is distributed under the GNU General Public License."
- [methods] Installation instructions are found [here](https://github.com/picrust/picrust2/wiki/Installation).: "Installation instructions are found [here](https://github.com/picrust/picrust2/wiki/Installation)."
- [supplementary] EPA-ng: Massively Parallel Evolutionary Placement of Genetic Sequences: "EPA-ng: Massively Parallel Evolutionary Placement of Genetic Sequences"
