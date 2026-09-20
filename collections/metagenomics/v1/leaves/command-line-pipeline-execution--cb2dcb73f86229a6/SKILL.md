---
name: command-line-pipeline-execution
description: Use when raw metatranscriptome FASTQ files must be processed through
  preprocessing, annotation, and aggregation on a cluster with local databases, or
  when benchmarking DIAMOND resource allocation (RAM sweep at fixed CPUs).
when_to_use_negative:
- Input is already an aggregated annotation/abundance table rather than raw FASTQ
  reads — jump directly to the aggregation or R analysis scripts instead
- A web-service metatranscriptome analysis is desired (SAMSA2 v2 exists precisely
  to avoid MG-RAST dependence and supports only local, DIAMOND-indexed databases)
- Only Python 3 environments are available and aggregation scripts cannot be run,
  since SAMSA2 requires Python 2 for aggregation
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SAMSA2
  role: 'The pipeline itself: bash master scripts chaining preprocessing, DIAMOND
    annotation, and aggregation steps on a cluster'
  repo: https://github.com/transcript/samsa2
- name: DIAMOND
  role: Accelerated BLAST-like aligner (v0.8.3) used in the annotation step against
    local DIAMOND-indexed databases; benchmarked for RAM scaling
  repo: https://github.com/bbuchfink/diamond
- name: Trimmomatic
  role: Read cleaner removing low-quality sequences and adapter contamination in the
    preprocessing phase
- name: PEAR
  role: Merges paired-end FASTQ mate pairs into single extended-fragment files (preprocessing)
- name: SortMeRNA
  role: Removes ribosomal reads during preprocessing so they do not pollute the mRNA
    functional profile
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:1e536175325e9e9d99fa709d9206bc88b15d120148013f0abb84dd06a1f8b404
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/samsa2
  tools:
  - DIAMOND
  - PEAR
  - SAMSA2
  - SortMeRNA
  - Trimmomatic
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# command-line-pipeline-execution

## Summary

Executes the SAMSA2 standalone metatranscriptome pipeline end-to-end via bash master scripts on an HPC/SLURM cluster, chaining preprocessing (PEAR, Trimmomatic, SortMeRNA), DIAMOND annotation against local DIAMOND-indexed databases, and Python-based aggregation.

## When to use

Use when raw metatranscriptome FASTQ files must be processed through preprocessing, annotation, and aggregation on a cluster with local databases, or when benchmarking DIAMOND resource allocation (RAM sweep at fixed CPUs).

## When NOT to use

- Input is already an aggregated annotation/abundance table rather than raw FASTQ reads — jump directly to the aggregation or R analysis scripts instead
- A web-service metatranscriptome analysis is desired (SAMSA2 v2 exists precisely to avoid MG-RAST dependence and supports only local, DIAMOND-indexed databases)
- Only Python 3 environments are available and aggregation scripts cannot be run, since SAMSA2 requires Python 2 for aggregation

## Inputs

- raw paired-end or single-end FASTQ metatranscriptome read files
- local DIAMOND-indexed protein reference database (e.g. ~6 GB NCBI nr Bacteria; SEED Subsystems)
- adapter-contamination FASTA file for Trimmomatic
- user-edited master_script.bash configuration (read locations, program paths, scheduler settings)

## Outputs

- per-read DIAMOND annotation results (line-item 'ticker tape' output)
- ranked abundance summary tables from DIAMOND_analysis_counter.py per sample
- checkpoint file naming completed pipeline steps
- graphical outputs (PCA plots) and DESeq differential-abundance statistics from R

## How to apply

Clone the SAMSA2 repository and either install dependencies (DIAMOND v0.8.3, Trimmomatic, PEAR, SortMeRNA) from the provided binaries with setup_and_test/package_installation.bash. Download reference databases (RefSeq Bacteria for organism/function, SEED Subsystems for hierarchical functional ontology) with setup_and_test/full_database_download.bash. Edit the options block at the top of master_script.bash — which runs the first 3 of 4 pipeline steps (preprocessing, annotation, aggregation) — specifying all read and program locations; the script is formatted for SLURM but the scheduler wrapper can be removed. During annotation, DIAMOND runs by default in --fast mode against a local, DIAMOND-indexed database; use --sensitive instead if accuracy matters, accepting significantly higher memory. For resource benchmarking, hold thread count fixed, sweep RAM allocation stepwise against a fixed 50,000-sequence FASTA subset, and record wall-clock runtimes: speedup saturates once the ~6 GB NCBI nr Bacteria database fits fully in memory, with ~10 CPUs / 20 GB RAM as the minimum efficient configuration. A checkpoint file records completed steps so reruns skip already-finished CPU-intensive stages.

## Related tools

- **SAMSA2** (The pipeline itself: bash master scripts chaining preprocessing, DIAMOND annotation, and aggregation steps on a cluster) — https://github.com/transcript/samsa2
- **DIAMOND** (Accelerated BLAST-like aligner (v0.8.3) used in the annotation step against local DIAMOND-indexed databases; benchmarked for RAM scaling) — https://github.com/bbuchfink/diamond
- **Trimmomatic** (Read cleaner removing low-quality sequences and adapter contamination in the preprocessing phase)
- **PEAR** (Merges paired-end FASTQ mate pairs into single extended-fragment files (preprocessing))
- **SortMeRNA** (Removes ribosomal reads during preprocessing so they do not pollute the mRNA functional profile)

## Examples

```
git clone https://github.com/transcript/samsa2.git && bash setup_and_test/package_installation.bash && bash master_script.bash
```

## Evaluation signals

- checkpoint file is created and lists each completed step by name, and reruns of the master script skip those steps instead of recomputing them
- DIAMOND annotation completes in minutes rather than the months BLAST would require at 10 annotations/second for millions of sequences
- DIAMOND_analysis_counter.py produces a ranked abundance summary for every metatranscriptome input file
- In a RAM-sweep benchmark, the speedup-versus-RAM curve plateaus once the ~6 GB nr Bacteria database fits fully in memory (~10 CPUs / 20 GB RAM minimum)
- Each pipeline step emits a defined exit statement if the master script dies unexpectedly, allowing failure localization

## Limitations

- Aggregation scripts require Python 2; the pipeline works 'mostly' with Python 3 but errors may remain uncaught, and it has only been tested on Linux
- Switching DIAMOND from --fast to --sensitive mode (lines 162 and 199 of master_script) may significantly increase memory requirements
- The RAM-speedup benefit is capped: higher RAM allocation helps DIAMOND only until the reference database is fully in memory, so over-allocating yields no further gains
- The --num_alignments 0 flag was removed from the SortMeRNA ribosomal step because it caused slowdowns; rRNA alignment counts are intentionally discarded

## Evidence

- [abstract] SAMSA2 is an upgrade to the original Simple Annotation of Metatranscriptomes by Sequence Analysis (SAMSA) pipeline that has been redesigned for standalone use on a supercomputing cluster.: "SAMSA2 is an upgrade to the original Simple Annotation of Metatranscriptomes by Sequence Analysis (SAMSA) pipeline that has been redesigned for standalone use on a supercomputing cluster."
- [methods] Master script performs first 3 of 4 steps: "git clone https://github.com/transcript/samsa2.git"
- [methods] use DIAMOND to annotate your reads against a database of your choosing: "use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed)"
- [readme] README: SLURM formatting and checkpoint mechanism: "The script is formated to be run on a HPC using a SLURM job scheduler, but this can be easily changed / removed."
- [readme] README: fast vs sensitive mode memory note: [evidence span withheld — anchor status: external_doc]
- [discussion] Python 2.7 / Linux-only caveat: "Operating system(s): any supporting Python 2.7 (tested on Linux)"
