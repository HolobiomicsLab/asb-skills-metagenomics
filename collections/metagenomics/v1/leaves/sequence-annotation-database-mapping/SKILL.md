---
name: sequence-annotation-database-mapping
description: Use when raw metatranscriptome reads (e.g. FASTQ) must be assigned organism
  or functional annotations by aligning them to a local protein database, and speed
  or reproducibility rules out BLAST and web services like MG-RAST.
when_to_use_negative:
- Reads are unfiltered ribosomal sequences — SortMeRNA filtering must come first since
  rRNA alignments are discarded regardless
- High accuracy is required and compute/memory budgets do not allow DIAMOND --sensitive
  mode's significantly increased memory requirements
- Analysis goal is de novo assembly or genome annotation rather than direct read-to-protein-database
  mapping
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: DIAMOND
  role: Accelerated BLAST-like sequence aligner that annotates reads against the local
    protein database (version 0.8.3)
  repo: https://github.com/bbuchfink/diamond
- name: SAMSA2
  role: Pipeline wrapper whose master_script.bash invokes DIAMOND annotation between
    preprocessing and aggregation steps
  repo: https://github.com/transcript/samsa2
- name: SortMeRNA
  role: Removes ribosomal reads prior to annotation to reduce mapping load and increase
    speed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:e27c094fedf935813171a1306cd5f74557ede013f161d2c12d0ac0f602e9a62a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bbuchfink/diamond
  tools:
  - DIAMOND
  - MG-RAST
  - PEAR
  - SAMSA
  - SAMSA2
  - SortMeRNA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# sequence-annotation-database-mapping

## Summary

Annotation of metatranscriptomic reads by direct alignment against local, DIAMOND-indexed protein reference databases (NCBI RefSeq, SEED Subsystems, or custom databases such as CAZy) using DIAMOND within the SAMSA2 pipeline, replacing slow BLAST or web-service (MG-RAST) annotation.

## When to use

Use when raw metatranscriptome reads (e.g. FASTQ) must be assigned organism or functional annotations by aligning them to a local protein database, and speed or reproducibility rules out BLAST and web services like MG-RAST.

## When NOT to use

- Reads are unfiltered ribosomal sequences — SortMeRNA filtering must come first since rRNA alignments are discarded regardless
- High accuracy is required and compute/memory budgets do not allow DIAMOND --sensitive mode's significantly increased memory requirements
- Analysis goal is de novo assembly or genome annotation rather than direct read-to-protein-database mapping

## Inputs

- Preprocessed mRNA reads in FASTQ (or FASTA) format from paired-end metatranscriptomes
- Local DIAMOND-indexed protein reference database (e.g. RefSeq bacteria, SEED Subsystems, CAZy, or custom databases installed via full_database_download.bash)

## Outputs

- Per-read DIAMOND alignment results mapping each sequence to database entries
- Ranked abundance summary tables produced by DIAMOND_analysis_counter.py, importable into R for DESeq differential-abundance statistics

## How to apply

First preprocess reads (Trimmomatic cleaning, PEAR paired-end merging, SortMeRNA rRNA filtering) so only mRNA remains, since ribosomal reads do not contribute to the functional profile. Then run DIAMOND (version 0.8.3, default --fast mode; switch to --sensitive on lines 162/199 of master_script.bash if accuracy matters more than speed) against a local, DIAMOND-indexed protein database of your choosing, such as the RefSeq bacteria database for organism/specific function identification, SEED Subsystems for hierarchical functional ontology, or a custom indexed database. Aggregate the per-read DIAMOND 'ticker-tape' output with DIAMOND_analysis_counter.py to create ranked abundance summary tables. Rationale: DIAMOND is 100x-10,000x faster than BLAST, making annotation of millions of reads tractable offline, and local databases give full parameter control for reproducibility. In benchmarking, database mapping (RefSeq, SEED Subsystems) is the most time-consuming SAMSA2 step, and CAZy-only mapping of a 50,000-sequence metatranscriptome completes in under 30 minutes.

## Related tools

- **DIAMOND** (Accelerated BLAST-like sequence aligner that annotates reads against the local protein database (version 0.8.3)) — https://github.com/bbuchfink/diamond
- **SAMSA2** (Pipeline wrapper whose master_script.bash invokes DIAMOND annotation between preprocessing and aggregation steps) — https://github.com/transcript/samsa2
- **SortMeRNA** (Removes ribosomal reads prior to annotation to reduce mapping load and increase speed)

## Examples

```
git clone https://github.com/transcript/samsa2.git; bash example_DIAMOND_annotation_script.bash
```

## Evaluation signals

- Every input metatranscriptome file yields a ranked abundance summary table from DIAMOND_analysis_counter.py with database feature counts
- The reference database used was DIAMOND-indexed locally (e.g. RefSeq bacteria or SEED Subsystems downloaded via full_database_download.bash), not a remote web service
- Benchmark sanity check: for a 50,000-sequence sample, CAZy-only mapping completes in under 30 minutes; full RefSeq + SEED + CAZy annotation averages 7 h 41 min across five samples
- DIAMOND was run in its default --fast mode (or a deliberate --sensitive switch with accepted higher memory use), verified in the master script's DIAMOND command

## Limitations

- DIAMOND --fast mode prioritizes speed; --sensitive mode may significantly increase memory requirements
- SAMSA2 requires Python 2.7 for aggregation scripts and was only tested on Linux
- Databases must be local and DIAMOND-indexed before annotation; no support for remote lookup
- SAMSA 1.0 baseline via MG-RAST reports longer runtimes when permission is not given to make the data public immediately

## Evidence

- [abstract] DIAMOND aligner enables fast local-database annotation: "SAMSA2 is faster due to the use of the DIAMOND aligner, and more flexible and reproducible because it uses local databases."
- [methods] Annotation uses DIAMOND against a local indexed database: "use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed)"
- [methods] Aggregation script converts per-read DIAMOND output into ranked summaries: "use "DIAMOND_analysis_counter.py" to create a ranked abundance summary of the DIAMOND results from each metatransciptome file"
- [readme] Readme rationale for DIAMOND over BLAST: "Why DIAMOND?  At a standard rate of 10 annotations per second, a standard BLAST approach would take several months to finish - just for a single file!"
