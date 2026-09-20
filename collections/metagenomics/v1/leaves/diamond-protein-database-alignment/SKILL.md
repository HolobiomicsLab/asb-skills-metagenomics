---
name: diamond-protein-database-alignment
description: Use when raw FASTQ metatranscriptome reads need organism or functional
  annotation against a local protein database, and speed or reproducibility rules
  out BLAST or web services such as MG-RAST.
when_to_use_negative:
- Input is already an aggregated abundance/annotation summary table — the alignment
  step operates on per-read FASTQ data
- The analysis requires rRNA or 16S-based community profiling — SortMeRNA-filtered
  reads exclude ribosomal sequences, which DIAMOND protein alignment is not designed
  to annotate
- Only a web service with no local database control is acceptable — this skill requires
  local, DIAMOND-indexed databases and Python 2.7 for aggregation
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: DIAMOND
  role: Accelerated BLAST-like sequence aligner used to annotate reads against the
    local protein database; version 0.8.3 used by SAMSA2
  repo: https://github.com/bbuchfink/diamond
- name: SAMSA2
  role: Metatranscriptome pipeline wrapping the DIAMOND annotation step; provides
    master_script.bash, DIAMOND_analysis_counter.py, and database download scripts
  repo: https://github.com/transcript/samsa2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:95ebb6cf0a258f93975c2d0d2f360857706b135ad87a627a279ceca737045634
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bbuchfink/diamond
  tools:
  - DIAMOND
  - SAMSA2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# diamond-protein-database-alignment

## Summary

Annotate metatranscriptome reads by translating and aligning them to a local, DIAMOND-indexed protein reference database (e.g. RefSeq for organisms, SEED Subsystems for functions) using the DIAMOND aligner, as implemented in the SAMSA2 pipeline. This replaces BLAST and web services like MG-RAST, yielding per-read annotations orders of magnitude faster and reproducibly on a cluster.

## When to use

Use when raw FASTQ metatranscriptome reads need organism or functional annotation against a local protein database, and speed or reproducibility rules out BLAST or web services such as MG-RAST.

## When NOT to use

- Input is already an aggregated abundance/annotation summary table — the alignment step operates on per-read FASTQ data
- The analysis requires rRNA or 16S-based community profiling — SortMeRNA-filtered reads exclude ribosomal sequences, which DIAMOND protein alignment is not designed to annotate
- Only a web service with no local database control is acceptable — this skill requires local, DIAMOND-indexed databases and Python 2.7 for aggregation

## Inputs

- mRNA (non-ribosomal) reads in FASTQ format, post PEAR merging, Trimmomatic trimming, and SortMeRNA rRNA filtering
- Local, DIAMOND-indexed protein reference database (e.g. NCBI RefSeq bacteria, SEED Subsystems, or a custom indexed database)

## Outputs

- Per-read DIAMOND annotation results (BLAST-like tabular output)
- Ranked abundance summary tables per metatranscriptome file produced by DIAMOND_analysis_counter.py

## How to apply

First make sure the chosen reference database (e.g. NCBI RefSeq bacteria for organism/specific-function results, or SEED Subsystems for hierarchical functional ontology) is local and DIAMOND-indexed; SAMSA2's full_database_download.bash fetches both. Run DIAMOND (version 0.8.3 was used in the SAMSA2 paper) against the indexed database, by default in '--fast' mode for speed; if accuracy is paramount, switch to '--sensitive' (lines 162 and 199 of master_script.bash), accepting significantly higher memory use. DIAMOND emits per-read 'ticker tape' results, which are then condensed with DIAMOND_analysis_counter.py into ranked abundance summaries per metatranscriptome file. The rationale, per the paper, is that a BLAST approach at ~10 annotations/second would take months per file, whereas DIAMOND aligns at 100x-10,000x BLAST speed, enabling millions of sequences per sample on a supercomputing cluster.

## Related tools

- **DIAMOND** (Accelerated BLAST-like sequence aligner used to annotate reads against the local protein database; version 0.8.3 used by SAMSA2) — https://github.com/bbuchfink/diamond
- **SAMSA2** (Metatranscriptome pipeline wrapping the DIAMOND annotation step; provides master_script.bash, DIAMOND_analysis_counter.py, and database download scripts) — https://github.com/transcript/samsa2

## Examples

```
bash example_DIAMOND_annotation_script.bash  # then: python DIAMOND_analysis_counter.py <diamond_results> <ranked_output.tsv>
```

## Evaluation signals

- Every input read receives an annotation line from DIAMOND; reads without database hits are accounted for in the aggregation summary
- On MetaSim-simulated gut microbiome metatranscriptomes, genus-level organism accuracy approximates ~95% (vs 91% for SAMSA 1.0), and hierarchy-level functional accuracy exceeds 95%
- Aggregated output via DIAMOND_analysis_counter.py is a ranked abundance summary (per-read 'line item receipt' condensed to summary tables)
- Annotation runtime is feasible on millions of sequences per sample — a BLAST run at ~10 annotations/second taking months indicates the wrong aligner mode was used

## Limitations

- DIAMOND defaults to '--fast' mode which prioritizes speed over accuracy; '--sensitive' mode significantly increases memory requirements
- The database must be local and DIAMOND-indexed; SAMSA2 as published works mostly with Python 2.7 and was only tested on Linux
- Version pinning matters for reproducibility: SAMSA2 used DIAMOND version 0.8.3

## Evidence

- [readme] use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMONDEX-indexed): "use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed)"
- [readme] Note: DIAMOND is set by default to run in fast mode, where it prioritizes speed.: [evidence span withheld — anchor status: external_doc]
- [readme] Annotation is completed using DIAMOND, an accelerated BLAST-like sequence aligner.: "Annotation is completed using [DIAMOND, an accelerated BLAST-like sequence aligner.](https://github.com/bbuchfink/diamond)"
- [abstract] SAMSA2 is faster due to the use of the DIAMOND aligner, and more flexible and reproducible because it uses local databases.: "SAMSA2 is faster due to the use of the DIAMOND aligner, and more flexible and reproducible because it uses local databases."
- [intro] rely on BLAST [9] for annotations, which may be unacceptably slow when processing multiple millions of sequences per sample: "rely on BLAST [9] for annotations, which may be unacceptably slow when processing multiple millions of sequences per sample"
