---
name: cross-model-capability-comparison
description: Use when you have reconstructed GSMMs from MAGs and matched complete
  genomes and need to quantify whether annotation expansion (e.g. KEMET-recovered
  KOs) closes the metabolic-capability gap between draft and reference models.
when_to_use_negative:
- Only one model set exists (no matched complete-genome reference), so no gap-closure
  comparison is possible
- The goal is KEGG Module completeness reporting rather than phenotype/capability
  comparison — use kemet.py with --skip_hmm to get recap tables only
- Existing draft models will be rebuilt rather than extended — when a CarveMe/gapseq
  draft already exists, use KEMET --gsmm_mode existing instead of de-novo reconstruction
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_3174
tools:
- name: KEMET
  role: Recovers missing KOs via HMM profile searches and gapfills/expand GSMMs (--gsmm_mode
    denovo or existing) so expanded models can be compared against MAG-only and complete-genome
    models
  repo: https://github.com/Matteopaluh/KEMET
- name: CarveMe
  role: Reconstructs reference GSMMs from complete genomes and draft GSMMs from MAGs
    (v1.4.1, --fbc2 -u) used as the comparison endpoints
- name: gapseq
  role: Alternative tool for GSMM reconstruction from complete genomes and MAGs
- name: Python
  role: Runs FVA (flux variability analysis) and comparison of exchange-reaction flux
    ranges across model sets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:0f41964a9fb7e5a5eb65d40d75ab7d6748b6274ef5e24ca96f886e7c3d00e0d2
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
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cross-model-capability-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark genome-scale metabolic models by running flux variability analysis (FVA) on matched model sets (MAG-only, KEMET-annotation-expanded, and complete-genome reference) at fixed maximal growth rate and comparing exchange-reaction flux ranges to quantify how recovered KOs improve phenotype prediction.

## When to use

Use when you have reconstructed GSMMs from MAGs and matched complete genomes and need to quantify whether annotation expansion (e.g. KEMET-recovered KOs) closes the metabolic-capability gap between draft and reference models.

## When NOT to use

- Only one model set exists (no matched complete-genome reference), so no gap-closure comparison is possible
- The goal is KEGG Module completeness reporting rather than phenotype/capability comparison — use kemet.py with --skip_hmm to get recap tables only
- Existing draft models will be rebuilt rather than extended — when a CarveMe/gapseq draft already exists, use KEMET --gsmm_mode existing instead of de-novo reconstruction

## Inputs

- MAG FASTA files with KEGG annotation files in eggNOG/KAAS/KofamKOALA format (e.g. .emapper.annotations)
- Complete reference genomes corresponding to the MAGs (from NCBI or PATRIC/IGGdb)
- GSMM/GEM files for MAG-only, KEMET-expanded, and complete-genome model sets

## Outputs

- FVA exchange-reaction flux ranges (min/max) per model at fixed maximal growth rate
- Per-metabolite capability comparison across the three model sets vs wet-lab observations
- Quantified gap-closure (approx. 10% improvement of expanded-MAG models over MAG-only models)

## How to apply

Reconstruct three matched model sets: MAG-only GSMMs and complete-genome reference GSMMs via CarveMe v1.4.1 with --fbc2 -u (gapseq as alternative), and KEMET-expanded MAG models by running kemet.py with --hmm_mode onebm and --gsmm_mode denovo (default HMM quality threshold 0.43) so HMM-recovered KOs map to BiGG/ModelSEED reactions. For each model, fix the objective to maximal growth rate and perform flux variability analysis (FVA) to obtain min/max flux ranges for exchange reactions, which summarize metabolic capabilities (e.g. metabolite production/consumption). Compare FVA-predicted capabilities across the three sets against wet-lab-observed metabolite phenotypes, counting capabilities recovered by expansion that were absent in MAG-only models. Success is judged when expanded models close roughly 10% of the gap toward complete-genome model accuracy, with specific checks such as Selenomonas ruminantium recovering cellobiose and related sugar exchange reactions.

## Related tools

- **KEMET** (Recovers missing KOs via HMM profile searches and gapfills/expand GSMMs (--gsmm_mode denovo or existing) so expanded models can be compared against MAG-only and complete-genome models) — https://github.com/Matteopaluh/KEMET
- **CarveMe** (Reconstructs reference GSMMs from complete genomes and draft GSMMs from MAGs (v1.4.1, --fbc2 -u) used as the comparison endpoints)
- **gapseq** (Alternative tool for GSMM reconstruction from complete genomes and MAGs)
- **Python** (Runs FVA (flux variability analysis) and comparison of exchange-reaction flux ranges across model sets)

## Examples

```
./kemet.py bin1.fasta -a eggnog --hmm_mode onebm --gsmm_mode denovo --log; carved from complete genomes with CarveMe v1.4.1 --fbc2 -u before FVA comparison
```

## Evaluation signals

- KEMET-expanded MAG models recover metabolic capabilities (exchange-reaction flux ranges) absent from MAG-only models
- Expanded models close roughly 10% of the accuracy gap toward complete-genome models in predicting wet-lab-observed metabolite production/consumption
- Selenomonas ruminantium expanded model recovers cellobiose and related sugar exchange reactions
- FVA is run at the fixed maximal growth rate so flux ranges are comparable across the three model sets
- KEMET mock-removal validation benchmarks: 55% (20/36) true positive recovery rate for MAGs and 67% (8/12) for complete genomes

## Limitations

- MAG-based GSMMs are prone to reconstruction errors due to the gapped nature of metagenomic assemblies
- GSMM gap-fill reactions are not always supported by genomic evidence, possibly resulting in erroneous predictions
- The HMM hit quality filter (default 0.43) can be adjusted via --threshold_value; changing it alters which recovered KOs enter the expanded model
- MAG quality filtering matters: MAGs with ≥90% completeness and ≤5% contamination were only considered when multiple MAGs per species existed

## Evidence

- [abstract] recovered gene annotations can sensibly increase the quality of draft genome-scale metabolic models obtained from metagenome-assembled genomes, in some cases reaching the accuracy of models generated: "recovered gene annotations can sensibly increase the quality of draft genome-scale metabolic models obtained from metagenome-assembled genomes, in some cases reaching the accuracy of models generated"
- [results] GSMMs were reconstructed from complete genomes and MAGs using CarveMe v1.4.1 with the options --fbc2 -u: "GSMMs were reconstructed from complete genomes and MAGs using CarveMe v1.4.1 [10] with the options --fbc2 -u"
- [results] Flux variability analysis (FVA) was performed on the obtained GSMMs for assessing such metabolic capabilities: "Flux variability analysis (FVA) was performed on the obtained GSMMs for assessing such metabolic capabilities"
- [results] 20/36 and 8/12 KO mock removals (55% and 67% true positive rate) resulted in the correct gene and annotation recovery: "20/36 and 8/12 KO mock removals (55% and 67% true positive rate) resulted in the correct gene and annotation recovery"
- [readme] use this parameter to indicate whether to perform de-novo GSMM/GEM reconstruction or add reactions to an existing model: "when GSMM/GEM gapfilling is desired, use this parameter to indicate whether to perform de-novo GSMM/GEM reconstruction or add reactions to an existing model"
