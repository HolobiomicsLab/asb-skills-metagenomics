---
name: kegg-module-block-partitioning-comparison
description: Use when evaluating microbial genome or MAG functional potential via
  KEGG Modules and needing to compare block-level Module completeness across KEMET,
  KEGG Mapper, and METABOLIC v4.0.
when_to_use_negative:
- Input is raw sequence without prior KO annotation and only Module completeness is
  needed — gene calling with Prodigal v2.6.3 and annotation with eggNOG-mapper v2
  must be done first
- The goal is HMM-based recovery of missing KOs or GSMM gapfilling rather than Module
  completeness comparison — use KEMET's --hmm_mode or --gsmm_mode workflows instead
  of --skip_hmm
- A single-tool completeness report suffices with no cross-tool benchmarking required
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0605
tools:
- name: KEMET
  role: Command-line Python tool that evaluates KEGG Module completeness via block
    partitioning and generates recap tables with --skip_hmm and --as_kegg
  repo: https://github.com/Matteopaluh/KEMET
- name: KEGG Mapper
  role: Reference tool whose Module partitioning convention (Modules with <3 blocks
    marked INCOMPLETE) is compared against KEMET's block structure
- name: METABOLIC
  role: Third-party Module evaluation tool (v4.0) used to obtain alternative block
    partitions for benchmarking against KEMET
- name: eggNOG-Mapper
  role: Produces the .emapper.annotations KO annotations consumed by KEMET via the
    -a eggnog option
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:b698e31112de1912d7fa10791fcaf149ad2ebd3e4e334e0f647e473261eb155f
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
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# kegg-module-block-partitioning-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark how KEMET partitions KEGG Modules into blocks and computes Module completeness against KEGG Mapper and METABOLIC v4.0 across genomes, verifying that KEMET captures more Modules and its block structure more closely resembles the original KEGG Module structure.

## When to use

Use when evaluating microbial genome or MAG functional potential via KEGG Modules and needing to compare block-level Module completeness across KEMET, KEGG Mapper, and METABOLIC v4.0.

## When NOT to use

- Input is raw sequence without prior KO annotation and only Module completeness is needed — gene calling with Prodigal v2.6.3 and annotation with eggNOG-mapper v2 must be done first
- The goal is HMM-based recovery of missing KOs or GSMM gapfilling rather than Module completeness comparison — use KEMET's --hmm_mode or --gsmm_mode workflows instead of --skip_hmm
- A single-tool completeness report suffices with no cross-tool benchmarking required

## Inputs

- Microbial genome or MAG FASTA file (e.g. genomes/bin1.fasta)
- KEGG KO annotation file in eggNOG .emapper.annotations format (or kaas / kofamkoala formats)
- KEGG GENES database downloaded via the KEGG API
- Module partition outputs from KEGG Mapper and METABOLIC v4.0 for the same genomes

## Outputs

- KEGG MODULES Completeness recap tables (modules_completeness_recap.tsv) summarizing metabolic potential per genome
- KEGG-Mapper-convention recap of incomplete Modules when run with --as_kegg
- Cross-tool comparison of Module counts and block partitioning across KEMET, KEGG Mapper, and METABOLIC v4.0

## How to apply

Set up the KEMET Conda environment per the GitHub wiki installation page, placing input FASTA files and KEGG annotation files (eggNOG .emapper.annotations format) in the appropriate working-directory subfolders. Run ./kemet.py [FASTA_file] -a eggnog with the --skip_hmm flag so execution stops after the KEGG MODULES Completeness Evaluation, producing organized recap tables (modules_completeness_recap.tsv) of metabolic potential. Run again with the --as_kegg option to summarize incomplete Modules following the KEGG-Mapper convention, under which Modules with fewer than 3 blocks are marked INCOMPLETE regardless of the number of incomplete blocks. Evaluate the same genomes with KEGG Mapper and METABOLIC v4.0 to obtain their Module partitions, then compare the number of KEGG Modules captured and the block partitioning (e.g., the splitting of the alternative reaction path such as the M00308 terminal glyceraldehyde-3-phosphate step) across the three tools. Validation: accept the output if recap tables confirm KEMET captures more Modules in the evaluation and has a block structure more closely resembling KEGG's than METABOLIC's, matching the comparison in KEMET's tests/README.md.

## Related tools

- **KEMET** (Command-line Python tool that evaluates KEGG Module completeness via block partitioning and generates recap tables with --skip_hmm and --as_kegg) — https://github.com/Matteopaluh/KEMET
- **KEGG Mapper** (Reference tool whose Module partitioning convention (Modules with <3 blocks marked INCOMPLETE) is compared against KEMET's block structure)
- **METABOLIC** (Third-party Module evaluation tool (v4.0) used to obtain alternative block partitions for benchmarking against KEMET)
- **eggNOG-Mapper** (Produces the .emapper.annotations KO annotations consumed by KEMET via the -a eggnog option)

## Examples

```
./kemet.py [FASTA_file] -a eggnog --skip_hmm --as_kegg
```

## Evaluation signals

- modules_completeness_recap.tsv is generated and confirms KEMET captures more KEGG Modules in the evaluation than METABOLIC
- KEMET's block structure more closely resembles the original KEGG Module structure than METABOLIC's, with the three tools interpreting Module block structure in a largely consistent way
- With --as_kegg, Modules with fewer than 3 blocks are marked INCOMPLETE regardless of the number of incomplete blocks, matching the KEGG-Mapper convention
- Alternative reaction paths are partitioned comparably across tools; e.g. the M00308 terminal glyceraldehyde-3-phosphate step splitting is reproduced and tabulated
- The cross-tool comparison matches the results reported in the KEMET repository's tests/README.md

## Limitations

- The --as_kegg option applies a more conservative approach regarding annotations by following the KEGG-Mapper convention for incomplete Modules
- The KEGG GENES database used by KEMET may need to be updated through the KEGG API; outdated databases affect completeness evaluation
- Validation scope is limited to the benchmark genomes compared against KEGG Mapper and METABOLIC v4.0 as reported in the KEMET repository's tests/README.md

## Evidence
