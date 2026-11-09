---
name: spike-in-ground-truth-accuracy-evaluation
description: Use when a prediction method (e.g., metagenomic fecal source tracking)
  must be validated against samples with known, predetermined input proportions, such
  as spike-in mesocosms or mock communities.
when_to_use_negative:
- Samples have no known composition — without ground-truth fractions, sensitivity/specificity
  cannot be computed
- Evaluating abundance accuracy for sources whose reference genomes are under-sampled
  (e.g., cow fecal microbiomes), where false negatives are systematic rather than
  method-driven
- Input is a single environmental sample without spike-in or mock-community controls
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: SourceApp
  role: 'Method under test: predicts source attribution and apportionment (cell fractions)
    that are scored against mesocosm ground truth'
  repo: https://github.com/bglindner/SourceApp
- name: CoverM
  role: Produces per-genome mapped read counts/depths from mesocosm metagenomes that
    feed the cell-fraction predictions being evaluated
  repo: https://github.com/wwood/CoverM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:939c9af8689b9f9c452a89cfd893fd1452b1f64eb53249345d0f204541e83cc4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bglindner/SourceApp
  tools:
  - CoverM
  - MicrobeCensus
  - Python
  - SourceApp
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spike-in-ground-truth-accuracy-evaluation

## Summary

Benchmarks a source-tracking or abundance-inference method against spike-in mesocosms of known composition by scoring predicted attribution/apportionment (cell fractions) against the ground-truth cell fractions established during mesocosm construction, reporting sensitivity, specificity, and residual correlations.

## When to use

Use when a prediction method (e.g., metagenomic fecal source tracking) must be validated against samples with known, predetermined input proportions, such as spike-in mesocosms or mock communities.

## When NOT to use

- Samples have no known composition — without ground-truth fractions, sensitivity/specificity cannot be computed
- Evaluating abundance accuracy for sources whose reference genomes are under-sampled (e.g., cow fecal microbiomes), where false negatives are systematic rather than method-driven
- Input is a single environmental sample without spike-in or mock-community controls

## Inputs

- Ground-truth cell fractions for each mesocosm recorded at construction time (e.g., Supporting Information Table S1)
- Shotgun metagenomic reads from spiked mesocosms (FASTQ, e.g., PRJNA1161616)
- Source-associated genome database with source-specific and cross-reactive genome flags
- Per-genome sequence depths and mapped read counts (e.g., from CoverM)

## Outputs

- Per-source sensitivity and specificity of attribution calls
- Regression/correlation statistics comparing predicted vs. true cell fractions (e.g., r2 = 0.64)
- Parameter-tuning grid results identifying optimal settings (e.g., 0.898 specificity, 0.902 sensitivity)
- Per-sample apportionment error/residual summaries

## How to apply

Construct mesocosms by spiking lake water (e.g., ~500 mL of Lake Lanier water) with fecal slurries or synthetic microbiomes at predetermined cell concentrations, quantified by DAPI fluorescence microscopy cell counting, and record the ground-truth cell fractions (Supporting Information Table S1). Shotgun sequence the mesocosms (e.g., Illumina NovaSeq 6000, 2 x 150-bp reads) after fastp quality trimming and bbnorm normalization (target=30, min=5). Run the method under test (e.g., SourceApp with 93% percent identity, 70% query coverage, LOD 0.01%, --use-geq) to obtain per-sample attribution and apportionment predictions. Score binary attribution against presence/absence truth to compute sensitivity and specificity, and regress predicted vs. true cell fractions to assess apportionment accuracy (e.g., r2 of linear correction). Tune parameters over a grid to maximize the sensitivity/specificity balance before finalizing settings.

## Related tools

- **SourceApp** (Method under test: predicts source attribution and apportionment (cell fractions) that are scored against mesocosm ground truth) — https://github.com/bglindner/SourceApp
- **CoverM** (Produces per-genome mapped read counts/depths from mesocosm metagenomes that feed the cell-fraction predictions being evaluated) — https://github.com/wwood/CoverM

## Examples

```
sourceapp.py -i mesocosm_*.fastq.gz -d sourceapp_db -o results/ --use-geq -r 0.93 -q 0.7 -l 0.0001
```

## Evaluation signals

- Attribution specificity near 0.898 and sensitivity near 0.902 with tuned parameters (93% identity, 70% overlap, LOD 0.01%)
- Predicted cell fractions should correlate with truth after linear correction, but residuals are expected (reported r2 = 0.64)
- Validation datasets (mesocosms, n = 35; spike-in slurries) are publicly available (PRJNA1161616, PRJNA1092107) for reproducible re-scoring
- Per-source error patterns should be inspected: cow sources are systematically underestimated/false-negative across samples

## Limitations

- SourceApp portions do not represent a mass balance but estimate proportions of cells belonging to source categories based on genome equivalents
- SourceApp cannot infer the age of contamination or identify nonconservative (growth-capable) populations
- Performance varies substantially by source; cows were underestimated due to under-sampling of cow fecal microbiomes
- Species-level marker genomes have largely unknown persistence in the environment

## Evidence

- [results] Optimal parameters and benchmark metrics: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance, which yielded 0.898"
- [abstract] Ground-truth design: "We performed shotgun sequencing experiments (n = 35) of mesocosms constructed from the water of a well-studied recreational and drinking water reservoir spiked with various fecal"
- [methods] Cell counting establishes truth: "Microscopy-based cell density measurements for lake water, sewage, septage, and animal fecal slurries were completed using 4′,6-diamidino-2-phenylindole (DAPI) cell staining"
- [results] Residuals limit apportionment accuracy: "substantial residuals remained after linear correction even with the best performing parameters (r 2 = 0.64"
- [abstract] Source-by-source performance variation: "SourceApp’s performance varied substantially by source, with cows being underestimated due to under sampling of cow fecal microbiomes."
