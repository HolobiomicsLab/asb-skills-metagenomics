---
name: reference-database-indexing
description: Use when constructing taxonomic classification indices from large reference
  collections (e.g. NCBI RefSeq .fna/.faa sets) or when comparing the build cost of
  indexing tools across tool/database/parameter combinations.
when_to_use_negative:
- The task is classification/profiling of reads against an already-built index, not
  constructing or benchmarking the index itself.
- Only a small reference set is needed and download/build scalability across full
  RefSeq subsets and multiple tools is not the goal.
- The reference sequences require assembly or quality processing before indexing (RefSeq-derived
  .fna/.faa files are assumed ready to index).
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: ganon2
  role: 'Primary indexer: builds compact HIBF-based indices from RefSeq subsets with
    ganon build (e.g. --max-fp 0.0001 as a ranged parameter); fastest to build the
    RefSeq RG index (20 min).'
  repo: https://github.com/pirovc/ganon
- name: kmcp
  role: Alternative indexer benchmarked for build time/memory/size; fastest for RefSeq
    CG (14 min) and full RefSeq (6 h 48 min).
- name: kraken2 + bracken
  role: Alternative indexer/classifier pair included in the build benchmark with default
    parameters.
- name: Metacache
  role: Alternative metagenomic classifier whose index build is benchmarked alongside
    the other tools.
- name: Snakemake
  role: Workflow engine driving the build pipeline; dry-run with -npr, then execute
    with --configfile and --use-conda.
- name: genome_updater.sh
  role: Downloads and keeps reference sequences up to date for building the databases.
  repo: https://raw.githubusercontent.com/pirovc/genome_updater/a301b927362aced47ed88a20b3f396df0bf5258d/genome_updater.sh
- name: conda
  role: Provides all tool dependencies for the Snakemake build workflow via --use-conda.
- name: diamond
  role: Consumes .faa protein reference files as part of the multi-tool database construction
    benchmark.
- name: wget
  role: Fetches archived RefSeq reference archives (e.g. all.fna.tar.gz) and pinned
    helper scripts.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaf094
    title: 'ganon2: up-to-date and scalable metagenomics analysis'
  source_content_hash: sha256:7075ae53f4d1d76dcfc90d0056d0af0f1ccd7531bc8f8e8a7aa0747a11a1354c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pirovc/ganon
  tools:
  - Metacache
  - Snakemake
  - conda
  - diamond
  - ganon2
  - genome_updater.sh
  - kmcp
  - kraken2 + bracken
  - wget
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# reference-database-indexing

## Summary

Builds and benchmarks reference sequence indices for metagenomic classifiers (ganon2, kmcp, kraken2+bracken, Metacache) from NCBI RefSeq subsets and full RefSeq, recording wall-clock build time, peak memory, and resulting index size so databases remain scalable as RefSeq grows ~20% per year.

## When to use

Use when constructing taxonomic classification indices from large reference collections (e.g. NCBI RefSeq .fna/.faa sets) or when comparing the build cost of indexing tools across tool/database/parameter combinations.

## When NOT to use

- The task is classification/profiling of reads against an already-built index, not constructing or benchmarking the index itself.
- Only a small reference set is needed and download/build scalability across full RefSeq subsets and multiple tools is not the goal.
- The reference sequences require assembly or quality processing before indexing (RefSeq-derived .fna/.faa files are assumed ready to index).

## Inputs

- Reference genome nucleotide files (.fna.gz) and protein files (.faa.gz) from NCBI RefSeq subsets
- NCBI taxdump files: nodes.dmp, names.dmp, merged.dmp
- assembly_summary.txt and acc_len_taxid_assembly.txt auxiliary mapping files
- Build configuration YAML (fasta/faa paths, taxonomy files, tools_path, parameters per tool/database)

## Outputs

- Tool-specific index files (e.g. ganon_db.hibf/.ibf/.tax, kmcp_db blocks with .uniki, name.map, taxid.map)
- prefix.log (build stdout/stderr) and prefix.time (/usr/bin/time -v resource usage)
- prefix.check (recorded essential-file sizes) and prefix.stats (tool, database, parameters, time elapsed, peak memory bytes, index size bytes)
- *.build.bench. / *.build.bench.tsv / *.build.size.tsv benchmark files per tool/version/database/parameter combination

## How to apply

Recover reference sequences with genome_updater.sh or wget (e.g. RefSeqCG .fna/.faa for bacteria, full RefSeq subsets) and unpack auxiliary files: *acc_len_taxid_assembly.txt.gz, assembly_summary.txt.gz, and the NCBI taxdump (nodes.dmp, names.dmp, merged.dmp). Configure a build YAML so fasta, faa, taxonomy dumps, acc_len_taxid_assembly and tools_path are set per tool/database combination, then dry-run Snakemake with -npr before executing the full build with --use-conda and default parameters. Wrap each tool's index build with '/usr/bin/time -v' so stdout/stderr land in prefix.log and resource usage in prefix.time under workdir/db_name/tool_db/db_config/. Record sizes of all essential index files into prefix.check and aggregate time elapsed, peak memory bytes, and index size bytes into prefix.stats; in MetaBench, per-tool build outputs are *.build.bench., *.build.bench.tsv, *.build.log and *.build.size.tsv (du --bytes of mandatory database files). ganon2 indices built this way are on average 50% smaller than state-of-the-art methods on full NCBI RefSeq and its subsets.

## Related tools

- **ganon2** (Primary indexer: builds compact HIBF-based indices from RefSeq subsets with ganon build (e.g. --max-fp 0.0001 as a ranged parameter); fastest to build the RefSeq RG index (20 min).) — https://github.com/pirovc/ganon
- **kmcp** (Alternative indexer benchmarked for build time/memory/size; fastest for RefSeq CG (14 min) and full RefSeq (6 h 48 min).)
- **kraken2 + bracken** (Alternative indexer/classifier pair included in the build benchmark with default parameters.)
- **Metacache** (Alternative metagenomic classifier whose index build is benchmarked alongside the other tools.)
- **Snakemake** (Workflow engine driving the build pipeline; dry-run with -npr, then execute with --configfile and --use-conda.)
- **genome_updater.sh** (Downloads and keeps reference sequences up to date for building the databases.) — https://raw.githubusercontent.com/pirovc/genome_updater/a301b927362aced47ed88a20b3f396df0bf5258d/genome_updater.sh
- **conda** (Provides all tool dependencies for the Snakemake build workflow via --use-conda.)
- **diamond** (Consumes .faa protein reference files as part of the multi-tool database construction benchmark.)
- **wget** (Fetches archived RefSeq reference archives (e.g. all.fna.tar.gz) and pinned helper scripts.)

## Examples

```
ganon build --db-prefix bac_cg_rs --source refseq --organism-group bacteria --complete-genomes --threads 24 --download-threads 12
```

## Evaluation signals

- Each tool-database-config folder contains a prefix.check whose recorded sizes match the on-disk essential index files.
- A prefix.stats row exists with non-zero time-elapsed, peak-memory and index-size fields for every tool/database combination.
- Snakemake --dry-run (-npr) completes cleanly and validates all build commands before execution.
- MetaBench emits *.build.bench. (standardized metrics), *.build.log (full stdout/stderr), and *.build.size.tsv (du --bytes of mandatory database files) for each combination.
- ganon2 index sizes should come out on average ~50% smaller than state-of-the-art methods on the full NCBI RefSeq and its subsets.

## Limitations

- Build times and memory are highly database-dependent: kmcp was fastest for RefSeq CG (14 min) and full RefSeq (6 h 48 min) while ganon2 was fastest for RefSeq RG (20 min) — no single tool wins everywhere.
- ganon2 achieved its fast build times only with intermediate memory usage, so memory-constrained hosts may need care.
- The old RefSeq FTP archive (ftp://ftp.ncbi.nlm.nih.gov/genomes/archive/old_refseq/) and specific genome_updater commits are pinned; later RefSeq layout changes may break the wget-based download step.
- Any information in fixed_args: is not accounted for in database folder/file names; only args: parameters (e.g. --max-fp) name the output database folders.
- Computational metagenomics faces challenges from data complexity and size; reference data are not equally distributed across the tree of life, and outdated or incomplete references cause methods to underperform.

## Evidence

- [abstract] ganon2 indices are on average 50% smaller than state-of-the-art methods based on the full NCBI RefSeq and its subsets: "Based on the full NCBI RefSeq and its subsets, ganon2 indices are on average 50% smaller than state-of-the-art methods."
- [readme] ganon build command from README: [evidence span withheld — anchor status: not_found]
- [readme] aux file unpacking step: "gzip -d files/*refseq*/*acc_len_taxid_assembly.txt.gz"
- [readme] Snakemake build invocation: [evidence span withheld — anchor status: external_doc]
- [readme] index size TSV definition: [evidence span withheld — anchor status: external_doc]
