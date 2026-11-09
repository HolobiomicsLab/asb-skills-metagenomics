---
name: genome-scale-metabolic-model-reconstruction
description: Use when draft GSMMs built from metagenome-assembled genomes underpredict
  metabolic phenotypes (e.g. missing exchange reactions) and you want to recover missing
  KOs with HMM searches and evidence-based gapfilling before flux analysis.
when_to_use_negative:
- The input is already a complete-genome isolate with a high-quality reference GSMM,
  so no MAG gapfilling is needed
- The goal is only KEGG Module completeness summarization with no genome-scale model
  — use --skip_gsmm / KEGG Module evaluation alone instead
- Existing gap-filled draft models are already trusted and no genomic-evidence-based
  refinement of added reactions is required
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_3174
tools:
- name: KEMET
  role: HMM-based KO recovery and evidence-based GSMM/GEM gapfilling (--hmm_mode onebm,
    --gsmm_mode denovo or existing)
  repo: https://github.com/Matteopaluh/KEMET
- name: CarveMe
  role: Reconstructs reference GSMMs from complete genomes and MAGs with options --fbc2
    -u
- name: gapseq
  role: Alternative reconstruction tool for complete-genome and MAG draft models
- name: eggNOG-Mapper
  role: Provides the KEGG KO annotation input file consumed by KEMET via the -a eggnog
    format flag
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:2b8b468f45d0cc3666c059c535fda06336dd96129515ba23f415a7cc7e4e950b
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
  - CarveMe
  - KEMET
  - Python
  - eggNOG-Mapper
  - gapseq
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# genome-scale-metabolic-model-reconstruction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct and expand genome-scale metabolic models (GSMM/GEM) from MAGs or complete genomes by gapfilling with HMM-recovered KEGG orthologs, then quantify metabolic capability via flux variability analysis (FVA). KEMET-driven annotation expansion closes ~10% of the accuracy gap between MAG-derived and complete-genome models.

## When to use

Use when draft GSMMs built from metagenome-assembled genomes underpredict metabolic phenotypes (e.g. missing exchange reactions) and you want to recover missing KOs with HMM searches and evidence-based gapfilling before flux analysis.

## When NOT to use

- The input is already a complete-genome isolate with a high-quality reference GSMM, so no MAG gapfilling is needed
- The goal is only KEGG Module completeness summarization with no genome-scale model — use --skip_gsmm / KEGG Module evaluation alone instead
- Existing gap-filled draft models are already trusted and no genomic-evidence-based refinement of added reactions is required

## Inputs

- MAG FASTA file (e.g. genomes/bin1.fasta)
- KEGG KO annotation file in eggNOG / KAAS / KofamKOALA format with default extension (.emapper.annotations, .ko)
- Complete-genome FASTA files of the corresponding reference organisms (from NCBI or PATRIC/IGGdb)
- Existing draft GSMM/GEM (CarveMe or gapseq output) when using --gsmm_mode existing

## Outputs

- KEGG MODULES completeness recap tables of metabolic potential
- De-novo or expanded GSMM/GEM with HMM-recovered reactions mapped to BiGG/ModelSEED reactions
- FVA exchange-reaction flux ranges per model (MAG-only, KEMET-expanded, complete-genome)
- Comparison of predicted metabolic capabilities across the three model sets

## How to apply

Annotate predicted genes with eggNOG-mapper (or KAAS/KofamKOALA) so the MAG has a KEGG KO annotation file with its default extension (e.g. .emapper.annotations), then run KEMET on the MAG FASTA with -a [FORMAT] --hmm_mode onebm --gsmm_mode denovo to reconstruct a de-novo GSMM in which HMM-recovered KOs (quality threshold default 0.43) are mapped to BiGG/ModelSEED reactions. For a complete-genome reference, reconstruct the GSMM with CarveMe v1.4.1 using --fbc2 -u (or gapseq as an alternative); when a CarveMe/gapseq draft model already exists, rerun KEMET with --gsmm_mode existing to add HMM-recovered reactions to the existing model instead of rebuilding. Perform flux variability analysis on the three model sets (MAG-only, KEMET-expanded, complete-genome) at fixed maximal growth rate to obtain exchange-reaction flux ranges, and compare predicted metabolic capabilities to measure how much of the gap toward complete-genome accuracy the expansion closes.

## Related tools

- **KEMET** (HMM-based KO recovery and evidence-based GSMM/GEM gapfilling (--hmm_mode onebm, --gsmm_mode denovo or existing)) — https://github.com/Matteopaluh/KEMET
- **CarveMe** (Reconstructs reference GSMMs from complete genomes and MAGs with options --fbc2 -u)
- **gapseq** (Alternative reconstruction tool for complete-genome and MAG draft models)
- **eggNOG-Mapper** (Provides the KEGG KO annotation input file consumed by KEMET via the -a eggnog format flag)

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --hmm_mode onebm --gsmm_mode denovo --log
```

## Evaluation signals

- KEMET-expanded MAG models recover metabolic capabilities absent from MAG-only models (e.g. Selenomonas ruminantium cellobiose and related sugar exchange reactions)
- Expanded models close roughly 10% of the gap toward complete-genome model accuracy in predicting metabolite production and consumption observed in wet lab experiments
- FVA runs at fixed maximal growth rate produce non-empty, comparable exchange-reaction flux ranges for all three model sets
- KO mock removal tests recover the removed gene/annotation, e.g. 55% (20/36) true positive rate for MAGs and 67% (8/12) for complete genomes with --hmm_mode onebm
- HMM hit quality respects the default threshold 0.43 unless --threshold_value is explicitly changed

## Limitations

- MAG-based GSMMs are prone to reconstruction errors due to the gapped nature of metagenomic assemblies
- HMM recovery of truncated/partial gene sequences is imperfect: mock KO removal achieved only 55% (MAGs) and 67% (complete genomes) true positive recovery
- Network gap-fill reactions not supported by genomic evidence can yield erroneous predictions — KEMET's evidence-based gapfilling addresses but does not eliminate this concern

## Evidence
