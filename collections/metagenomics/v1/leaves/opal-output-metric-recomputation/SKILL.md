---
name: opal-output-metric-recomputation
description: Decision support for comparing metagenomic taxonomic profilers (KMA variants,
  MetaPhlAn, mOTUs, Bracken) on benchmark data with known ground truth, when per-level
  Purity/Completeness/F1 must be recomputed from profiler output files rather than
  trusted from published figures.
when_to_use_negative:
- The data are real metagenomic samples without ground-truth profiles — the paper's
  own caveat is that 'Our choice was based on the in silico datasets, and while real
  data might contain many more unknowns', so these binary metrics are not meaningful
  without CAMI-style truth labels.
- The profiler outputs are already in non-CAMI native formats and the goal is only
  qualitative inspection rather than uniform metric recomputation against a gold standard.
- Abundance estimation (not profiling accuracy) is the target metric — Purity/Completeness/F1
  measure taxon classification, not quantitative abundance accuracy.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: OPAL
  role: Provides the scoring protocol, including the low-abundance filter ('-f 1'),
    under which Purity/Completeness/F1 are recomputed
- name: KMA
  role: Read-mapper whose .mapstat outputs (against mOTUs, Silva, and genomic databases)
    are converted to CAMI format and scored
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: MetaPhlAn 4
  role: Marker-gene profiler whose output is converted to CAMI format via tocami.py
    and used as the F1 comparison baseline
- name: mOTUs 3
  role: Profiler compared against KMA-mOTUs; also serves as KMA's reference database
- name: Bracken 2.8 (on Kraken2)
  role: Abundance re-estimator whose output is converted to CAMI format via tocami.py
    and benchmarked
- name: pandas
  role: Loads profiler result files and computes the purity/completeness/F1 table
- name: argprofiler-tax-benchmark
  role: Snakemake pipeline supplying kma2cami.py, tocami.py integration, and metric
    visualization code
  repo: https://github.com/genomicepidemiology/argprofiler-tax-benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:84d44db26ab115eaa2aea332a51934094262dcbb9bb5c3c93cd696f110512ac3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genomicepidemiology/ARGprofiler
  tools:
  - ARGprofiler
  - Bracken
  - Bracken 2.8 (on Kraken2)
  - KMA
  - Kraken2
  - MetaPhlAn
  - MetaPhlAn 4
  - OPAL
  - argprofiler-tax-benchmark
  - mOTUs
  - mOTUs 3
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# opal-output-metric-recomputation

## Summary

Recompute OPAL-style binary classification metrics (Purity, Completeness, F1) from microbial profiler outputs on the CAMI in silico datasets, applying a low-abundance filter before scoring. This reproduces the benchmark that justified selecting KMA-mOTUs as the taxonomic profiler for ARGprofiler.

## When to use

Decision support for comparing metagenomic taxonomic profilers (KMA variants, MetaPhlAn, mOTUs, Bracken) on benchmark data with known ground truth, when per-level Purity/Completeness/F1 must be recomputed from profiler output files rather than trusted from published figures.

## When NOT to use

- The data are real metagenomic samples without ground-truth profiles — the paper's own caveat is that 'Our choice was based on the in silico datasets, and while real data might contain many more unknowns', so these binary metrics are not meaningful without CAMI-style truth labels.
- The profiler outputs are already in non-CAMI native formats and the goal is only qualitative inspection rather than uniform metric recomputation against a gold standard.
- Abundance estimation (not profiling accuracy) is the target metric — Purity/Completeness/F1 measure taxon classification, not quantitative abundance accuracy.

## Inputs

- Benchmark profiler output files from Zenodo deposit 10.5281/zenodo.7923774 (KMA-mOTUs, KMA-Silva, KMA-genomic, MetaPhlAn 4, mOTUs 3, Bracken 2.8/Kraken2)
- CAMI-format gold-standard profiles for the six in silico sampling groups
- Metric computation code from the genomicepidemiology/argprofiler-tax-benchmark repository (tocami.py, kma2cami.py, OPAL-style scoring)
- KMA .mapstat files for conversion to CAMI format

## Outputs

- tax_profiler_metrics.csv — per-tool, per-taxonomic-level (superkingdom to species), per-sampling-group Purity/Completeness/F1 table
- Aggregated per-group metric summaries supporting cross-environment comparison

## How to apply

Retrieve the benchmark output files from Zenodo deposit 10.5281/zenodo.7923774 and the metric computation code from the argprofiler-tax-benchmark repository, which converts KMA .mapstat files to CAMI format via kma2cami.py and converts mOTUs/MetaPhlAn/Bracken results with the tocami.py script from the CAMI2 pipelines. Load the profiler results with pandas and apply the OPAL-style low-abundance filter ('-f 1') to drop low-abundance hits before scoring. Compute Purity = TP/(TP+FP), Completeness = TP/(TP+FN), and F1 = 2*purity*completeness/(purity+completeness) at each taxonomic level from superkingdom to species for KMA with the mOTUs, Silva, and genomic databases, MetaPhlAn 4, mOTUs 3, and Bracken 2.8 on Kraken2 output. Aggregate per tool, per taxonomic level, and per CAMI sampling group (six groups, including plant-associated and marine), then write the table to tax_profiler_metrics.csv. The rationale is that recomputation under one uniform scoring protocol (OPAL with '-f 1') is the only fair way to justify ARGprofiler's profiler choice across databases and profilers.

## Related tools

- **OPAL** (Provides the scoring protocol, including the low-abundance filter ('-f 1'), under which Purity/Completeness/F1 are recomputed)
- **KMA** (Read-mapper whose .mapstat outputs (against mOTUs, Silva, and genomic databases) are converted to CAMI format and scored) — https://github.com/genomicepidemiology/ARGprofiler
- **MetaPhlAn 4** (Marker-gene profiler whose output is converted to CAMI format via tocami.py and used as the F1 comparison baseline)
- **mOTUs 3** (Profiler compared against KMA-mOTUs; also serves as KMA's reference database)
- **Bracken 2.8 (on Kraken2)** (Abundance re-estimator whose output is converted to CAMI format via tocami.py and benchmarked)
- **pandas** (Loads profiler result files and computes the purity/completeness/F1 table)
- **argprofiler-tax-benchmark** (Snakemake pipeline supplying kma2cami.py, tocami.py integration, and metric visualization code) — https://github.com/genomicepidemiology/argprofiler-tax-benchmark

## Examples

```
snakemake --cores 40 --use-envmodules --profile profile/ --configfile profile/config.yaml
```

## Evaluation signals

- Recomputed metric table reproduces the paper's reported outcomes: KMA-mOTUs binary metrics exceed KMA-genomic and KMA-Silva in all six CAMI sampling groups (Supplementary Material Figs A5–A6).
- KMA-mOTUs F1 scores are on par with MetaPhlAn 4 and mOTUs 3 at each taxonomic level.
- KMA shows higher Completeness than MetaPhlAn, mOTUs, and Bracken on the plant-associated samples.
- Schema check: tax_profiler_metrics.csv contains one row per tool × taxonomic level × sampling group, with purity, completeness, and F1 columns satisfying F1 = 2*purity*completeness/(purity+completeness).
- Invariant: applying the OPAL '-f 1' low-abundance filter must remove low-abundance hits before scoring, changing (typically improving) purity relative to unfiltered scoring.

## Limitations

- The mOTUs choice was validated only on in silico datasets; real metagenomic data may contain many more unknowns, so the metrics may not transfer to real samples.
- The benchmark pipeline was implemented to run with the module system on the Computerome HPC; reproducing it elsewhere requires adapting the Snakemake profile.
- Scores cover only three binary metrics at standard taxonomic levels; abundance-level accuracy (e.g., L1 distance) is not part of this recomputation.

## Evidence

- [readme] To convert the results of mOTUs, MetaPhlAn, and bracken we used the tocami.py script: [evidence span withheld — anchor status: external_doc]
- [readme] Using the .mapstat files produced by KMA, we converted them into the CAMI format: [evidence span withheld — anchor status: external_doc]
- [discussion] Choice of mOTUs validated only on in silico datasets: "Our choice was based on the in silico datasets, and while real data might contain many more unknowns"
- [readme] Pipeline implemented in Snakemake for Computerome HPC: [evidence span withheld — anchor status: external_doc]
- [abstract] Paper objective: produce abundance tables of genes and microbes: "produce abundance tables of genes and microbes"
