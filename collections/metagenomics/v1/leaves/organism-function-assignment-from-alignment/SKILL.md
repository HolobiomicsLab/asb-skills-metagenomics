---
name: organism-function-assignment-from-alignment
description: Use when preprocessed metatranscriptome reads (post-PEAR/Trimmomatic/SortMeRNA)
  need organism and functional annotations, and a local protein reference FASTA with
  informative headers is available or can be built.
when_to_use_negative:
- Input is raw unpreprocessed FASTQ still containing adapters, low-quality reads,
  or rRNA — SAMSA2 preprocessing must run first
- The annotation target is 16S rRNA-based taxonomic profiling without activity information;
  the paper notes 16S approaches 'provide no information about potential activities'
- Only a web-based service is acceptable or the user cannot host a local DIAMOND-indexed
  database locally
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: DIAMOND
  role: Accelerated BLAST-like aligner used to run blastx of metatranscriptome reads
    against the local DIAMOND-indexed protein database; replaces MG-RAST and is orders
    of magnitude faster than BLAST
  repo: https://github.com/bbuchfink/diamond
- name: SAMSA2
  role: Pipeline wrapping the annotation step; provides example_DIAMOND_annotation_script.bash
    and DIAMOND_analysis_counter.py to turn per-read alignments into organism/function
    assignments and ranked summaries
  repo: https://github.com/transcript/samsa2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:8228a838d6ffd7fc7ae5b3d6589dd85727b0582cff48b65088e7617cb5a2bc3a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bbuchfink/diamond
  tools:
  - DIAMOND
  - SAMSA2
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# organism-function-assignment-from-alignment

## Summary

Assigns organism and function labels to preprocessed metatranscriptome reads by aligning them with DIAMOND blastx against a local DIAMOND-indexed protein database whose FASTA headers encode sequence ID, function, and organism, then parsing the tabular output into a per-read annotation table.

## When to use

Use when preprocessed metatranscriptome reads (post-PEAR/Trimmomatic/SortMeRNA) need organism and functional annotations, and a local protein reference FASTA with informative headers is available or can be built.

## When NOT to use

- Input is raw unpreprocessed FASTQ still containing adapters, low-quality reads, or rRNA — SAMSA2 preprocessing must run first
- The annotation target is 16S rRNA-based taxonomic profiling without activity information; the paper notes 16S approaches 'provide no information about potential activities'
- Only a web-based service is acceptable or the user cannot host a local DIAMOND-indexed database locally

## Inputs

- Preprocessed metatranscriptome reads in FASTA/FASTQ form (merged by PEAR, trimmed by Trimmomatic, rRNA-filtered by SortMeRNA)
- Local protein reference database in FASTA format with headers encoding sequence ID, function name, and organism name (e.g., NCBI RefSeq bacteria or custom database)
- DIAMOND-indexed version of that database (produced by diamond makedb)

## Outputs

- DIAMOND tabular alignment output (per-read line-item results)
- Per-read plaintext annotation table (e.g., samsa2_annotation_output.txt) with columns for read ID, organism, and function
- Ranked abundance summary produced downstream by DIAMOND_analysis_counter.py

## How to apply

Prepare a local reference protein database in FASTA format — either NCBI RefSeq or a custom database whose headers encode an internal ID, function name, and organism name — and index it with DIAMOND's makedb so all searching stays local (no web service such as MG-RAST). Run DIAMOND blastx to align the preprocessed mRNA reads against the indexed database, following SAMSA2's example_DIAMOND_annotation_script.bash; DIAMOND defaults to --fast mode, switch to --sensitive (lines 162/199 of master_script) if accuracy outweighs speed, accepting higher memory use. Parse the DIAMOND tabular output to extract, per read, the best-matching entry and translate its header into organism and function assignments. Write the per-read annotation table as a plaintext outfile (e.g., samsa2_annotation_output.txt) with read ID, organism, and function columns; DIAMOND returns results on a per-read basis which downstream Python aggregation scripts then condense into ranked abundance summaries via DIAMOND_analysis_counter.py.

## Related tools

- **DIAMOND** (Accelerated BLAST-like aligner used to run blastx of metatranscriptome reads against the local DIAMOND-indexed protein database; replaces MG-RAST and is orders of magnitude faster than BLAST) — https://github.com/bbuchfink/diamond
- **SAMSA2** (Pipeline wrapping the annotation step; provides example_DIAMOND_annotation_script.bash and DIAMOND_analysis_counter.py to turn per-read alignments into organism/function assignments and ranked summaries) — https://github.com/transcript/samsa2

## Examples

```
bash example_DIAMOND_annotation_script.bash  # then: python DIAMOND_analysis_counter.py --input samsa2_annotation_output.txt
```

## Evaluation signals

- Annotation outfile exists and is plaintext with one annotation record per input read
- Each annotation record contains non-empty organism and function fields populated from the reference database headers
- DIAMOND ran against a local DIAMOND-indexed database (makedb completed) rather than any external web service
- Per-read DIAMOND output can be condensed by DIAMOND_analysis_counter.py into a ranked abundance summary without errors
- If sensitivity was a concern, verify whether DIAMOND was switched from --fast to --sensitive and rerun with --sensitive when memory permits

## Limitations

- DIAMOND defaults to --fast mode which prioritizes speed; --sensitive mode 'may significantly increase memory requirements'
- The database must be local and DIAMOND-indexed before annotation; unindexed FASTA cannot be searched
- SAMSA2 aggregation requires Python 2.7 (tested on Linux only), and DIAMOND version 0.8.3 is the version packaged with SAMSA2

## Evidence

- [methods] DIAMOND annotation of reads against a local database: "use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed)"
- [readme] Per-read annotation output: "DIAMOND returns results on a per-read basis, a bit like a ticker tape or a line item receipt."
- [readme] Fast vs sensitive mode decision: [evidence span withheld — anchor status: external_doc]
- [intro] Why DIAMOND over BLAST: "rely on BLAST [9] for annotations, which may be unacceptably slow when processing multiple millions of sequences per sample"
- [abstract] Local databases for flexibility: "SAMSA2 is faster due to the use of the DIAMOND aligner, and more flexible and reproducible because it uses local databases."
