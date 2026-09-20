---
name: cross-environment-profiler-comparison-analysis
description: Decision support for choosing a taxonomic profiling tool and reference
  database when in silico ground-truth data (e.g. CAMI) are available and per-tool
  performance must be compared across multiple environments or sampling groups.
when_to_use_negative:
- Input consists of real (non-simulated) metagenomic data without ground truth — no
  true positives can be defined, so binary classification metrics cannot be computed
  (the article notes real data may contain many more unknowns)
- Input is raw sequencing reads rather than already-produced profiler output profiles
  that can be converted to CAMI format
- The goal is ARG abundance quantification rather than taxonomic profiler benchmarking
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: KMA
  role: Mapper whose .mapstat output, when paired with mOTUs/Silva/genomic reference
    databases, is converted to CAMI format and scored as a candidate profiler
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: MetaPhlAn
  role: Marker-gene-based profiler benchmarked as the comparison standard for F1 scores
- name: mOTUs
  role: Profiler benchmarked against KMA-mOTUs; its database is also used as KMA's
    reference
- name: Bracken
  role: Profiler run on Kraken2 output (Bracken 2.8) benchmarked for completeness
    on plant-associated samples
- name: Kraken2
  role: Classifier whose output feeds Bracken in the benchmark
- name: pandas
  role: Used to load the profiler result files for CAMI in silico datasets and compute
    the metric table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:7c655b8098a58ff7baee648062508b6d8b2dbc058e19372c08083fa6311633b4
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
  - mOTUs
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-environment-profiler-comparison-analysis

## Summary

Benchmark multiple metagenomic taxonomic profilers (e.g. KMA with different reference databases, MetaPhlAn 4, mOTUs 3, Bracken 2.8/Kraken2) on CAMI in silico datasets by computing OPAL-style binary classification metrics (Purity, Completeness, F1) per taxonomic level and per sampling group, to select the best profiler–database combination.

## When to use

Decision support for choosing a taxonomic profiling tool and reference database when in silico ground-truth data (e.g. CAMI) are available and per-tool performance must be compared across multiple environments or sampling groups.

## When NOT to use

- Input consists of real (non-simulated) metagenomic data without ground truth — no true positives can be defined, so binary classification metrics cannot be computed (the article notes real data may contain many more unknowns)
- Input is raw sequencing reads rather than already-produced profiler output profiles that can be converted to CAMI format
- The goal is ARG abundance quantification rather than taxonomic profiler benchmarking

## Inputs

- Profiler output files for CAMI in silico datasets (KMA .mapstat files, mOTUs/MetaPhlAn/Bracken outputs) from Zenodo deposit 10.5281/zenodo.7923774
- CAMI-format profiler profiles after conversion (tocami.py / kma2cami.py)
- Metric computation code from the argprofiler-tax-benchmark repository

## Outputs

- tax_profiler_metrics.csv — per-tool, per-taxonomic-level (superkingdom to species), per-sampling-group table of Purity, Completeness, and F1 scores
- Cross-environment comparison plots (e.g. Supplementary Figs A5–A6)

## How to apply

Retrieve benchmark output files (e.g. the Zenodo deposit 10.5281/zenodo.7923774) and metric computation code, then load the profiler results for the CAMI in silico datasets with pandas. Convert profiler outputs to CAMI format (tocami.py for mOTUs/MetaPhlAn/Bracken; kma2cami.py for KMA .mapstat files) and apply the OPAL-style low-abundance filter ('-f 1') before scoring. Compute Purity = TP/(TP+FP), Completeness = TP/(TP+FN), and F1 = 2*purity*completeness/(purity+completeness) at each taxonomic level from superkingdom to species for each tool. Aggregate the metrics across all sampling groups (including plant-associated and marine) so cross-environment comparison is possible. In the ARGprofiler case this showed KMA-mOTUs outperforming KMA-genomic and KMA-Silva on binary metrics in all six groups, with F1 on par with MetaPhlAn and mOTUs, which drove the adoption of KMA-mOTUs.

## Related tools

- **KMA** (Mapper whose .mapstat output, when paired with mOTUs/Silva/genomic reference databases, is converted to CAMI format and scored as a candidate profiler) — https://github.com/genomicepidemiology/ARGprofiler
- **MetaPhlAn** (Marker-gene-based profiler benchmarked as the comparison standard for F1 scores)
- **mOTUs** (Profiler benchmarked against KMA-mOTUs; its database is also used as KMA's reference)
- **Bracken** (Profiler run on Kraken2 output (Bracken 2.8) benchmarked for completeness on plant-associated samples)
- **Kraken2** (Classifier whose output feeds Bracken in the benchmark)
- **pandas** (Used to load the profiler result files for CAMI in silico datasets and compute the metric table)

## Examples

```
snakemake --cores 40 --use-envmodules --profile profile/ --configfile profile/config.yaml
```

## Evaluation signals

- KMA-mOTUs binary metrics (Purity, Completeness) exceed KMA-genomic and KMA-Silva in all six CAMI sampling groups
- KMA-mOTUs F1 scores are comparable to MetaPhlAn and mOTUs (as in Supplementary Figs A5–A6)
- Metric table reproduces Purity = TP/(TP+FP), Completeness = TP/(TP+FN), F1 = 2*purity*completeness/(purity+completeness) at each level from superkingdom to species
- All six sampling groups (including plant-associated and marine) are represented in the aggregated comparison
- Output file tax_profiler_metrics.csv contains per-tool, per-level, per-group rows matching the paper's reported outcomes

## Limitations

- The choice of mOTUs as KMA's reference database was validated only on in silico CAMI datasets; real data may contain many more unknowns
- OPAL-style low-abundance filtering ('-f 1') removes low-abundance hits before scoring, so results depend on that filter choice
- Comparison is restricted to short-read in silico data and the specific profiler versions tested (MetaPhlAn 4, mOTUs 3, Bracken 2.8 on Kraken2)

## Evidence

- [other] KMA with the mOTUs database outperformed KMA with the genomic and Silva databases on binary metrics across all six sampling groups, and its F1 scores were on par with the MetaPhlAn and mOTUs profilers: [evidence span withheld — anchor status: not_found]
- [other] Compute the three binary classification metrics: [evidence span withheld — anchor status: not_found]
- [readme] Convert profiler outputs to CAMI format: [evidence span withheld — anchor status: external_doc]
- [discussion] Choice of mOTUs validated only on in silico datasets: "Our choice was based on the in silico datasets, and while real data might contain many more unknowns"
- [discussion] Mapping reads against mOTUs database: "mapping the reads against the microbial reference sequence database mOTUs"
