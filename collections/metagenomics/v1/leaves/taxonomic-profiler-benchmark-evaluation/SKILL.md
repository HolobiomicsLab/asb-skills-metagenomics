---
name: taxonomic-profiler-benchmark-evaluation
description: Use when choosing among candidate taxonomic profilers or reference databases
  for metagenomic profiling, and benchmark results (e.g. CAMI profiler outputs, KMA
  .mapstat files) need to be scored and compared.
when_to_use_negative:
- The goal is to profile a new real (non-simulated) metagenome rather than to compare
  profiler performance — use the chosen profiler directly instead of re-benchmarking
- Profiler outputs are already in OPAL/CAMI-evaluated form with metrics computed —
  re-running the scoring would duplicate work
- The data are long-read or single-cell metagenomes; the ARGprofiler benchmark validated
  mOTUs choice only on in silico short-read CAMI datasets
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: KMA
  role: Read mapper run against mOTUs, Silva, and genomic databases to produce taxonomic
    assignments scored via .mapstat files converted to CAMI format
- name: MetaPhlAn
  role: Marker-gene-based profiler benchmarked as a standalone comparison point (version
    4)
- name: mOTUs
  role: Reference sequence database for KMA mapping and standalone profiler (version
    3) in the benchmark
- name: Bracken
  role: Bayesian re-estimation profiler (version 2.8) applied to Kraken2 output as
    a benchmarked candidate
- name: Kraken2
  role: k-mer classifier whose output is re-estimated by Bracken in the benchmark
- name: OPAL
  role: Provides the low-abundance filtering convention ('-f 1') and metric style
    (Purity, Completeness, F1) used for scoring
- name: pandas
  role: Python library used to load profiler results and compute the binary classification
    metrics
- name: Snakemake
  role: Workflow engine used to implement the benchmark pipeline (Snakefile run with
    profile/config.yaml)
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: ARGprofiler
  role: Downstream AMR pipeline whose taxonomic profiling choice (KMA-mOTUs) was informed
    by this benchmark
  repo: https://github.com/genomicepidemiology/ARGprofiler
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:6f682f7e5679cf746f570d5c06c4bbcbd4e5302ce1308f85cbc1853863fe880f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genomicepidemiology/ARGprofiler
  tools:
  - ARGprofiler
  - Bracken
  - KMA
  - Kraken2
  - MetaPhlAn
  - OPAL
  - Snakemake
  - mOTUs
  - pandas
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# taxonomic-profiler-benchmark-evaluation

## Summary

Benchmark microbial profiling tool/database combinations (KMA with mOTUs, Silva, and genomic databases; MetaPhlAn 4; mOTUs 3; Bracken 2.8 on Kraken2) on CAMI in silico datasets using OPAL-style binary classification metrics to select the profiler for a downstream pipeline such as ARGprofiler.

## When to use

Use when choosing among candidate taxonomic profilers or reference databases for metagenomic profiling, and benchmark results (e.g. CAMI profiler outputs, KMA .mapstat files) need to be scored and compared.

## When NOT to use

- The goal is to profile a new real (non-simulated) metagenome rather than to compare profiler performance — use the chosen profiler directly instead of re-benchmarking
- Profiler outputs are already in OPAL/CAMI-evaluated form with metrics computed — re-running the scoring would duplicate work
- The data are long-read or single-cell metagenomes; the ARGprofiler benchmark validated mOTUs choice only on in silico short-read CAMI datasets

## Inputs

- Profiler output files for CAMI in silico datasets (from Zenodo deposit 10.5281/zenodo.7923774)
- KMA .mapstat files converted to CAMI profile format via kma2cami.py
- mOTUs, MetaPhlAn, and Bracken outputs converted to CAMI format via tocami.py
- Metric computation code from the argprofiler-tax-benchmark repository
- CAMI gold-standard ground truth profiles for the six sampling groups

## Outputs

- tax_profiler_metrics.csv — per-tool, per-taxonomic-level, per-sampling-group table of Purity, Completeness, and F1
- Comparative ranking of tool/database combinations (e.g. KMA-mOTUs vs KMA-genomic vs KMA-Silva vs MetaPhlAn vs mOTUs vs Bracken)

## How to apply

Retrieve the benchmark output files of the compared profilers (e.g. from Zenodo deposit 10.5281/zenodo.7923774) and the metric computation code from the argprofiler-tax-benchmark repository. Load the profiler results for the CAMI in silico datasets with pandas and apply the OPAL-style low-abundance filter ('-f 1') to remove low-abundance hits before scoring. Compute Purity = TP/(TP+FP), Completeness = TP/(TP+FN), and F1 = 2*purity*completeness/(purity+completeness) at each taxonomic level from superkingdom to species for each tool. Aggregate metrics across all six CAMI sampling groups (including plant-associated and marine) for cross-environment comparison, then write the per-tool, per-taxonomic-level, per-group table to tax_profiler_metrics.csv. Validation: recomputed metrics should reproduce the paper's reported outcomes (Supplementary Material Figs A5–A6), notably that KMA-mOTUs is selected for ARGprofiler.

## Related tools

- **KMA** (Read mapper run against mOTUs, Silva, and genomic databases to produce taxonomic assignments scored via .mapstat files converted to CAMI format)
- **MetaPhlAn** (Marker-gene-based profiler benchmarked as a standalone comparison point (version 4))
- **mOTUs** (Reference sequence database for KMA mapping and standalone profiler (version 3) in the benchmark)
- **Bracken** (Bayesian re-estimation profiler (version 2.8) applied to Kraken2 output as a benchmarked candidate)
- **Kraken2** (k-mer classifier whose output is re-estimated by Bracken in the benchmark)
- **OPAL** (Provides the low-abundance filtering convention ('-f 1') and metric style (Purity, Completeness, F1) used for scoring)
- **pandas** (Python library used to load profiler results and compute the binary classification metrics)
- **Snakemake** (Workflow engine used to implement the benchmark pipeline (Snakefile run with profile/config.yaml)) — https://github.com/genomicepidemiology/ARGprofiler
- **ARGprofiler** (Downstream AMR pipeline whose taxonomic profiling choice (KMA-mOTUs) was informed by this benchmark) — https://github.com/genomicepidemiology/ARGprofiler

## Examples

```
snakemake --cores 40 --use-envmodules --profile profile/ --configfile profile/config.yaml
```

## Evaluation signals

- KMA-mOTUs binary metrics (Purity, Completeness) exceed KMA-genomic and KMA-Silva in all six CAMI sampling groups
- KMA-mOTUs F1 scores are on par with MetaPhlAn and mOTUs at each taxonomic level
- KMA shows higher completeness than MetaPhlAn, mOTUs, and Bracken on plant-associated samples
- Recomputed metric table reproduces Supplementary Material Figs A5–A6
- Output file tax_profiler_metrics.csv contains one row per tool × taxonomic level (superkingdom to species) × sampling group

## Limitations

- The choice of mOTUs database was validated only on in silico CAMI datasets; real data may contain many more unknowns
- Benchmark results are specific to short-read sequencing data, consistent with ARGprofiler's short-read-only design
- Profiler outputs require conversion to CAMI format (tocami.py for mOTUs/MetaPhlAn/Bracken; kma2cami.py for KMA .mapstat files) before scoring, which can introduce format-conversion artifacts

## Evidence

- [full_text] OPAL-style low-abundance filter and metric definitions: [evidence span withheld — anchor status: not_found]
- [full_text] KMA-mOTUs benchmark finding: [evidence span withheld — anchor status: not_found]
- [readme] Readme: profiler output conversion to CAMI format: [evidence span withheld — anchor status: external_doc]
- [readme] Readme: KMA mapstat conversion: [evidence span withheld — anchor status: external_doc]
- [discussion] Discussion: mOTUs mapping in ARGprofiler: "mapping the reads against the microbial reference sequence database mOTUs"
- [discussion] Discussion: validation limited to in silico data: "Our choice was based on the in silico datasets, and while real data might contain many more unknowns"
