---
name: mag-annotation-expansion-gsmm-gapfilling
description: Use when you have a MAG FASTA with an existing KEGG KO annotation (eggNOG/KAAS/KofamKOALA
  format) and need to recover missing KOs and improve MAG-derived GSMM metabolic capability
  predictions.
when_to_use_negative:
- Input is a complete isolate genome with exhaustive annotation rather than a gapped
  MAG, where annotation recovery adds little value
- You only need KEGG Module completeness tables without HMM search or model gap-filling
  — use --skip_hmm or --no_genome mode instead
- A draft GSMM does not exist and you do not intend to reconstruct one (stop after
  HMM analysis with --skip_gsmm)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0610
tools:
- name: KEMET
  role: 'Core tool: HMM-based KO recovery (onebm mode) and evidence-driven GSMM gap-filling/de-novo
    reconstruction'
  repo: https://github.com/Matteopaluh/KEMET
- name: CarveMe
  role: Reconstructs reference and draft GSMMs from complete genomes and MAGs (--fbc2
    -u options)
- name: gapseq
  role: Alternative GSMM reconstruction tool for generating draft models that KEMET
    can then expand
- name: eggNOG-Mapper
  role: Produces the input KEGG KO annotations (eggNOG format) consumed by KEMET
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:039a02a3ad560555af7fc059dc6d894924082172de164c7df8b44823c18ea659
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# mag-annotation-expansion-gsmm-gapfilling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Expand KEGG functional annotations of metagenome-assembled genomes (MAGs) with KEMET's HMM-based ortholog recovery (onebm mode) and use the recovered KOs to gapfill/reconstruct genome-scale metabolic models (GSMMs), improving phenotype prediction toward complete-genome model accuracy.

## When to use

Use when you have a MAG FASTA with an existing KEGG KO annotation (eggNOG/KAAS/KofamKOALA format) and need to recover missing KOs and improve MAG-derived GSMM metabolic capability predictions.

## When NOT to use

- Input is a complete isolate genome with exhaustive annotation rather than a gapped MAG, where annotation recovery adds little value
- You only need KEGG Module completeness tables without HMM search or model gap-filling — use --skip_hmm or --no_genome mode instead
- A draft GSMM does not exist and you do not intend to reconstruct one (stop after HMM analysis with --skip_gsmm)

## Inputs

- MAG FASTA file (e.g. genomes/bin1.fasta)
- KEGG KO annotation file in eggNOG (.emapper.annotations), KAAS, or KofamKOALA format
- Optional: existing draft GSMM/GEM from CarveMe or gapseq (for --gsmm_mode existing)
- Complete genome FASTA of corresponding isolate (for reference model reconstruction)

## Outputs

- KEGG Modules completeness recap tables of metabolic potential
- List of HMM-recovered KEGG orthologs (KOs) missing from the original annotation
- KEMET-expanded GSMM/GEM with recovered KOs mapped to BiGG/ModelSEED reactions
- FVA exchange-reaction flux ranges quantifying metabolic capabilities per model set

## How to apply

Run KEMET (./kemet.py) on the MAG FASTA with a KEGG annotation file in eggNOG/KAAS/KofamKOALA format (-a flag, keeping default extensions like .emapper.annotations), using --hmm_mode onebm to search missing KOs via curated profile HMMs at the default hit-quality threshold 0.43 (overridable with --threshold_value). Use --gsmm_mode denovo to reconstruct a de-novo GSMM/GEM with recovered KOs mapped to BiGG/ModelSEED reactions, or --gsmm_mode existing to gapfill a pre-existing CarveMe/gapseq draft model. Reconstruct reference GSMMs from complete genomes using CarveMe v1.4.1 with --fbc2 -u (or gapseq). Assess metabolic capabilities with flux variability analysis (FVA) at fixed maximal growth rate across MAG-only, KEMET-expanded, and complete-genome model sets and compare exchange-reaction flux ranges. Rationale: MAG-based GSMMs are prone to reconstruction errors from gapped assemblies, and KEMET's HMM searches recover missing orthologs even from partially truncated gene sequences, reducing unsupported network gap-filling and manual curation.

## Related tools

- **KEMET** (Core tool: HMM-based KO recovery (onebm mode) and evidence-driven GSMM gap-filling/de-novo reconstruction) — https://github.com/Matteopaluh/KEMET
- **CarveMe** (Reconstructs reference and draft GSMMs from complete genomes and MAGs (--fbc2 -u options))
- **gapseq** (Alternative GSMM reconstruction tool for generating draft models that KEMET can then expand)
- **eggNOG-Mapper** (Produces the input KEGG KO annotations (eggNOG format) consumed by KEMET)

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --hmm_mode onebm --gsmm_mode denovo --log
```

## Evaluation signals

- KEMET-expanded MAG models recover exchange reactions (e.g. Selenomonas ruminantium cellobiose/sugar exchanges) absent from MAG-only models
- Expanded MAG models close roughly 10% of the gap toward complete-genome GSMM accuracy in predicting metabolite production/consumption seen in wet lab experiments
- KO mock-removal benchmark: correct gene/annotation recovery at ~55% (20/36) true positive rate for MAGs and ~67% (8/12) for complete genomes
- FVA on all three model sets (MAG-only, KEMET-expanded, complete-genome) at fixed maximal growth rate yields flux ranges for expanded models between the other two
- HMM hits pass the default quality threshold 0.43 unless explicitly changed via --threshold_value

## Limitations

- MAG-based GSMMs remain prone to reconstruction errors due to the gapped nature of metagenomic assemblies; expansion mitigates but does not eliminate this
- Network gap-fill reactions are not always supported by genomic evidence, possibly resulting in erroneous predictions
- MAGs should be high quality — the study only considered MAGs with ≥90% completeness and ≤5% contamination
- HMM recovery of truncated genes is imperfect: only 55% of KO mock removals in MAGs were correctly recovered, and recovery degrades with 30%/70% sequence truncation tests

## Evidence
