---
name: metabolic-potential-recap-table-generation
description: Use when you have a genome or MAG FASTA plus KEGG KO annotations (eggNOG
  .emapper.annotations, KAAS, or KofamKoala format) and need tabular KEGG Module completeness/metabolic
  potential summaries without HMM-based annotation expansion.
when_to_use_negative:
- Input lacks pre-computed KEGG KO annotations and HMM-based recovery of missing orthologs
  is the goal — use KEMET's --hmm_mode (e.g. onebm) instead of --skip_hmm.
- The goal is GSMM reconstruction or gapfilling rather than Module completeness tabulation
  — use --gsmm_mode (denovo or existing) workflows instead.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_3174
tools:
- name: KEMET
  role: Command-line Python tool that performs the KEGG MODULES Completeness Evaluation
    and writes the recap tables when run with --skip_hmm
  repo: https://github.com/Matteopaluh/KEMET
- name: Python
  role: Language/runtime of the kemet.py command-line toolbox
  repo: https://github.com/Matteopaluh/KEMET
- name: eggNOG-Mapper
  role: Upstream annotator producing the .emapper.annotations KO input consumed via
    -a eggnog
- name: KEGG Mapper
  role: Reference tool whose INCOMPLETE convention (--as_kegg) and Module partitioning
    are used for comparison/validation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:1fef3bb0bfad42897003e4c91f48908735c48590952f15f0e75e98613a052653
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Matteopaluh/KEMET
  repo_url: https://github.com/Matteopaluh/KEMET
  tools:
  - KEGG Mapper
  - KEMET
  - METABOLIC
  - Python
  - eggNOG-Mapper
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# metabolic-potential-recap-table-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates organized recap tables of KEGG Module completeness and metabolic potential for MAGs or genomes using KEMET's KEGG MODULES Completeness Evaluation, stopping before HMM analysis. This summarizes which KEGG Modules are complete/incomplete from existing KEGG KO annotations without running profile HMM searches.

## When to use

Use when you have a genome or MAG FASTA plus KEGG KO annotations (eggNOG .emapper.annotations, KAAS, or KofamKoala format) and need tabular KEGG Module completeness/metabolic potential summaries without HMM-based annotation expansion.

## When NOT to use

- Input lacks pre-computed KEGG KO annotations and HMM-based recovery of missing orthologs is the goal — use KEMET's --hmm_mode (e.g. onebm) instead of --skip_hmm.
- The goal is GSMM reconstruction or gapfilling rather than Module completeness tabulation — use --gsmm_mode (denovo or existing) workflows instead.

## Inputs

- MAG/genome FASTA file (e.g. genomes/bin1.fasta) or a KEGG annotation file
- KEGG KO annotation file in eggnog (.emapper.annotations), kaas, or kofamkoala (.ko) format

## Outputs

- Organized recap tables of metabolic potential (modules_completeness_recap.tsv) with KEGG Module completeness per genome
- KEGG Module block partitioning summary, optionally using KEGG-Mapper INCOMPLETE convention with --as_kegg

## How to apply

Set up the KEMET Conda environment per the GitHub wiki installation page and place the FASTA file and KEGG annotation file (e.g. eggNOG .emapper.annotations format) in the appropriate working-directory subfolders. Run ./kemet.py [FASTA_file] -a eggnog with the --skip_hmm flag so execution stops after the KEGG MODULES Completeness Evaluation, producing organized recap tables of metabolic potential (modules_completeness_recap.tsv). Optionally add --as_kegg to summarize incomplete Modules using the KEGG-Mapper convention, under which Modules with less than 3 blocks are marked as INCOMPLETE regardless of the number of incomplete blocks — a more conservative approach regarding annotations. The -a FORMAT parameter selects the annotation program used to assign KEGG KOs (eggnog / kaas / kofamkoala), and default file extensions must be maintained (e.g. .emapper.annotations, .ko). Validation against KEGG Mapper and METABOLIC v4.0 showed KEMET captures more Modules in the evaluation and has a block structure more closely resembling that of KEGG.

## Related tools

- **KEMET** (Command-line Python tool that performs the KEGG MODULES Completeness Evaluation and writes the recap tables when run with --skip_hmm) — https://github.com/Matteopaluh/KEMET
- **Python** (Language/runtime of the kemet.py command-line toolbox) — https://github.com/Matteopaluh/KEMET
- **eggNOG-Mapper** (Upstream annotator producing the .emapper.annotations KO input consumed via -a eggnog)
- **KEGG Mapper** (Reference tool whose INCOMPLETE convention (--as_kegg) and Module partitioning are used for comparison/validation)

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --skip_hmm --as_kegg --log
```

## Evaluation signals

- modules_completeness_recap.tsv is produced and contains organized Module completeness tables for the input genome(s).
- With --skip_hmm, execution terminates after the KEGG MODULES Completeness Evaluation (no HMM or GSMM outputs are generated).
- With --as_kegg, Modules with fewer than 3 blocks are marked INCOMPLETE regardless of the number of incomplete blocks, matching the KEGG-Mapper convention.
- Block partitioning of Modules (e.g. M00308 terminal glyceraldehyde-3-phosphate step splitting) matches the KEGG Module structure more closely than METABOLIC v4.0 when compared with KEGG Mapper and METABOLIC.
- Recap results reproduce the comparison reported in the KEMET repository's tests/README.md, where KEMET captures more Modules in the evaluation than METABOLIC.

## Limitations

- With --skip_hmm, the output reflects only existing KO annotations; missing orthologs are not recovered, so Module completeness depends on upstream annotation quality.
- --as_kegg applies a more conservative approach regarding annotations, marking Modules with less than 3 blocks as INCOMPLETE regardless of the number of incomplete blocks.
- Default file extensions must be maintained for the annotation input (e.g. .emapper.annotations, .ko).

## Evidence
