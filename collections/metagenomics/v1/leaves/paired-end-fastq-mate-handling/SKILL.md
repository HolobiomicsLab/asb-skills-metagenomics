---
name: paired-end-fastq-mate-handling
description: Use when mapping paired-end shotgun metagenomic FASTQ reads to a protein-reference
  gene database (e.g., UniRef90) and you need per-mate alignment before downstream
  -hit filtering and gene counting.
when_to_use_negative:
- Input is raw unfiltered FASTQ — MetaLAFFA expects reads to have already passed host
  read filtering, duplicate read filtering, and quality trimming/filtering
- Input reads are single-end only with no mate structure — mate-separate handling
  is unnecessary overhead
- Nucleotide-level read mapping to a host genome (e.g., hs37d5 host filtering) — that
  step uses Bowtie 2, not DIAMOND translated alignment
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0623
tools:
- name: DIAMOND
  role: Aligner run in translated-alignment mode to map each FASTQ mate separately
    against the UniRef90 protein database, producing m8 tabular output
  repo: https://github.com/bbuchfink/diamond_docs
- name: MetaLAFFA
  role: Snakemake pipeline that orchestrates per-mate FASTQ processing through the
    read-mapping step before downstream concatenation, best-hit filtering, and gene
    counting
  repo: https://github.com/borenstein-lab/MetaLAFFA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:1f408689568fe64ad1e244dcce8d197ef92b31016b8274ee97f4b05971f79dde
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bbuchfink/diamond_docs
  tools:
  - DIAMOND
  - MetaLAFFA
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# paired-end-fastq-mate-handling

## Summary

In MetaLAFFA's read mapping phase, paired-end shotgun metagenomic reads are mapped mate-by-mate (R1, R2, and singletons separately) with DIAMOND in translated-search mode against the UniRef90 protein database, avoiding over-penalization of read pairs where only one mate aligns to a gene.

## When to use

Use when mapping paired-end shotgun metagenomic FASTQ reads to a protein-reference gene database (e.g., UniRef90) and you need per-mate alignment before downstream best-hit filtering and gene counting.

## When NOT to use

- Input is raw unfiltered FASTQ — MetaLAFFA expects reads to have already passed host read filtering, duplicate read filtering, and quality trimming/filtering
- Input reads are single-end only with no mate structure — mate-separate handling is unnecessary overhead
- Nucleotide-level read mapping to a host genome (e.g., hs37d5 host filtering) — that step uses Bowtie 2, not DIAMOND translated alignment

## Inputs

- Quality-filtered, gzipped FASTQ files per sample (<sample_id>.<type_id>.fastq.gz) for forward, reverse, and singleton mates
- Prepared UniRef90 DIAMOND database (<name_of_database>.dmnd)

## Outputs

- DIAMOND tabular (m8-style) alignment hit table per input FASTQ, containing read-to-gene matches for downstream best-hit filtering

## How to apply

Load the quality-filtered, gzipped FASTQ files per sample following MetaLAFFA's naming convention (<sample_id>.<type_id>.fastq.gz) and align each mate type (forward, reverse, singleton) independently through the mapping step, as MetaLAFFA concatenates results only downstream. Run DIAMOND in translated-alignment mode against the prepared UniRef90 DIAMOND database (<name_of_database>.dmnd) so reads are 6-frame translated against protein sequences. Use MetaLAFFA's defaults --block_size 36 and --index_chunks 1 (these control how much of the reference is loaded at a time; reduce them if memory is limited), requesting roughly 22 cores and ~220G RAM per DIAMOND process. Write the DIAMOND tabular (m8-style) hit table for each input FASTQ, retaining read-to-gene matches for the subsequent best e-value hit filtering step.

## Related tools

- **DIAMOND** (Aligner run in translated-alignment mode to map each FASTQ mate separately against the UniRef90 protein database, producing m8 tabular output) — https://github.com/bbuchfink/diamond_docs
- **MetaLAFFA** (Snakemake pipeline that orchestrates per-mate FASTQ processing through the read-mapping step before downstream concatenation, best-hit filtering, and gene counting) — https://github.com/borenstein-lab/MetaLAFFA

## Examples

```
prepare_databases.py -u && diamond blastx -d UniRef90.dmnd -q <sample_id>.1.fastq.gz -o <sample_id>.1.hits.m8 --block-size 36 --index-chunks 1 -p 22
```

## Evaluation signals

- A hits output file exists for every input FASTQ (R1, R2, and singleton) and is parseable as a tab-separated DIAMOND tabulation
- Mapped-read fraction falls within the paper's reported 53%–78% range of reads matching UniRef90 across stool samples
- Each mate type was processed independently through the mapping step, with concatenation deferred to downstream steps
- DIAMOND resource settings match MetaLAFFA defaults (--block_size 36, --index_chunks 1) or were explicitly reduced for memory-limited environments

## Limitations

- Mapping mates separately means paired-end pairing information is not used to penalize/validate alignments, by design, to avoid over-penalizing pairs where only one mate aligns
- DIAMOND memory footprint is large (~220G RAM with default block_size 36 and index_chunks 1); block_size/index_chunks must be reduced on memory-limited systems
- UniRef90 database preparation is resource-intensive: several hours, ~40GB RAM, and ~150–300GB disk space depending on intermediate-file cleanup

## Evidence

- [methods] paired-end reads are mapped separately (as in HUMAnN2 and other pipelines) to avoid overly penalizing read pairs where only one mate aligns to a gene: [evidence span withheld — anchor status: not_found]
- [intro] aligning reads to the UniRef90 database of protein sequences using DIAMOND: "aligning reads to the UniRef90 database of protein sequences using DIAMOND"
- [intro] MetaLAFFA scans these matches to identify the best match (or best matches if there are ties) for each read in the database: "MetaLAFFA scans these matches to identify the best match (or best matches if there are ties) for each read in the database"
- [results] The percentage of reads that had a match in the UniRef90 [21] database varied from 53% to 78% across samples: "The percentage of reads that had a match in the UniRef90 [21] database varied from 53% to 78% across samples"
- [readme] Download and prepare the UniRef90 database for read mapping and functional annotation.: "`-u`: Download and prepare the UniRef90 database for read mapping and functional annotation."
