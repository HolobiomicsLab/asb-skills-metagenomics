---
name: reference-database-indexing
description: Use when preparing a local reference protein database (RefSeq, SEED Subsystems,
  or a custom FASTA) for DIAMOND-based read annotation in SAMSA2, with no external
  web service.
when_to_use_negative:
- The goal is web-service-based annotation or a pipeline that does not require a local,
  DIAMOND-indexed reference (e.g., MG-RAST-dependent SAMSA 1.0)
- Input is already a DIAMOND .dmnd index or a pre-computed per-read annotation table
  rather than a FASTA database
- The analysis targets 16S rRNA amplicon data, which the paper notes lacks resolution
  for closely related genera/species and provides no activity information
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0610
tools:
- name: DIAMOND
  role: Accelerated BLAST-like aligner; its makedb step indexes the local protein
    FASTA so reads can be annotated offline (version 0.8.3 as distributed with SAMSA2)
  repo: https://github.com/bbuchfink/diamond
- name: SAMSA2
  role: Pipeline that consumes the DIAMOND-indexed database; provides full_database_download.bash
    and example_DIAMOND_annotation_script.bash for downloading databases and running
    the annotation step
  repo: https://github.com/transcript/samsa2
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:81c217378b42204f14131aa1addad2f0fc206dfa768ba8e94cb4258286da3446
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bbuchfink/diamond
  tools:
  - DIAMOND
  - SAMSA2
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# reference-database-indexing

## Summary

Builds a local, DIAMOND-indexed protein reference database (e.g., NCBI RefSeq Bacteria or a custom FASTA whose headers encode sequence ID, function, and organism name) so that SAMSA2 can annotate metatranscriptome reads offline with the DIAMOND aligner.

## When to use

Use when preparing a local reference protein database (RefSeq, SEED Subsystems, or a custom FASTA) for DIAMOND-based read annotation in SAMSA2, with no external web service.

## When NOT to use

- The goal is web-service-based annotation or a pipeline that does not require a local, DIAMOND-indexed reference (e.g., MG-RAST-dependent SAMSA 1.0)
- Input is already a DIAMOND .dmnd index or a pre-computed per-read annotation table rather than a FASTA database
- The analysis targets 16S rRNA amplicon data, which the paper notes lacks resolution for closely related genera/species and provides no activity information

## Inputs

- Reference protein database in FASTA format (e.g., NCBI RefSeq Bacteria or SEED Subsystems downloaded via full_database_download.bash)
- Custom protein FASTA with headers encoding sequence ID, function name, and organism name

## Outputs

- DIAMOND-indexed database file (.dmnd) usable locally by DIAMOND blastx
- Per-read plaintext annotation outfile (samsa2_annotation_output.txt) with read ID, organism, and function columns

## How to apply

Obtain the reference protein database in FASTA format: either download the full RefSeq Bacteria and SEED Subsystems databases via SAMSA2's full_database_download.bash script, or construct a custom FASTA whose headers encode an internal sequence ID, function name, and organism name. Index the FASTA with DIAMOND's makedb step, producing a .dmnd file that DIAMOND version 0.8.3 (as distributed with SAMSA2) can query locally. The database must remain local and DIAMOND-indexed — this is what makes SAMSA2 flexible and reproducible compared with MG-RAST-dependent workflows. After indexing, run DIAMOND blastx from SAMSA2's example_DIAMOND_annotation_script.bash against the indexed database; DIAMOND defaults to --fast mode for speed, switchable to --sensitive at the cost of higher memory. Databases can be upgraded, altered, or customized without reinstalling the pipeline, so re-index when a newer database version is released.

## Related tools

- **DIAMOND** (Accelerated BLAST-like aligner; its makedb step indexes the local protein FASTA so reads can be annotated offline (version 0.8.3 as distributed with SAMSA2)) — https://github.com/bbuchfink/diamond
- **SAMSA2** (Pipeline that consumes the DIAMOND-indexed database; provides full_database_download.bash and example_DIAMOND_annotation_script.bash for downloading databases and running the annotation step) — https://github.com/transcript/samsa2

## Examples

```
bash setup_and_test/full_database_download.bash  # then: diamond makedb --in refseq_bacteria.fasta -d refseq_bacteria
```

## Evaluation signals

- The DIAMOND makedb command completes and produces a non-empty .dmnd index file from the input FASTA
- A test DIAMOND blastx run against the index returns tabular hits whose headers resolve to non-empty organism and function fields
- The annotation outfile (samsa2_annotation_output.txt) is plaintext with one annotation record per input read and no empty organism/function columns
- No network calls to external services (e.g., MG-RAST) occur during annotation — the database is strictly local
- Custom database headers correctly yield the intended organism/function assignments when parsed by DIAMOND_analysis_counter.py

## Limitations

- SAMSA2 requires Python 2.7 for aggregation scripts and was only tested on Linux despite claiming any OS
- Running DIAMOND in --sensitive mode may significantly increase memory requirements relative to the default --fast mode
- Datasets/reference databases are described only as available in the SAMSA2 repository with no accession IDs, complicating provenance tracking

## Evidence

- [methods] Database must be local and DIAMOND-indexed: "use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed)"
- [abstract] DIAMOND aligner enables local databases: "SAMSA2 is faster due to the use of the DIAMOND aligner, and more flexible and reproducible because it uses local databases."
- [readme] Full database download in SAMSA2 setup: [evidence span withheld — anchor status: external_doc]
- [readme] DIAMOND fast mode default: [evidence span withheld — anchor status: external_doc]
