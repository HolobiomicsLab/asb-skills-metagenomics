---
name: cami-simulated-metagenome-evaluation
description: 'Decision support for choosing a taxonomic profiler or reference database:
  when in silico CAMI datasets with known ground truth are available and candidate
  tools such as KMA-mOTUs, KMA-Silva, KMA-genomic, MetaPhlAn, mOTUs, or Bracken must
  be compared quantitatively.'
when_to_use_negative:
- Evaluating profilers on real metagenomic data without ground truth — the article
  notes the choice of mOTUs 'was based on the in silico datasets, and while real data
  might contain many more unknowns'
- Comparing tools on datasets lacking per-taxon abundance ground truth, making OPAL-style
  TP/FP/FN assignment impossible
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: KMA
  role: Alignment-based profiler evaluated with three reference databases (mOTUs,
    Silva, genomic); .mapstat outputs converted with kma2cami.py
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: OPAL
  role: Benchmark metric framework; its low-abundance filter ('-f 1') is applied before
    scoring Purity, Completeness, and F1
- name: MetaPhlAn
  role: Marker-gene-based profiler used as a comparison baseline; outputs converted
    with tocami.py
- name: mOTUs
  role: Profiler and reference database; provided the best-performing KMA database
    combination
- name: Bracken
  role: Profiler run on Kraken2 output used as a comparison baseline
- name: pandas
  role: Loads profiler results and computes/aggregates the binary metrics across taxonomic
    levels and sampling groups
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:bc88f80d0e2f2ef7e6a3896bb43d93567b4177c02a37d7444fbe950108004d37
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
  - mOTUs
  - pandas
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cami-simulated-metagenome-evaluation

## Summary

Benchmark microbial profiling tools on CAMI in silico simulated metagenomes by converting profiler outputs (KMA .mapstat, mOTUs, MetaPhlAn, Bracken) to the CAMI format and scoring them with OPAL-style binary classification metrics (Purity, Completeness, F1) from superkingdom to species.

## When to use

Decision support for choosing a taxonomic profiler or reference database: when in silico CAMI datasets with known ground truth are available and candidate tools such as KMA-mOTUs, KMA-Silva, KMA-genomic, MetaPhlAn, mOTUs, or Bracken must be compared quantitatively.

## When NOT to use

- Evaluating profilers on real metagenomic data without ground truth — the article notes the choice of mOTUs 'was based on the in silico datasets, and while real data might contain many more unknowns'
- Comparing tools on datasets lacking per-taxon abundance ground truth, making OPAL-style TP/FP/FN assignment impossible

## Inputs

- Profiler output files for CAMI in silico datasets: KMA .mapstat files, mOTUs/MetaPhlAn/Bracken profiler results
- CAMI ground-truth profiles for the six in silico sampling groups
- Benchmark output files from Zenodo deposit 10.5281/zenodo.7923774
- Metric computation code from the argprofiler-tax-benchmark repository (Snakefile, kma2cami.py)

## Outputs

- Per-tool, per-taxonomic-level, per-group metric table written to tax_profiler_metrics.csv
- Binary classification metrics (Purity, Completeness, F1) from superkingdom to species for each tool/database combination

## How to apply

Retrieve profiler result files (e.g. from Zenodo deposit 10.5281/zenodo.7923774) and the metric computation code from the argprofiler-tax-benchmark repository. Convert outputs to the CAMI profile format: use tocami.py from the cami2_pipelines repository for mOTUs, MetaPhlAn, and Bracken, and kma2cami.py for KMA .mapstat files. Load results with pandas and apply the OPAL-style low-abundance filter ('-f 1') before scoring. Compute Purity = TP/(TP+FP), Completeness = TP/(TP+FN), and F1 = 2*purity*completeness/(purity+completeness) at each taxonomic level from superkingdom to species, then aggregate across the six CAMI sampling groups (including plant-associated and marine) for cross-environment comparison. In the ARGprofiler paper this benchmark showed KMA with the mOTUs database outperformed KMA-genomic and KMA-Silva on binary metrics across all six groups with F1 on par with MetaPhlAn and mOTUs, informing the choice of KMA-mOTUs for ARGprofiler's microbial abundance tables.

## Related tools

- **KMA** (Alignment-based profiler evaluated with three reference databases (mOTUs, Silva, genomic); .mapstat outputs converted with kma2cami.py) — https://github.com/genomicepidemiology/ARGprofiler
- **OPAL** (Benchmark metric framework; its low-abundance filter ('-f 1') is applied before scoring Purity, Completeness, and F1)
- **MetaPhlAn** (Marker-gene-based profiler used as a comparison baseline; outputs converted with tocami.py)
- **mOTUs** (Profiler and reference database; provided the best-performing KMA database combination)
- **Bracken** (Profiler run on Kraken2 output used as a comparison baseline)
- **pandas** (Loads profiler results and computes/aggregates the binary metrics across taxonomic levels and sampling groups)

## Examples

```
snakemake --cores 40 --use-envmodules --profile profile/ --configfile profile/config.yaml
```

## Evaluation signals

- The recomputed metric table reproduces the paper's reported outcomes: KMA-mOTUs binary metrics exceed KMA-genomic and KMA-Silva in all six sampling groups
- KMA-mOTUs F1 scores are comparable to MetaPhlAn and mOTUs (Supplementary Material Figs A5–A6)
- KMA shows higher completeness than MetaPhlAn, mOTUs, and Bracken on plant-associated samples
- tax_profiler_metrics.csv contains one row per tool x taxonomic level x sampling group, with Purity, Completeness, and F1 columns
- Profiler outputs are correctly converted to CAMI format before scoring (tocami.py for mOTUs/MetaPhlAn/Bracken, kma2cami.py for KMA .mapstat)

## Limitations

- The choice of mOTUs database was validated only on in silico datasets; real data may contain many more unknowns
- OPAL-style low-abundance filtering ('-f 1') removes low-abundance hits, which may not suit datasets with genuinely low-abundance taxa of interest
- The benchmark pipeline in argprofiler-tax-benchmark was implemented for the HPC Computerome module system and may need adaptation elsewhere

## Evidence

- [other] KMA with the mOTUs database outperformed KMA with the genomic and Silva databases on binary metrics across all six sampling groups, and its F1 scores were on par with the MetaPhlAn and mOTUs profilers: [evidence span withheld — anchor status: not_found]
- [other] Purity and Completeness formulas: [evidence span withheld — anchor status: not_found]
- [readme] Conversion scripts: [evidence span withheld — anchor status: external_doc]
- [discussion] mOTUs mapping in pipeline: "mapping the reads against the microbial reference sequence database mOTUs"
- [discussion] In silico validation caveat: "Our choice was based on the in silico datasets, and while real data might contain many more unknowns"
