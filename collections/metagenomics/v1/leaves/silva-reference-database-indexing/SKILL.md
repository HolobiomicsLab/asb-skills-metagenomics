---
name: silva-reference-database-indexing
description: Use when setting up the RiboTaxa pipeline and sortmerna/emirge need their
  own indexed SILVA SSU 138.1 databases, or when a custom rRNA reference database
  must be indexed via indexDB_arguments.conf.
when_to_use_negative:
- When the pipeline steps consuming the indexes (SortMeRNA filtering, EMIRGE reconstruction)
  are not being run — e.g. classification-only workflows using the QIIME2 sklearn
  classifier with its own trained SILVA 138 reference sequences
- When the target sequences are LSU rRNA genes or markers outside SSU 16S/18S — the
  indexed reference here is SILVA SSU 138.1
- When you already have the RiboTaxa-provided indexed SILVA SSU 138.1 database; re-indexing
  is unnecessary unless you need a different reference database
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: SortMeRNA
  role: Consumes the sortmerna_indexed_DB to extract 16S/18S rRNA reads from trimmed
    metagenomic reads, yielding *_16S18Sreads.fastq for EMIRGE
  repo: https://github.com/oschakoory/RiboTaxa
- name: EMIRGE (Emirge_amplicon.py)
  role: Consumes the Bowtie-indexed SILVA database to reconstruct full-length SSU
    rRNA sequences from SortMeRNA-filtered reads, using USEARCH internally
  repo: https://github.com/oschakoory/RiboTaxa
- name: RiboTaxa
  role: Pipeline that ships indexDB_arguments.conf and pre-indexed SILVA SSU 138.1
    databases for its sortmerna and emirge steps
  repo: https://github.com/oschakoory/RiboTaxa
- name: USEARCH
  role: Dependency invoked by EMIRGE during reconstruction; must be installed in $PATH
    before indexing-based reconstruction runs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:509e8d7339e4c32756ec44b16d8ab690edd3f0cdd7f39b1b13d64c9b00a1dd79
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  repo_url: https://github.com/oschakoory/RiboTaxa
  tools:
  - BBTools
  - EMIRGE (Emirge_amplicon.py)
  - RiboTaxa
  - SortMeRNA
  - USEARCH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# silva-reference-database-indexing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds the tool-specific indexed forms of the SILVA SSU 138.1 database that the RiboTaxa pipeline requires: a SortMeRNA index for filtering 16S/18S reads and a Bowtie index used by Emirge_amplicon.py for SSU rRNA gene reconstruction.

## When to use

Use when setting up the RiboTaxa pipeline and sortmerna/emirge need their own indexed SILVA SSU 138.1 databases, or when a custom rRNA reference database must be indexed via indexDB_arguments.conf.

## When NOT to use

- When the pipeline steps consuming the indexes (SortMeRNA filtering, EMIRGE reconstruction) are not being run — e.g. classification-only workflows using the QIIME2 sklearn classifier with its own trained SILVA 138 reference sequences
- When the target sequences are LSU rRNA genes or markers outside SSU 16S/18S — the indexed reference here is SILVA SSU 138.1
- When you already have the RiboTaxa-provided indexed SILVA SSU 138.1 database; re-indexing is unnecessary unless you need a different reference database

## Inputs

- SILVA SSU 138.1 NR99 FASTA database (510,508 SSU rRNA gene sequences), or an alternative rRNA database such as RDP, Greengenes, UNITE or GTDB
- indexDB_arguments.conf configuration file for specifying indexing parameters, directories and input files

## Outputs

- sortmerna_indexed_DB: pre-indexed SILVA SSU 138.1 database for SortMeRNA read filtering
- Bowtie-indexed SILVA database consumed by Emirge_amplicon.py during SSU reconstruction
- Indexing log files (e.g. SortMeRNA .log reporting the percentage of SSU reads filtered)

## How to apply

Download the latest SILVA SSU 138.1 database from SILVA, or reuse the pre-indexed SILVA SSU 138.1 database distributed for RiboTaxa. If indexing your own database (e.g. RDP, Greengenes, UNITE), fill the config file indexDB_arguments.conf, keeping parameters as defined except for directories and input files, since uncertain parameters should be left at their defaults. Two indexes are required: the sortmerna_indexed_DB used by SortMeRNA to extract 16S/18S reads (producing *_16S18Sreads.fastq and a .log reporting the percentage of SSU reads filtered), and the Bowtie-indexed SILVA database used by Emirge_amplicon.py for SSU reconstruction (with JOIN_TRESHOLD/IDENTITY = 1 and 120 iterations). EMIRGE additionally relies on USEARCH being available in the $PATH.

## Related tools

- **SortMeRNA** (Consumes the sortmerna_indexed_DB to extract 16S/18S rRNA reads from trimmed metagenomic reads, yielding *_16S18Sreads.fastq for EMIRGE) — https://github.com/oschakoory/RiboTaxa
- **EMIRGE (Emirge_amplicon.py)** (Consumes the Bowtie-indexed SILVA database to reconstruct full-length SSU rRNA sequences from SortMeRNA-filtered reads, using USEARCH internally) — https://github.com/oschakoory/RiboTaxa
- **RiboTaxa** (Pipeline that ships indexDB_arguments.conf and pre-indexed SILVA SSU 138.1 databases for its sortmerna and emirge steps) — https://github.com/oschakoory/RiboTaxa
- **USEARCH** (Dependency invoked by EMIRGE during reconstruction; must be installed in $PATH before indexing-based reconstruction runs)

## Evaluation signals

- sortmerna_indexed_DB and the Bowtie-indexed SILVA databases exist and are found by the pipeline's conda environments (RiboTaxa_py27 for indexing-dependent steps)
- Running SortMeRNA against the index produces *_16S18Sreads.fastq files plus a .log reporting the percentage of SSU reads filtered for each input sample
- Emirge_amplicon.py runs to its 120 iterations against the Bowtie-indexed SILVA database without index-related errors, and USEARCH is reachable via `usearch --version`
- The reconstructed metagenome_SSU_sequences.fasta in the SSU_sequences output directory is valid FASTA containing full-length/nearly full-length 16S/18S sequences

## Limitations

- The indexes are reference-specific: SILVA SSU 138.1 indexes cannot be reused for other databases (RDP, Greengenes, UNITE, GTDB), which must be indexed separately via indexDB_arguments.conf
- Pre-built indexed SILVA SSU 138.1 databases are distributed via an external download link, so availability or version match with the SILVA release must be checked
- Certain indexing parameters may be unfamiliar; the README advises leaving them as defined except for directories and input files, so misuse of these defaults is a risk when indexing custom databases

## Evidence

- [methods] The latest database SILVA SSU 138.1 can be downloaded here and the indexed database SILVA SSU 138.1 for RiboTaxa are available: [evidence span withheld — anchor status: not_found]
