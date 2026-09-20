---
name: protein-alignment-output-parsing
description: Use when metatranscriptome reads (FASTQ, preprocessed) have been aligned
  with DIAMOND blastx against a local DIAMOND-indexed protein database and the tabular
  per-read output must be converted into organism/function annotation records.
when_to_use_negative:
- Input is already a summarized feature/abundance table rather than raw per-read DIAMOND
  tabular output (aggregation, not parsing, is the correct step)
- The database is not local and DIAMOND-indexed — SAMSA2 requires the database to
  be local and DIAMOND-indexed; web-service annotation (e.g., MG-RAST) is out of scope
- The goal is nucleotide-level 16S/rRNA identification — SortMeRNA-filtered mRNA profiles
  via protein alignment will not provide rRNA-based organism calls
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0623
tools:
- name: DIAMOND
  role: Runs blastx to align preprocessed reads against the local DIAMOND-indexed
    protein database and emits the per-read tabular output that is parsed
  repo: https://github.com/bbuchfink/diamond
- name: SAMSA2
  role: Provides example_DIAMOND_annotation_script.bash and the parsing/aggregation
    scripts (DIAMOND_analysis_counter.py) that convert raw alignments into per-read
    and summarized annotation tables
  repo: https://github.com/transcript/samsa2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:aac7d578045316cc4583918ed968dcff7b85e315415d286b75999aa43f110f32
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
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Protein-alignment output parsing

## Summary

Parses DIAMOND blastx tabular output from SAMSA2's annotation step into a per-read plaintext table of read ID, organism, and function assignments decoded from custom reference database FASTA headers.

## When to use

Use when metatranscriptome reads (FASTQ, preprocessed) have been aligned with DIAMOND blastx against a local DIAMOND-indexed protein database and the tabular per-read output must be converted into organism/function annotation records.

## When NOT to use

- Input is already a summarized feature/abundance table rather than raw per-read DIAMOND tabular output (aggregation, not parsing, is the correct step)
- The database is not local and DIAMOND-indexed — SAMSA2 requires the database to be local and DIAMOND-indexed; web-service annotation (e.g., MG-RAST) is out of scope
- The goal is nucleotide-level 16S/rRNA identification — SortMeRNA-filtered mRNA profiles via protein alignment will not provide rRNA-based organism calls

## Inputs

- Preprocessed metatranscriptome reads (FASTQ, rRNA-filtered)
- Local protein reference database in FASTA format with headers encoding sequence ID, function name, and organism name (e.g., NCBI RefSeq bacteria or custom)
- DIAMOND-indexed database produced by diamond makedb
- DIAMOND blastx tabular output (per-read alignments)

## Outputs

- Per-read plaintext annotation table (samsa2_annotation_output.txt) with columns: read ID, organism, function
- Ranked abundance summary tables produced by DIAMOND_analysis_counter.py (downstream)

## How to apply

Prepare the reference protein FASTA (e.g., NCBI RefSeq or a custom database) so each sequence header encodes an internal sequence ID, a function name, and an organism name, then index it with DIAMOND makedb so it is usable locally without any web service. Run DIAMOND blastx to align the preprocessed reads against the indexed database, following SAMSA2's example_DIAMOND_annotation_script.bash; DIAMOND defaults to --fast mode, which can be switched to --sensitive in the master_script (lines 162 and 199) at the cost of higher memory. Parse the DIAMOND tabular output to extract, per read, the best-matching database entry and decode its header into organism and function assignments. Write the result as a plaintext per-read outfile (samsa2_annotation_output.txt) with columns for read ID, organism, and function; downstream DIAMOND_analysis_counter.py then condenses these line-item results into ranked abundance summaries. The rationale for DIAMOND over BLAST is speed: at ~10 annotations/second, BLAST would take months on a single metatranscriptome file.

## Related tools

- **DIAMOND** (Runs blastx to align preprocessed reads against the local DIAMOND-indexed protein database and emits the per-read tabular output that is parsed) — https://github.com/bbuchfink/diamond
- **SAMSA2** (Provides example_DIAMOND_annotation_script.bash and the parsing/aggregation scripts (DIAMOND_analysis_counter.py) that convert raw alignments into per-read and summarized annotation tables) — https://github.com/transcript/samsa2

## Examples

```
diamond blastx --db refseq_bacteria.dmnd --query preprocessed_reads.fq --out samsa2_annotation_output.txt --outfmt 6 --fast
```

## Evaluation signals

- The plaintext outfile exists and contains one annotation record per input read
- Every record has non-empty organism and function fields populated from the reference database FASTA headers
- Read IDs in the outfile correspond to reads present in the preprocessed FASTQ input
- Running DIAMOND_analysis_counter.py on the outfile yields a ranked abundance summary consistent with the per-read counts
- The DIAMOND index was built with diamond makedb from the header-annotated FASTA (no external/web-service annotation was used)

## Limitations

- DIAMOND is set to --fast mode by default, prioritizing speed over accuracy; --sensitive mode may significantly increase memory requirements
- Custom databases must carry headers encoding ID, function, and organism — databases lacking this header structure cannot be translated into organism/function assignments
- SAMSA2's aggregation scripts depend on Python 2.7 and the pipeline was only tested on Linux, despite claiming any OS supporting Python 2.7

## Evidence

- [methods] Reads are annotated against a DIAMOND-indexed database with headers encoding ID, function, and organism; output is a per-read plaintext file.: "use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed)"
- [readme] DIAMOND tabular per-read results are parsed into summary tables.: "DIAMOND returns results on a per-read basis, a bit like a ticker tape or a line item receipt. In the aggregation step, Python scripts condense these line-by-line results to create summary tables."
- [readme] DIAMOND is fast-mode by default; sensitive mode costs memory.: [evidence span withheld — anchor status: external_doc]
- [readme] DIAMOND was chosen over BLAST for speed on millions of metatranscriptome reads.: "At a standard rate of 10 annotations per second, a standard BLAST approach would take several months to finish - just for a single file!"
- [abstract] SAMSA2 is faster and more reproducible because it uses DIAMOND with local databases.: "SAMSA2 is faster due to the use of the DIAMOND aligner, and more flexible and reproducible because it uses local databases."
