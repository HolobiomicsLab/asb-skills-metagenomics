---
name: kmer-based-sequence-classification
description: Use when metagenomic FASTQ reads must be taxonomically binned against
  large RefSeq reference sets, especially at strain/assembly level where default ganon2
  cutoff and filter settings need threshold sweeping to reach acceptable sensitivity/precision.
when_to_use_negative:
- The task is metagenomic profiling with abundance estimation rather than taxonomic
  binning/classification cutoff tuning
- The input is already a binning/profiling result table — no FASTQ reads or reference
  index are available to classify
- References lack taxonomy mapping (no nodes.dmp/names.dmp/merged.dmp or acc_len_taxid_assembly),
  so ground-truth evaluation at assembly/strain level is impossible
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: ganon2
  role: k-mer/Bloom-filter-based classifier that indexes large RefSeq reference sets
    and performs taxonomic binning; cutoff and filter thresholds are swept for strain-level
    sensitivity/precision tuning
  repo: https://github.com/pirovc/ganon
- name: Snakemake
  role: workflow engine (>=5.2.0) running the build and classify Snakefiles with conda-managed
    dependencies (--use-conda)
- name: AMBER 2.0.21-beta
  role: evaluation of binning predictions against ground truth in rank-based mode,
    yielding sensitivity, precision, and f1-score in *.rank.tsv outputs
  repo: https://github.com/CAMI-challenge/AMBER
- name: genome_updater.sh
  role: downloads and keeps up-to-date RefSeq genomic .fna reference sequences with
    assembly_summary.txt and taxdump
  repo: https://raw.githubusercontent.com/pirovc/genome_updater/a301b927362aced47ed88a20b3f396df0bf5258d/genome_updater.sh
- name: metabench
  role: benchmarking pipeline (Snakemake) that structures database builds, multi-parameter
    classification runs (e.g. --rel-cutoff grid), and standardized evaluation output
  repo: https://github.com/pirovc/metabench
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqaf094
    title: 'ganon2: up-to-date and scalable metagenomics analysis'
  source_content_hash: sha256:2f8bff60f576fdba0b71a368b0c87085d098ede1a4ea5a9bf01a55371c6a6cde
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pirovc/ganon
  tools:
  - AMBER 2.0.21-beta
  - Snakemake
  - conda
  - ganon2
  - genome_updater.sh
  - metabench
  - wget
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 18
---

# kmer-based-sequence-classification

## Summary

A k-mer/interleaved-bloom-filter-based sequence classification workflow using ganon2 to bin metagenomic reads against large NCBI RefSeq reference sets, including strain-level classification where cutoff and filter thresholds must be swept to trade sensitivity against precision.

## When to use

Use when metagenomic FASTQ reads must be taxonomically binned against large RefSeq reference sets, especially at strain/assembly level where default ganon2 cutoff and filter settings need threshold sweeping to reach acceptable sensitivity/precision.

## When NOT to use

- The task is metagenomic profiling with abundance estimation rather than taxonomic binning/classification cutoff tuning
- The input is already a binning/profiling result table — no FASTQ reads or reference index are available to classify
- References lack taxonomy mapping (no nodes.dmp/names.dmp/merged.dmp or acc_len_taxid_assembly), so ground-truth evaluation at assembly/strain level is impossible

## Inputs

- RefSeq genomic .fna reference sequence files
- acc_len_taxid_assembly auxiliary file (accession.version, sequence length, taxid, assembly id)
- NCBI taxdump files (nodes.dmp, names.dmp, merged.dmp)
- CAMI strain-madness simulated FASTQ read sets
- Ground truth readid/assembly/taxid files (*readid_assembly_taxid.txt)
- assembly_summary.txt from genome_updater download

## Outputs

- ganon2 index files (ganon_db.hibf, ganon_db.ibf, ganon_db.tax) with prefix.check/prefix.log/prefix.time build artifacts
- Classification predictions (read id, assembly id, taxid) per read
- AMBER rank-based evaluation tables (*.rank.tsv) with sensitivity, precision, and f1-score
- Compiled threshold sweep table (strain_threshold_sweep_rank.tsv)

## How to apply

Assemble the reference set from RefSeq genomic .fna files together with the acc_len_taxid_assembly auxiliary file (accession.version, sequence length, taxid, assembly id) and matching nodes.dmp, names.dmp, and merged.dmp from the NCBI taxdump, downloading references with genome_updater.sh. Build the ganon2 assembly-level index via the build Snakefile under Snakemake >=5.2.0 with --use-conda (conda-managed dependencies), recording prefix.check, prefix.log, and prefix.time artifacts. Classify the CAMI strain-madness read sets with the classify Snakefile at assembly level, writing read id / assembly id / taxid predictions. Sweep the ganon2 cutoff and filter thresholds across a grid of candidate values (e.g. --rel-cutoff values like 0.25 and 0.8 as configured in metabench), re-running classification per setting. Evaluate each setting against the ground truth (accession.version, assembly id, taxonomic id) with AMBER 2.0.21-beta in rank-based mode, extracting sensitivity, precision, and f1-score from the *.rank.tsv outputs, and select the configuration with the highest strain-level F1-score.

## Related tools

- **ganon2** (k-mer/Bloom-filter-based classifier that indexes large RefSeq reference sets and performs taxonomic binning; cutoff and filter thresholds are swept for strain-level sensitivity/precision tuning) — https://github.com/pirovc/ganon
- **Snakemake** (workflow engine (>=5.2.0) running the build and classify Snakefiles with conda-managed dependencies (--use-conda))
- **AMBER 2.0.21-beta** (evaluation of binning predictions against ground truth in rank-based mode, yielding sensitivity, precision, and f1-score in *.rank.tsv outputs) — https://github.com/CAMI-challenge/AMBER
- **genome_updater.sh** (downloads and keeps up-to-date RefSeq genomic .fna reference sequences with assembly_summary.txt and taxdump) — https://raw.githubusercontent.com/pirovc/genome_updater/a301b927362aced47ed88a20b3f396df0bf5258d/genome_updater.sh
- **metabench** (benchmarking pipeline (Snakemake) that structures database builds, multi-parameter classification runs (e.g. --rel-cutoff grid), and standardized evaluation output) — https://github.com/pirovc/metabench

## Examples

```
ganon classify --db-prefix bac_cg_rs --output-prefix classify_results --paired-reads my_reads.1.fq.gz
```

## Evaluation signals

- Build artifacts prefix.check, prefix.log, and prefix.time are produced for the ganon2 index
- Classification output rows follow the read id / assembly id / taxid schema
- AMBER run produces *.rank.tsv files containing sensitivity, precision, and f1-score values
- The cutoff/filter sweep shows the expected sensitivity/precision trade-off across the threshold grid, and the selected configuration maximizes strain-level F1-score
- Build and classify Snakefiles complete under Snakemake >=5.2.0 with --use-conda without dependency failures

## Limitations

- ganon2 treats strain-level targets equally by its algorithm, but default cutoff and filter threshold values are not strain-optimal and must be adjusted for more sensitive or precise strain-level results
- Recovering the reference set depends on external resources (RefSeq FTP, genome_updater) whose URLs and snapshots may change; the benchmark pins specific commits (e.g. genome_updater a301b927362aced47ed88a20b3f396df0bf5258d)
- AMBER evaluation requires ground-truth readid/assembly/taxid files, so the approach is limited to simulated or otherwise ground-truthed datasets (e.g. CAMI strain-madness)

## Evidence

- [abstract] ganon2, a sequence classification method that performs taxonomic binning and profiling for metagenomics analysis: "We propose a generational leap in performance and usability with ganon2, a sequence classification method that performs taxonomic binning and profiling for metagenomics analysis"
- [methods] The pipeline is divided into build and classify steps.: "The pipeline is divided into build and classify steps."
- [readme] AMBER==2.0.21-beta: "AMBER==2.0.21-beta (https://github.com/CAMI-challenge/AMBER/archive/658a268bc86622e6f41304d38c1856b727c995ed.tar.gz)"
- [readme] --rel-cutoff grid in metabench: [evidence span withheld — anchor status: external_doc]
- [readme] reads subset for evaluation: "A sub-set of a random 1 million reads from the complete set used to evaluate the tools is provided in this repository under `files/reads/`"
