---
name: flux-variability-analysis-phenotype-prediction
description: Use when you have draft GSMMs/GEMs (e.g. from MAGs via CarveMe or gapseq)
  and need to quantify metabolic capabilities via exchange-reaction flux ranges, or
  to measure how annotation expansion closes the phenotype-prediction gap toward complete-genome
  models.
when_to_use_negative:
- Input is raw sequence or annotation data and no metabolic model has been reconstructed
  yet — reconstruct GSMMs first (CarveMe/gapseq) before FVA is possible.
- The goal is KEGG Module completeness evaluation or KO recovery alone — use KEMET
  with --skip_gsmm instead, since FVA requires a fully built model.
- Network gap-fill reactions lack genomic evidence — article warns such reactions
  'are not always supported by genomic evidence, possibly resulting in erroneous predictions',
  so FVA capabilities from evidence-free gap-filling should not be treated as validated.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2259
tools:
- name: KEMET
  role: Expands MAG-derived GSMMs with HMM-recovered KOs mapped to BiGG/ModelSEED
    reactions (--gsmm_mode denovo or existing, --hmm_mode onebm) before FVA comparison
  repo: https://github.com/Matteopaluh/KEMET
- name: CarveMe
  role: Reconstructs GSMMs from complete genomes and MAGs with --fbc2 -u, producing
    the models on which FVA is performed
- name: gapseq
  role: Alternative GSMM reconstruction tool mentioned as interchangeable with CarveMe
    in the article
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:40beb8d7303fa542b44fb4282e3c2caeb7232595f617c804ff2ea6acaca7fdeb
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
  - gapseq
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# flux-variability-analysis-phenotype-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Use flux variability analysis (FVA) at fixed maximal growth rate on genome-scale metabolic models (MAG-derived, annotation-expanded, and complete-genome) to compare predicted metabolite production/consumption capabilities against wet-lab observed phenotypes.

## When to use

Use when you have draft GSMMs/GEMs (e.g. from MAGs via CarveMe or gapseq) and need to quantify metabolic capabilities via exchange-reaction flux ranges, or to measure how annotation expansion closes the phenotype-prediction gap toward complete-genome models.

## When NOT to use

- Input is raw sequence or annotation data and no metabolic model has been reconstructed yet — reconstruct GSMMs first (CarveMe/gapseq) before FVA is possible.
- The goal is KEGG Module completeness evaluation or KO recovery alone — use KEMET with --skip_gsmm instead, since FVA requires a fully built model.
- Network gap-fill reactions lack genomic evidence — article warns such reactions 'are not always supported by genomic evidence, possibly resulting in erroneous predictions', so FVA capabilities from evidence-free gap-filling should not be treated as validated.

## Inputs

- GSMM/GEM files (e.g. SBML fbc2 from CarveMe) for MAG-only, KEMET-expanded, and complete-genome model sets
- MAG/genome FASTA files with KEGG annotation files in eggNOG/KAAS/KofamKOALA format (e.g. .emapper.annotations) for KEMET expansion
- Wet-lab observed metabolite production/consumption phenotypes for benchmarking

## Outputs

- Per-model exchange-reaction flux ranges (FVA min/max) at fixed maximal growth rate
- Comparison of predicted metabolic capabilities across the three model sets vs. observed phenotypes
- Quantified improvement (~10%) of expanded-MAG models over MAG-only models toward complete-genome accuracy

## How to apply

Reconstruct GSMMs from MAGs and from the corresponding complete genomes using CarveMe v1.4.1 with --fbc2 -u (or gapseq), and generate an expanded model set by running KEMET (./kemet.py) with --hmm_mode onebm (default HMM hit threshold 0.43) and --gsmm_mode denovo so HMM-recovered KOs are mapped to BiGG/ModelSEED reactions; where a CarveMe/gapseq draft already exists, use --gsmm_mode existing to add recovered reactions instead of rebuilding. Then run flux variability analysis on all three model sets (MAG-only, KEMET-expanded, complete-genome) at fixed maximal growth rate, recording exchange-reaction flux ranges as the metabolic capabilities of each model. Compare the FVA-predicted capabilities against metabolite production/consumption phenotypes observed in wet-lab experiments, counting which capabilities each model set recovers. Success is judged when KEMET-expanded MAG models recover capabilities absent from MAG-only models (e.g. the Selenomonas ruminantium cellobiose and related sugar exchange reactions), closing roughly 10% of the gap toward complete-genome model accuracy.

## Related tools

- **KEMET** (Expands MAG-derived GSMMs with HMM-recovered KOs mapped to BiGG/ModelSEED reactions (--gsmm_mode denovo or existing, --hmm_mode onebm) before FVA comparison) — https://github.com/Matteopaluh/KEMET
- **CarveMe** (Reconstructs GSMMs from complete genomes and MAGs with --fbc2 -u, producing the models on which FVA is performed)
- **gapseq** (Alternative GSMM reconstruction tool mentioned as interchangeable with CarveMe in the article)

## Examples

```
./kemet.py mag.fasta -a eggnog --hmm_mode onebm --gsmm_mode denovo; carve --fbc2 -u mag.fasta
```

## Evaluation signals

- KEMET-expanded MAG models recover metabolic capabilities absent from MAG-only models, including the Selenomonas ruminantium cellobiose and related sugar exchange reactions
- Expanded-MAG FVA capability agreement with wet-lab phenotypes improves by ~10%, reaching accuracy comparable to complete-genome GSMMs
- FVA is run at fixed maximal growth rate so exchange-reaction flux ranges are comparable across the three model sets
- KO mock-removal recovery checks: 55% (20/36) true positive rate for MAGs and 67% (8/12) for complete genomes when validating HMM-based KO recovery
- MAGs used for model building meet ≥90% completeness and ≤5% contamination filters

## Limitations

- MAG-based GSMMs are prone to reconstruction errors due to the gapped nature of metagenomic assemblies
- Network gap-fill reactions added during reconstruction are not always supported by genomic evidence, possibly resulting in erroneous predictions
- HMM-recovery validation showed only 55% (20/36) of mock-removed KOs recovered for MAGs (67% for complete genomes), so expansion is partial rather than exhaustive

## Evidence

- [results] FVA workflow: "Flux variability analysis (FVA) was performed on the obtained GSMMs for assessing such metabolic capabilities"
- [results] GSMM reconstruction options: "GSMMs were reconstructed from complete genomes and MAGs using CarveMe v1.4.1 [10] with the options --fbc2 -u"
- [abstract] KEMET expansion quality finding: "recovered gene annotations can sensibly increase the quality of draft genome-scale metabolic models obtained from metagenome-assembled genomes"
