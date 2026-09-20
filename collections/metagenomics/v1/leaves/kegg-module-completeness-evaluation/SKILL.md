---
name: kegg-module-completeness-evaluation
description: Use when you have a FASTA genome/MAG plus a KEGG KO annotation file (eggNOG/kaas/kofamkoala
  format) and need Module-level completeness tables or a comparison of Module block
  partitioning across KEMET, KEGG Mapper, and METABOLIC.
when_to_use_negative:
- You need HMM-based recovery of missing KOs or GSMM gapfilling — skip this completeness-only
  mode and instead run kemet.py without --skip_hmm (or with --hmm_mode onebm/module/kos).
- Input is raw sequence without KEGG KO annotations and you do not want to run an
  annotator (eggNOG-mapper/kaas/kofamkoala) first — kemet.py requires an existing
  KO annotation format.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0769
tools:
- name: KEMET
  role: 'Primary tool: runs the KEGG MODULES Completeness Evaluation and produces
    the recap tables of metabolic potential'
  repo: https://github.com/Matteopaluh/KEMET
- name: Python
  role: Implementation language of the kemet.py command-line toolbox
- name: KEGG Mapper
  role: 'Comparison tool: provides an alternative Module partitioning used for benchmarking,
    whose convention KEMET reproduces with --as_kegg'
- name: METABOLIC
  role: 'Comparison tool: METABOLIC v4.0 Module partitions are evaluated against KEMET''s
    block structure'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:68b5e1110b2192b25a510563edf89213d4e3ae3a3ba33d561e16c33d844c6b8a
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
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# kegg-module-completeness-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate KEGG Module completeness for a microbial genome or MAG by running KEMET's kemet.py, which parses KEGG orthology annotations (e.g. eggNOG .emapper.annotations) into Module block structures and produces organized recap tables of metabolic potential. Also supports benchmarking Module partitioning against KEGG Mapper and METABOLIC v4.0.

## When to use

Use when you have a FASTA genome/MAG plus a KEGG KO annotation file (eggNOG/kaas/kofamkoala format) and need Module-level completeness tables or a comparison of Module block partitioning across KEMET, KEGG Mapper, and METABOLIC.

## When NOT to use

- You need HMM-based recovery of missing KOs or GSMM gapfilling — skip this completeness-only mode and instead run kemet.py without --skip_hmm (or with --hmm_mode onebm/module/kos).
- Input is raw sequence without KEGG KO annotations and you do not want to run an annotator (eggNOG-mapper/kaas/kofamkoala) first — kemet.py requires an existing KO annotation format.

## Inputs

- MAG/genome FASTA file (e.g. genomes/bin1.fasta)
- KEGG KO annotation file in eggnog (.emapper.annotations), kaas, or kofamkoala (.ko) format

## Outputs

- modules_completeness_recap.tsv — organized recap tables of KEGG Module completeness and metabolic potential
- Module block partitioning comparable to KEGG Mapper and METABOLIC v4.0 outputs

## How to apply

Install KEMET in a Conda environment per the GitHub wiki installation page, and place input FASTA files and KEGG annotation files (eggNOG .emapper.annotations format) in the appropriate working-directory subfolders. Run ./kemet.py [FASTA_file] -a eggnog with the --skip_hmm flag so execution stops after the KEGG MODULES Completeness Evaluation, yielding organized recap tables of metabolic potential in modules_completeness_recap.tsv. Optionally add the --as_kegg option to summarize incomplete Modules following the KEGG-Mapper convention, under which Modules with fewer than 3 blocks are marked INCOMPLETE regardless of the number of incomplete blocks (a more conservative treatment of annotations). To benchmark block partitioning, additionally run the same genomes with KEGG Mapper and METABOLIC v4.0 and compare the number of Modules captured and how alternative reaction paths (e.g. the M00308 terminal glyceraldehyde-3-phosphate step) are split across tools.

## Related tools

- **KEMET** (Primary tool: runs the KEGG MODULES Completeness Evaluation and produces the recap tables of metabolic potential) — https://github.com/Matteopaluh/KEMET
- **Python** (Implementation language of the kemet.py command-line toolbox)
- **KEGG Mapper** (Comparison tool: provides an alternative Module partitioning used for benchmarking, whose convention KEMET reproduces with --as_kegg)
- **METABOLIC** (Comparison tool: METABOLIC v4.0 Module partitions are evaluated against KEMET's block structure)

## Examples

```
./kemet.py [FASTA_file] -a eggnog --skip_hmm --as_kegg
```

## Evaluation signals

- modules_completeness_recap.tsv is produced with organized Module completeness tables and no HMM/GSMM stages executed (with --skip_hmm execution stops after Completeness Evaluation).
- With --as_kegg, Modules with fewer than 3 blocks are marked INCOMPLETE regardless of the number of incomplete blocks, matching the KEGG-Mapper convention.
- KEMET captures more KEGG Modules in the evaluation than METABOLIC v4.0 and its block structure more closely resembles the original KEGG Module structure.
- The three tools (KEMET, KEGG Mapper, METABOLIC) interpret Module block structure in a largely consistent way, e.g. alternative reaction paths such as the M00308 terminal glyceraldehyde-3-phosphate step split consistently.
- Results match the comparison reported in the KEMET repository's tests/README.md (https://github.com/Matteopaluh/KEMET/blob/main/tests/README.md).

## Limitations

- The --as_kegg option applies the KEGG-Mapper convention, a more conservative approach regarding annotations, which marks Modules with less than 3 blocks as INCOMPLETE regardless of incomplete block count.
- Completeness evaluation alone does not recover missing orthologs; KEMET's HMM profile searches (which identified unannotated genes with partial sequence truncations, e.g. 55% TPR for MAGs and 67% for complete genomes in KO mock removal tests) are a separate downstream step.
- The tool is designed for UNIX-based machines (though code is Windows-compatible) and requires external KEGG annotation formats with maintained file extensions.

## Evidence
