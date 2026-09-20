---
name: binary-classification-metric-computation
description: Use when profiler outputs (KMA with mOTUs/Silva/genomic databases, MetaPhlAn
  4, mOTUs 3, Bracken 2.8 on Kraken2) on CAMI in silico datasets must be scored per
  taxonomic rank against ground truth to compare binary classification performance.
when_to_use_negative:
- Input profiles are not in (or convertible to) the CAMI profiling format — the metric
  scripts expect CAMI-format profiler output
- You need abundance correlation or beta-diversity-style comparisons rather than binary
  presence/absence classification metrics
- The analysis targets ARG abundance or flanking-region assembly rather than microbial
  taxonomic profiling quality
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: OPAL
  role: Provides the evaluation framework and the low-abundance filter ('-f 1') applied
    before scoring profiler outputs
- name: KMA
  role: Profiler whose .mapstat outputs are converted to CAMI format and scored against
    mOTUs, Silva, and genomic reference databases
- name: pandas
  role: Loads profiler results for the CAMI in silico datasets and computes/aggregates
    the metric table
- name: argprofiler-tax-benchmark
  role: Snakemake pipeline hosting metric computation code, kma2cami.py, and the tocami.py-based
    conversion of profiler outputs
  repo: https://github.com/genomicepidemiology/argprofiler-tax-benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:027340026c8fc3eef9cbbde7d537470f34fdbaa3169d39d1913e64e25a743142
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genomicepidemiology/argprofiler-tax-benchmark
  tools:
  - ARGprofiler
  - Bracken
  - KMA
  - Kraken2
  - MetaPhlAn
  - OPAL
  - argprofiler-tax-benchmark
  - mOTUs
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# binary-classification-metric-computation

## Summary

Compute Purity, Completeness, and F1 per taxonomic level (superkingdom to species) to benchmark metagenomic taxonomic profilers against CAMI ground truth, enabling evidence-based profiler/database selection (e.g., KMA-mOTUs for ARGprofiler).

## When to use

Use when profiler outputs (KMA with mOTUs/Silva/genomic databases, MetaPhlAn 4, mOTUs 3, Bracken 2.8 on Kraken2) on CAMI in silico datasets must be scored per taxonomic rank against ground truth to compare binary classification performance.

## When NOT to use

- Input profiles are not in (or convertible to) the CAMI profiling format — the metric scripts expect CAMI-format profiler output
- You need abundance correlation or beta-diversity-style comparisons rather than binary presence/absence classification metrics
- The analysis targets ARG abundance or flanking-region assembly rather than microbial taxonomic profiling quality

## Inputs

- Benchmark output files of taxonomic profilers (CAMI-format profiles) from Zenodo deposit 10.5281/zenodo.7923774
- KMA .mapstat files (converted to CAMI format via kma2cami.py)
- mOTUs, MetaPhlAn, and Bracken profiler outputs (converted via tocami.py)
- Metric computation code from the argprofiler-tax-benchmark repository

## Outputs

- tax_profiler_metrics.csv with Purity, Completeness, and F1 per tool, per taxonomic level (superkingdom to species), per CAMI sampling group

## How to apply

Retrieve the benchmark output files from Zenodo deposit 10.5281/zenodo.7923774 and the metric computation code from the argprofiler-tax-benchmark repository, which first converts profiler results to CAMI format (tocami.py for mOTUs/MetaPhlAn/Bracken; kma2cami.py for KMA .mapstat files). Load the profiler results for the CAMI in silico datasets with pandas and apply the OPAL-style low-abundance filter ('-f 1') to remove low-abundance hits before scoring. At each taxonomic level from superkingdom to species, compute Purity = TP/(TP+FP), Completeness = TP/(TP+FN), and F1 = 2*purity*completeness/(purity+completeness). Aggregate metrics across the six CAMI sampling groups (including plant-associated and marine) to enable cross-environment comparison, then write the per-tool, per-level, per-group table to tax_profiler_metrics.csv.

## Related tools

- **OPAL** (Provides the evaluation framework and the low-abundance filter ('-f 1') applied before scoring profiler outputs)
- **KMA** (Profiler whose .mapstat outputs are converted to CAMI format and scored against mOTUs, Silva, and genomic reference databases)
- **pandas** (Loads profiler results for the CAMI in silico datasets and computes/aggregates the metric table)
- **argprofiler-tax-benchmark** (Snakemake pipeline hosting metric computation code, kma2cami.py, and the tocami.py-based conversion of profiler outputs) — https://github.com/genomicepidemiology/argprofiler-tax-benchmark

## Examples

```
purity = tp/(tp+fp); completeness = tp/(tp+fn); f1 = 2*purity*completeness/(purity+completeness)  # per tool/taxonomic level/group, after OPAL '-f 1' filtering
```

## Evaluation signals

- Recomputed metric table reproduces the paper's reported outcomes: KMA-mOTUs binary metrics exceed KMA-genomic and KMA-Silva in all six CAMI sampling groups
- KMA-mOTUs F1 scores are on par with MetaPhlAn and mOTUs (cf. Supplementary Material Figs A5–A6)
- KMA shows higher completeness than MetaPhlAn, mOTUs, and Bracken on plant-associated samples
- Output table tax_profiler_metrics.csv contains one row per tool × taxonomic level × sampling group with Purity, Completeness, and F1 columns
- All low-abundance hits are removed via the OPAL '-f 1' filter before scoring, so recomputation without the filter yields different Purity/Completeness values

## Limitations

- The choice of mOTUs was validated only on in silico datasets; real data may contain many more unknowns
- Benchmark covers binary presence/absence metrics only and was run on CAMI in silico communities with six sampling groups, not real metagenomes
- Profiler outputs must first be converted into CAMI format; mOTUs/MetaPhlAn/Bracken use tocami.py while KMA requires the repo's kma2cami.py for .mapstat files

## Evidence

- [methods] Compute the three binary classification metrics — Purity = TP/(TP+FP) and Completeness = TP/(TP+FN), and F1 = 2*purity*completeness/(purity+completeness) — at each taxonomic level from superkingdom to species: [evidence span withheld — anchor status: not_found]
- [results] KMA-mOTUs outperformance across sampling groups: [evidence span withheld — anchor status: not_found]
- [readme] KMA .mapstat to CAMI conversion: [evidence span withheld — anchor status: external_doc]
- [readme] Profiler output conversion: [evidence span withheld — anchor status: external_doc]
- [discussion] Real-data caveat: "Our choice was based on the in silico datasets, and while real data might contain many more unknowns"
