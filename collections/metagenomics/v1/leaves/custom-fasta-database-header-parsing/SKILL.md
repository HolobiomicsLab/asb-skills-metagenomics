---
name: custom-fasta-database-header-parsing
description: Use when metatranscriptome reads (FASTQ, already preprocessed) must be
  assigned organism and function annotations against a custom or RefSeq protein FASTA
  where header fields (ID, function, organism) must be translated into a per-read
  annotation table, without MG-RAST or any web service.
when_to_use_negative:
- Input is raw, unpreprocessed FASTQ — SAMSA2's preprocessing (PEAR, Trimmomatic,
  SortMeRNA) must run first so rRNA reads are removed before annotation.
- The target database is not available locally or cannot be DIAMOND-indexed (e.g.,
  only accessible via a web service) — the database must be local and DIAMOND-indexed.
- Header lines of the reference FASTA do not encode function and organism information
  in a parseable format, so headers cannot be translated into organism/function assignments.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: DIAMOND
  role: Accelerated BLAST-like aligner; makedb indexes the local FASTA and blastx
    aligns translated reads against it to produce tabular per-read matches
  repo: https://github.com/bbuchfink/diamond
- name: SAMSA2
  role: Pipeline whose annotation stage wraps DIAMOND (example_DIAMOND_annotation_script.bash)
    and parses database headers into a per-read organism/function outfile
  repo: https://github.com/transcript/samsa2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:be3278277e956d94397ce08ba7e139e97119e8ff6a0bca4d21d9427298a62bed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bbuchfink/diamond
  tools:
  - DIAMOND
  - SAMSA2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Custom FASTA database header parsing

## Summary

In SAMSA2's annotation stage, preprocessed metatranscriptome reads are aligned with DIAMOND blastx against a local, DIAMOND-indexed protein FASTA (e.g., NCBI RefSeq or a custom database) whose sequence headers encode internal sequence ID, function name, and organism name, yielding per-read organism and function assignments in a plaintext output table.

## When to use

Use when metatranscriptome reads (FASTQ, already preprocessed) must be assigned organism and function annotations against a custom or RefSeq protein FASTA where header fields (ID, function, organism) must be translated into a per-read annotation table, without MG-RAST or any web service.

## When NOT to use

- Input is raw, unpreprocessed FASTQ — SAMSA2's preprocessing (PEAR, Trimmomatic, SortMeRNA) must run first so rRNA reads are removed before annotation.
- The target database is not available locally or cannot be DIAMOND-indexed (e.g., only accessible via a web service) — the database must be local and DIAMOND-indexed.
- Header lines of the reference FASTA do not encode function and organism information in a parseable format, so headers cannot be translated into organism/function assignments.

## Inputs

- Preprocessed metatranscriptome reads (PEAR-merged, Trimmomatic-trimmed, SortMeRNA rRNA-filtered FASTQ)
- Local protein reference database in FASTA format (NCBI RefSeq bacteria or custom, headers encoding sequence ID, function name, organism name)
- DIAMOND-indexed version of that database (.dmnd from diamond makedb)

## Outputs

- DIAMOND tabular per-read alignment output
- Per-read plaintext annotation table with columns for read ID, organism, and function (samsa2_annotation_output.txt)

## How to apply

Prepare a local reference protein database in FASTA format whose headers encode sequence ID, function, and organism (NCBI RefSeq bacteria or a custom database), then index it with DIAMOND's makedb step so it can be used locally without any external web service. Run DIAMOND in blastx mode (default --fast mode; switch to --sensitive at lines 162/199 of master_script if accuracy outweighs speed and memory cost) to align the preprocessed reads against the indexed database, following the example_DIAMOND_annotation_script.bash approach from the SAMSA2 repository. Parse the DIAMOND tabular output to extract, per read, the best-matching database entry and translate its FASTA header into organism and function assignments. Write the result as a plaintext per-read annotation table (columns: read ID, organism, function) that downstream DIAMOND_analysis_counter.py can condense into ranked abundance summaries.

## Related tools

- **DIAMOND** (Accelerated BLAST-like aligner; makedb indexes the local FASTA and blastx aligns translated reads against it to produce tabular per-read matches) — https://github.com/bbuchfink/diamond
- **SAMSA2** (Pipeline whose annotation stage wraps DIAMOND (example_DIAMOND_annotation_script.bash) and parses database headers into a per-read organism/function outfile) — https://github.com/transcript/samsa2

## Examples

```
diamond makedb --in custom_refseq.fasta -d custom_db && diamond blastx -d custom_db.dmnd -q reads.merged.trimmed.filtered --fast -o samsa2_annotation_output.txt
```

## Evaluation signals

- The DIAMOND-indexed database file (.dmnd) exists and blastx completes without referencing any web service.
- The annotation outfile exists, is plaintext, and contains one annotation record per input read.
- Every record carries non-empty organism and function fields successfully parsed from the reference database FASTA headers.
- Feed the outfile to DIAMOND_analysis_counter.py and confirm a ranked abundance summary is produced without parse errors.
- With the full RefSeq bacteria database, genus-level assignment accuracy of ~95% is expected (vs. 91% for SAMSA 1.0 via MG-RAST).

## Limitations

- DIAMOND is set by default to run in fast mode, prioritizing speed; switching to --sensitive may significantly increase memory requirements.
- SAMSA2 aggregation scripts require Python 2.7 (tested on Linux); Python 3 support is incomplete.
- Custom database headers must contain the encoded ID, function, and organism fields — unstructured headers cannot yield organism/function assignments.
- A standard BLAST approach would take several months at ~10 annotations per second for a single file, which is why DIAMOND replaces it.

## Evidence

- [methods] Local DIAMOND-indexed database requirement: "use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed)"
- [readme] DIAMOND replaces BLAST for annotation: "Annotation is completed using DIAMOND, an accelerated BLAST-like sequence aligner."
- [discussion] Custom databases can be indexed and added: "custom databases that may be indexed and used as references allows SAMSA2 to be upgraded when newer database versions become available"
- [readme] Per-read DIAMOND output condensed by aggregation: "DIAMOND returns results on a per-read basis, a bit like a ticker tape or a line item receipt."
- [readme] Full database download script contents: [evidence span withheld — anchor status: external_doc]
