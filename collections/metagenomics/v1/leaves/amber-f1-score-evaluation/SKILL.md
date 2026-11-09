---
name: amber-f1-score-evaluation
description: Reference for reproducing or extending metagenomic taxonomic binning
  benchmarks that need rank-based F1-scores computed with AMBER over multiple tools,
  samples, and reference database configurations.
when_to_use_negative:
- The goal is taxonomic profiling with abundance estimation rather than taxonomic
  binning — profiling evaluation uses different metrics (balanced L1-norm error) not
  covered by this rank-based F1 workflow
- Input samples are real (not simulated) reads without ground-truth binning assignments,
  since AMBER evaluation requires bioboxes-format ground truth
- The comparison is of index size or build time rather than classification accuracy
  — database size metrics (.build.size.tsv) are produced separately
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: AMBER 2.0.21-beta
  role: Computes rank-based and cumulative binning evaluation tables (tp, fp, sens,
    prec, f1s) from bioboxes-format predictions and ground truth
  repo: https://github.com/CAMI-challenge/AMBER/archive/658a268bc86622e6f41304d38c1856b727c995ed.tar.gz
- name: Snakemake >=5.2.0
  role: Orchestrates the classify pipeline (classify/Snakefile) that runs all classifiers
    over samples and database configurations
- name: ganon2
  role: Primary classifier under benchmark evaluation; predictions are converted to
    bioboxes and scored with AMBER
  repo: https://github.com/pirovc/ganon
- name: kraken2
  role: Competing classifier evaluated in default binning mode for median F1-score
    comparison
- name: metacache
  role: Competing classifier (v2.3.1) evaluated in default binning mode for median
    F1-score comparison
  repo: https://github.com/pirovc/metabench
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqaf094
    title: 'ganon2: up-to-date and scalable metagenomics analysis'
  source_content_hash: sha256:92f55b0eb61c37c1dc3169038ea021e8d64bdb01bf31ef13f89968f04529b40f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AMBER
  - AMBER 2.0.21-beta
  - Snakemake >=5.2.0
  - conda
  - ganon2
  - kraken2
  - metacache
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# amber-f1-score-evaluation

## Summary

A benchmarking skill for evaluating taxonomic binning performance: convert classifier predictions and ground truth into AMBER bioboxes format, run AMBER 2.0.21-beta, and aggregate species-level F1-scores across simulated samples, as done to compare ganon2, kraken2, and Metacache on 16 CAMI-derived short-read samples against full NCBI RefSeq.

## When to use

Reference for reproducing or extending metagenomic taxonomic binning benchmarks that need rank-based F1-scores computed with AMBER over multiple tools, samples, and reference database configurations.

## When NOT to use

- The goal is taxonomic profiling with abundance estimation rather than taxonomic binning — profiling evaluation uses different metrics (balanced L1-norm error) not covered by this rank-based F1 workflow
- Input samples are real (not simulated) reads without ground-truth binning assignments, since AMBER evaluation requires bioboxes-format ground truth
- The comparison is of index size or build time rather than classification accuracy — database size metrics (.build.size.tsv) are produced separately

## Inputs

- 16 simulated short-read samples (.fq.gz) from studies including CAMI 1+2 challenge
- Ground-truth binning files (readid_assembly_taxid.txt) with accession.version, assembly id, and taxonomic id
- Classifier prediction files from ganon2, kraken2, Metacache (and kmcp where feasible) in default binning mode
- NCBI taxdump files (nodes.dmp, names.dmp, merged.dmp)
- AMBER 2.0.21-beta installation (tools_path configured in Snakemake config)

## Outputs

- amber.gt.bioboxes files (bioboxes-format ground truth and predictions)
- *.rank.tsv per tool-sample-database combination with gt, db, class, tp, fp, sens, prec, f1s
- *.cumu.tsv cumulative evaluation tables
- Per-sample species-level F1-scores and per-tool median F1-scores (e.g. ganon2 ~0.77, kraken2 ~0.61, Metacache ~0.45 on full RefSeq)
- .time and .stats files with runtime and peak memory per tool run

## How to apply

Run the classification tools (ganon2, kraken2, Metacache, kmcp) in default binning mode on the simulated .fq.gz short-read samples, wrapping each run with '/usr/bin/time -v' to capture runtime and peak memory in .time and .stats files. Convert each tool's predictions and the ground-truth files (accession.version, assembly id, taxonomic id) into the AMBER bioboxes format (amber.gt.bioboxes). Run AMBER 2.0.21-beta for every tool-sample-database combination to produce rank-based (*.rank.tsv) and cumulative (*.cumu.tsv) evaluation tables containing gt, db, class, tp, fp, sens, prec and f1s. Aggregate species-level f1s values from the *.rank.tsv files across the 16 samples to compute per-sample F1-scores and per-tool medians, and extract runtime and peak memory from the .stats files. The Snakemake classify pipeline (classify/Snakefile with --use-conda --cores 48 -k and classify_toy_taxid.yaml) orchestrates the classification step across database configurations (RefSeqOLD, RefSeqCG, RefSeqCG-top-3, RefSeqALL, RefSeqALL-top-3).

## Related tools

- **AMBER 2.0.21-beta** (Computes rank-based and cumulative binning evaluation tables (tp, fp, sens, prec, f1s) from bioboxes-format predictions and ground truth) — https://github.com/CAMI-challenge/AMBER/archive/658a268bc86622e6f41304d38c1856b727c995ed.tar.gz
- **Snakemake >=5.2.0** (Orchestrates the classify pipeline (classify/Snakefile) that runs all classifiers over samples and database configurations)
- **ganon2** (Primary classifier under benchmark evaluation; predictions are converted to bioboxes and scored with AMBER) — https://github.com/pirovc/ganon
- **kraken2** (Competing classifier evaluated in default binning mode for median F1-score comparison)
- **metacache** (Competing classifier (v2.3.1) evaluated in default binning mode for median F1-score comparison) — https://github.com/pirovc/metabench

## Examples

```
snakemake -s classify/Snakefile --configfile config/classify_toy_taxid.yaml --use-conda --cores 48 -k
```

## Evaluation signals

- Per-tool median species-level binning F1-scores on full RefSeq should match reported values: ganon2 ~0.77, kraken2 ~0.61, Metacache ~0.45
- All tools should reach ~0.55 median species-level F1-score on the RefSeq RG configuration
- Runtime and peak memory values extracted from .stats files should be consistent with Table 4 of the manuscript
- *.rank.tsv files must contain the columns gt, db, class, tp, fp, sens, prec, f1s for every tool-sample-database combination
- Ground-truth files must convert cleanly into amber.gt.bioboxes format with accession.version, assembly id, and taxonomic id fields

## Limitations

- kmcp evaluation may not be feasible for all database configurations ('kmcp where feasible')
- The workflow depends on pinned AMBER 2.0.21-beta and requires updating 'tools_path' in the configuration files to point at the AMBER installation
- Evaluation relies on simulated samples with ground truth (including a 1 million read subset provided under files/reads/); real reads cannot be scored this way

## Evidence

- [methods] Run AMBER 2.0.21-beta for every tool-sample-database combination to produce rank-based (*.rank.tsv) and cumulative (*.cumu.tsv) evaluation tables containing gt, db, class, tp, fp, sens, prec and f1s.: "AMBER==2.0.21-beta (https://github.com/CAMI-challenge/AMBER/archive/658a268bc86622e6f41304d38c1856b727c995ed.tar.gz)"
- [abstract] With the full RefSeq, ganon2 achieved the highest species-level binning median F1-score of 0.77, followed by kraken2 with a median of 0.61 and Metacache with 0.45.: "Using 16 simulated samples from various studies, including the CAMI 1+2 challenge, ganon2 achieved up to 0.15 higher median F1-score in taxonomic binning."
- [readme] The pipeline is divided into build and classify steps.: "The pipeline is divided into build and classify steps.
- The folder config/ contains examples of configuration files for building and classifying"
- [readme] It requires reference sequences to build databases and a set of reads in fastq and ground truth in the bioboxes format.: "It requires:
- Reference sequences to build databases
- Set of reads in fastq and ground truth in the bioboxes format"
