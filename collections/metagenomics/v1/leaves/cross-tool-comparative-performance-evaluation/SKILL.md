---
name: cross-tool-comparative-performance-evaluation
description: 'Use when choosing among metagenomic fecal source tracking tools or claiming
  a new tool outperforms existing ones: run all candidates on the same benchmark samples
  with known source mixing proportions and tabulate sensitivity/specificity.'
when_to_use_negative:
- Input metagenomes lack any ground truth or spike-in samples with known cell fractions
  — sensitivity/specificity cannot be computed and the comparison becomes untestable
- The candidate tools require sequencing of the actual candidate sources (FEAST/metaSourceTracker
  need source profiles) and no source sequencing data are available
- The question is taxonomic profiling or coverage estimation rather than source attribution/apportionment
  of fecal signal
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: SourceApp
  role: Tool under evaluation; run with tuned parameters (93% identity, 70% query
    coverage, 0.01% LOD) to benchmark against alternatives
  repo: https://github.com/bglindner/SourceApp
- name: FEAST
  role: Alternative source tracking tool run on Kraken2 taxonomic profiles of the
    mesocosm samples with fecal slurry profiles as training data
- name: metaSourceTracker
  role: Alternative source tracking tool run on Kraken2 taxonomic profiles of the
    mesocosm samples
- name: decOM
  role: Alternative source tracking tool run on kmtricks k-mer count tables of the
    mesocosm samples
- name: Kraken2
  role: Generates default taxonomic profiles for both mesocosm and fecal slurry metagenomes,
    required by metaSourceTracker and FEAST
- name: kmtricks
  role: Generates k-mer count tables following the decOM documentation, used as decOM
    input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:ec0978cf0570cd55cac436e30ba72c3e3d47a597f0a9bad5b09e5d50c2c69a6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bglindner/SourceApp
  tools:
  - FEAST
  - Kraken2
  - Python
  - SourceApp
  - decOM
  - kmtricks
  - metaSourceTracker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# cross-tool-comparative-performance-evaluation

## Summary

Benchmark a metagenomic fecal source tracking tool (SourceApp) head-to-head against alternative methods (FEAST, metaSourceTracker, decOM) on identical spike-in mesocosm samples, scoring source attribution and apportionment against ground-truth cell fractions.

## When to use

Use when choosing among metagenomic fecal source tracking tools or claiming a new tool outperforms existing ones: run all candidates on the same benchmark samples with known source mixing proportions and tabulate sensitivity/specificity.

## When NOT to use

- Input metagenomes lack any ground truth or spike-in samples with known cell fractions — sensitivity/specificity cannot be computed and the comparison becomes untestable
- The candidate tools require sequencing of the actual candidate sources (FEAST/metaSourceTracker need source profiles) and no source sequencing data are available
- The question is taxonomic profiling or coverage estimation rather than source attribution/apportionment of fecal signal

## Inputs

- Shotgun mesocosm metagenome reads (FASTQ, BioProject PRJNA1161616, sequenced on Illumina NovaSeq 6000, 2 x 150 bp)
- Fecal spike-in slurry reads as source training data (FASTQ, BioProject PRJNA1092107)
- Ground-truth source cell fractions from mesocosm construction (Supporting Information Table S1)
- Tool-specific input formats: Kraken2 taxonomic profiles for FEAST/metaSourceTracker, kmtricks k-mer count tables for decOM, gzipped FASTQ reads for SourceApp

## Outputs

- tool_comparison_table.csv with per-tool sensitivity, specificity, and apportionment accuracy (correlation with ground truth)
- SourceApp source attribution and apportionment calls with tuned parameters (0.898 specificity, 0.902 sensitivity)

## How to apply

Download the shotgun mesocosm reads (NCBI BioProject PRJNA1161616) and the fecal spike-in slurry reads (PRJNA1092107) used as source training data. Prepare tool-specific inputs: Kraken2 taxonomic profiles (default settings) for metaSourceTracker and FEAST, and kmtricks k-mer count tables for decOM, then run each alternative on the same mesocosm samples. Run SourceApp with its tuned parameters — 93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a 0.01% relative-abundance limit of detection — to obtain source apportionments. Score every tool's predictions against the ground-truth cell fractions established during mesocosm construction (Supporting Information Table S1), computing sensitivity and specificity per tool. In this benchmark, SourceApp reached a correlation with ground truth of 0.86 versus 0.79 (FEAST), 0.74 (decOM), and 0.60 (metaSourceTracker), with alternatives showing very poor specificity (0.30 for FEAST, 0 for decOM and metaSourceTracker) while tuned SourceApp achieved 0.898 specificity and 0.902 sensitivity.

## Related tools

- **SourceApp** (Tool under evaluation; run with tuned parameters (93% identity, 70% query coverage, 0.01% LOD) to benchmark against alternatives) — https://github.com/bglindner/SourceApp
- **FEAST** (Alternative source tracking tool run on Kraken2 taxonomic profiles of the mesocosm samples with fecal slurry profiles as training data)
- **metaSourceTracker** (Alternative source tracking tool run on Kraken2 taxonomic profiles of the mesocosm samples)
- **decOM** (Alternative source tracking tool run on kmtricks k-mer count tables of the mesocosm samples)
- **Kraken2** (Generates default taxonomic profiles for both mesocosm and fecal slurry metagenomes, required by metaSourceTracker and FEAST)
- **kmtricks** (Generates k-mer count tables following the decOM documentation, used as decOM input)

## Examples

```
python sourceapp.py --input mesocosm_metagenome.fastq.gz --percent-identity 0.93 --query-coverage 0.7 --use-geq
```

## Evaluation signals

- Reproduced tool_comparison_table.csv must show SourceApp's sensitivity (0.902) and specificity (0.898) near 0.90, exceeding all alternative tools
- Apportionment accuracy (correlation with ground truth) must rank as SourceApp 0.86 > FEAST 0.79 > decOM 0.74 > metaSourceTracker 0.60
- Alternative tools should reproduce the reported specificity failure: 0.30 for FEAST and 0 for decOM and metaSourceTracker
- All tools must be evaluated on the identical mesocosm sample set scored against Supporting Information Table S1 ground-truth cell fractions
- SourceApp's tuned run must use the validated parameter set: 93% minimum identity, 70% minimum alignment-to-read overlap, 0% masking, 0.01% LOD

## Limitations

- SourceApp's performance varied substantially by source, with cows underestimated due to under sampling of cow fecal microbiomes
- SourceApp's apportioned portions do not represent a mass balance but estimate proportions of cells by genome equivalents
- Species-level marker genomes used as markers have largely unknown persistence in the environment
- No prior work had benchmarked metagenomic FST methodologies against gold standard spike-in mesocosm data, so comparative results are specific to this benchmark design

## Evidence

- [abstract] Tuned SourceApp exceeds alternatives: "Parameter tuning revealed sensitivity and specificity near 0.90 overall, which exceeded all alternative tools."
- [results] No prior benchmark exists: "To our knowledge, no work exists which has benchmarked metagenomic FST methodologies against gold standard data sets such as the spike-in mesocosms constructed herein"
- [results] Tuned parameter set: [evidence span withheld — anchor status: not_found]
- [readme] SourceApp output metric: "SourceApp reports metrics akin to prokaryotic cell fraction (when `--use-geq` is called) and has been benchmarked in laboratory trials"
- [abstract] Performance varies by source: "SourceApp’s performance varied substantially by source, with cows being underestimated due to under sampling of cow fecal microbiomes."
