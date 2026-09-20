---
name: taxonomic-abundance-profiling
description: Use when raw trimmed .fq.gz metagenomic reads must be binned and profiled
  at a target taxonomic rank against a large reference database, e.g. reproducing
  genus-level abundance trajectories across a sample time series.
when_to_use_negative:
- When genomes must be assembled or strain-resolved rather than reads classified against
  references — this skill classifies short reads directly without assembly
- When input is already a bioboxes-format profiling output or abundance table rather
  than raw FASTQ reads
- When the reference database must be tiny/in-memory only — ganon2 targets large,
  up-to-date reference sets and its profiling depends on reference completeness
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0637
tools:
- name: ganon2
  role: Classifies reads against the reference index and performs taxonomic binning
    and metagenomics profiling at the genus level
  repo: https://github.com/pirovc/ganon
- name: Snakemake >=5.2.0
  role: Orchestrates the build and classify pipeline steps with conda dependencies
    and multi-core execution
- name: AMBER 2.0.21-beta
  role: Evaluates taxonomic binning/profiling against ground truth (sens, prec, f1s
    metrics in *.rank.tsv)
  repo: https://github.com/CAMI-challenge/AMBER
- name: genome_updater.sh
  role: Downloads and keeps reference sequences current for index building
  repo: https://github.com/pirovc/genome_updater
- name: conda
  role: Covers all pipeline dependencies during Snakemake --use-conda execution
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqaf094
    title: 'ganon2: up-to-date and scalable metagenomics analysis'
  source_content_hash: sha256:9f5e75438f25f5cd9a69b1ffbc52dfb2bf710065173389885efed9f3e2b1921f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pirovc/ganon
  tools:
  - AMBER 2.0.21-beta
  - Snakemake >=5.2.0
  - conda
  - ganon2
  - genome_updater.sh
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# taxonomic-abundance-profiling

## Summary

Generates genus-level (or other rank) relative abundance profiles from metagenomic short reads by classifying them directly against a large reference set (e.g. GTDB R220, 596,859 genomes) with ganon2, without genome assembly, and aggregating per-sample profiles into a single abundance table.

## When to use

Use when raw trimmed .fq.gz metagenomic reads must be binned and profiled at a target taxonomic rank against a large reference database, e.g. reproducing genus-level abundance trajectories across a sample time series.

## When NOT to use

- When genomes must be assembled or strain-resolved rather than reads classified against references — this skill classifies short reads directly without assembly
- When input is already a bioboxes-format profiling output or abundance table rather than raw FASTQ reads
- When the reference database must be tiny/in-memory only — ganon2 targets large, up-to-date reference sets and its profiling depends on reference completeness

## Inputs

- Trimmed paired-end short reads in .fq.gz/.fastq.gz format (fq1/fq2)
- Reference genome fasta sequences (e.g. full GTDB R220, 596,859 genomes, or NCBI RefSeq)
- Taxonomy dump files: nodes.dmp, names.dmp, merged.dmp
- Auxiliary files: acc_len_taxid_assembly file and assembly_summary.txt
- Ground-truth mapping file (gt): accession.version, assembly id, taxonomic id per line

## Outputs

- Per-sample ganon2 rank-based profiling tables (*.rank.tsv with sens, prec, f1s fields)
- Aggregated genus-level abundance table, e.g. ganon2_sewage_genus_profiles.tsv (rows = samples x genera)
- Bloom time-point table, e.g. pseudomonas_bloom_analysis.tsv (taxon/time-point/relative-abundance/bloom-flag layout)
- Abundance trajectory plot, e.g. pseudomonas_bloom_comparison.png analogous to plot_rank.png outputs

## How to apply

Build a ganon2 index over the full reference set (build/Snakefile Snakemake workflow) supplying the fasta sequences, nodes.dmp/names.dmp/merged.dmp taxonomy files, and the acc_len_taxid_assembly auxiliary file. Configure classification with a YAML file pointing fq1 at each sample's reads and a gt file mapping 'accession.version <tab> assembly id <tab> taxonomic id'. Run ganon2 classification and metagenomics profiling per sample under Snakemake (--use-conda, multi-core), producing rank-based outputs (*.rank.tsv with gt, db, class, tp, fp, sens, prec, f1s fields). Aggregate per-sample genus-level relative abundances into one table (rows = samples x genera, columns = abundance values), order samples by time point for longitudinal comparisons, and flag taxa as bloom-level where relative abundance substantially exceeds their median across the series. Validate that abundances lie in [0,1] and each sample contributes a genus-level row.

## Related tools

- **ganon2** (Classifies reads against the reference index and performs taxonomic binning and metagenomics profiling at the genus level) — https://github.com/pirovc/ganon
- **Snakemake >=5.2.0** (Orchestrates the build and classify pipeline steps with conda dependencies and multi-core execution)
- **AMBER 2.0.21-beta** (Evaluates taxonomic binning/profiling against ground truth (sens, prec, f1s metrics in *.rank.tsv)) — https://github.com/CAMI-challenge/AMBER
- **genome_updater.sh** (Downloads and keeps reference sequences current for index building) — https://github.com/pirovc/genome_updater
- **conda** (Covers all pipeline dependencies during Snakemake --use-conda execution)

## Examples

```
ganon classify --db-prefix bac_cg_rs --output-prefix classify_results --paired-reads my_reads.1.fq.gz
```

## Evaluation signals

- Each per-sample run produces the expected rank-based output (*.rank.tsv containing sens, prec, f1s fields)
- Every sample (e.g. all 28 Bologna sewage samples) contributes a genus-level abundance row to the final aggregated table
- All relative abundances in the aggregated table lie in [0,1]
- Bloom-flagged time points (relative abundance substantially exceeding the taxon's median across the series) are visually consistent with the reference profile, e.g. fig. 1B of the original study
- Genus-level abundances are comparable to the original study's profile, e.g. Pseudomonas_E blooms correctly detected

## Limitations

- The use of an outdated version of references or the inability to use all currently available references leads most methods, in the best case, to underperform
- Results depend on reference data distribution; reference data are not equally distributed in the tree of life, with growth biased toward human-related bacteria and model organisms
- ganon2 targets minutes-to-hours per sample with limited memory, but very large reference sets (hundreds of thousands of genomes) still require the Snakemake build step with taxdump and auxiliary files

## Evidence

- [abstract] ganon2 achieves genus-level profiling without assembly: "a sequence classification method that performs taxonomic binning and profiling for metagenomics analysis"
- [intro] ganon2 scales to large reference sets with low memory: "we propose ganon2, a sequence classification and metagenomics profiling tool that is able to classify large amounts of sequences against large amounts of references"
- [readme] Pipeline is divided into build and classify steps per the benchmarking workflow: "The pipeline is divided into build and classify steps."
- [abstract] Profiling quality metrics from abundance estimation: "In profiling, improvements in the F1-score median are up to 0.35, keeping a balanced L1-norm error in the abundance estimation"
