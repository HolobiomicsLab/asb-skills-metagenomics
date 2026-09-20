---
name: kegg-orthology-annotation-processing
description: Use when you already have KO-annotated genes for a genome or MAG in eggnog/kaas/kofamkoala
  format and need KEGG Module completeness summaries, or need the KEGG-Mapper INCOMPLETE
  convention (Modules with <3 blocks) for cross-tool comparison.
when_to_use_negative:
- When the goal is de novo gene calling or functional annotation from raw sequences
  — gene calling with Prodigal v2.6.3 and annotation with eggNOG-mapper v2 happen
  upstream of this skill.
- When you need to recover missing KOs via HMM profile searches or gapfill a GSMM
  — those require the --hmm_mode or --gsmm_mode stages rather than the completeness-evaluation-only
  path.
- When the input has no KEGG orthology annotations at all (KEMET requires a KO annotation
  file or its own HMM pipeline to assign KOs).
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0091
tools:
- name: KEMET
  role: command-line tool that parses the KO annotation file and generates the KEGG
    Modules completeness recap tables
  repo: https://github.com/Matteopaluh/KEMET
- name: KEGG Mapper
  role: external tool whose INCOMPLETE convention (Modules with less than 3 blocks)
    is reproduced via --as_kegg for comparison
- name: eggNOG-Mapper
  role: upstream annotator producing the .emapper.annotations KO file consumed with
    -a eggnog
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:c9c240551a67ae8a572741b8e6fb26c7cb0e3c4b9a81be14ebcfa51b22e99312
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
  - Python
  - eggNOG-Mapper
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# kegg-orthology-annotation-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ingest pre-computed KEGG orthology (KO) annotations — from eggNOG-Mapper (.emapper.annotations), KAAS, or KofamKOALA (.ko) formats — and process them with KEMET to produce organized KEGG Module completeness recap tables (modules_completeness_recap.tsv) summarizing metabolic potential.

## When to use

Use when you already have KO-annotated genes for a genome or MAG in eggnog/kaas/kofamkoala format and need KEGG Module completeness summaries, or need the KEGG-Mapper INCOMPLETE convention (Modules with <3 blocks) for cross-tool comparison.

## When NOT to use

- When the goal is de novo gene calling or functional annotation from raw sequences — gene calling with Prodigal v2.6.3 and annotation with eggNOG-mapper v2 happen upstream of this skill.
- When you need to recover missing KOs via HMM profile searches or gapfill a GSMM — those require the --hmm_mode or --gsmm_mode stages rather than the completeness-evaluation-only path.
- When the input has no KEGG orthology annotations at all (KEMET requires a KO annotation file or its own HMM pipeline to assign KOs).

## Inputs

- FASTA file of the MAG/genome of interest (e.g. genomes/bin1.fasta)
- KEGG KO annotation file in eggnog (.emapper.annotations), KAAS, or KofamKOALA (.ko) format
- Annotation-only KEGG file (used with --no_genome when no FASTA is supplied)

## Outputs

- modules_completeness_recap.tsv — KEGG Modules completeness recap table of metabolic potential
- Organized recap tables of metabolic potential (Module block completeness, with KEGG-Mapper-convention INCOMPLETE labels if --as_kegg is set)

## How to apply

Place the input FASTA and its KEGG annotation file (preserving the default extension, e.g. .emapper.annotations or .ko) in the appropriate working-directory subfolders after setting up the KEMET Conda environment per the wiki pages. Run kemet.py with -a set to the annotation format (eggnog / kaas / kofamkoala) and use --skip_hmm to stop after the KEGG MODULES Completeness Evaluation, which yields the organized recap tables of metabolic potential; alternatively use --no_genome with an annotation-only file for the same stopping point. If a KEGG-Mapper-convention summary is desired, add --as_kegg so that incomplete Modules with fewer than 3 blocks are marked INCOMPLETE regardless of the number of incomplete blocks — a more conservative approach used when benchmarking against KEGG Mapper. The optional --threshold_value flag (default 0.43) applies the HMM hit quality filter when continuing into HMM-search mode. Outputs are tabulated in modules_completeness_recap.tsv, which serves as the comparison substrate against KEGG Mapper and METABOLIC v4.0 block partitioning.

## Related tools

- **KEMET** (command-line tool that parses the KO annotation file and generates the KEGG Modules completeness recap tables) — https://github.com/Matteopaluh/KEMET
- **KEGG Mapper** (external tool whose INCOMPLETE convention (Modules with less than 3 blocks) is reproduced via --as_kegg for comparison)
- **eggNOG-Mapper** (upstream annotator producing the .emapper.annotations KO file consumed with -a eggnog)

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --skip_hmm --log
```

## Evaluation signals

- The recap table (modules_completeness_recap.tsv) is produced and lists KEGG Modules with completeness/block status for every input genome or MAG.
- With --skip_hmm, execution terminates right after the KEGG MODULES Completeness Evaluation and no HMM-search outputs are generated.
- With --as_kegg, any Module with fewer than 3 blocks is labeled INCOMPLETE regardless of how many blocks are incomplete, matching the KEGG-Mapper convention.
- The annotation file's default extension is preserved (.emapper.annotations, .ko), and KEMET correctly resolves the -a format among eggnog/kaas/kofamkoala.
- In benchmarking, KEMET's block partitioning more closely resembles the original KEGG Module structure than METABOLIC's (e.g. the M00308 terminal glyceraldehyde-3-phosphate step split), while agreeing with KEGG Mapper and METABOLIC on block interpretation overall.

## Limitations

- The --as_kegg KEGG-Mapper convention is a more conservative approach regarding annotations and can mark Modules INCOMPLETE that KEMET's default summary would report with partial blocks.
- The annotation formats supported are limited to eggnog / kaas / kofamkoala, and the default file extension must be maintained.
- With --no_genome (annotation-only input), execution stops after the KEGG MODULES Completeness Evaluation — no HMM searches or GSMM gapfilling are possible.
- KEMET's KEGG GENES database download/update depends on the KEGG API, so annotation processing depends on KEGG availability.

## Evidence
