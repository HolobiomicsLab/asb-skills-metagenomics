---
name: metatranscriptome-read-annotation
description: Use when raw metatranscriptome FASTQ reads have been preprocessed (PEAR-merged,
  Trimmomatic-trimmed, SortMeRNA rRNA-filtered) and must be assigned organism and
  function labels from a local protein database.
when_to_use_negative:
- Input is still raw paired-end FASTQ that has not been merged, trimmed, and ribosome-filtered
  — run SAMSA2 preprocessing first, since rRNA reads do not contribute to the mRNA
  functional profile
- The goal is 16S rRNA amplicon-style taxonomic profiling rather than expressed-mRNA
  functional annotation
- Only a web-service workflow is acceptable — this skill requires local, DIAMOND-indexed
  databases and a cluster/local compute environment
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: DIAMOND
  role: Accelerated BLAST-like aligner used for blastx annotation of translated reads
    against local DIAMOND-indexed databases; version 0.8.3 as distributed with SAMSA2
  repo: https://github.com/bbuchfink/diamond
- name: SAMSA2
  role: Pipeline wrapping the annotation step (example_DIAMOND_annotation_script.bash,
    master_script.bash) plus aggregation scripts (DIAMOND_analysis_counter.py) that
    convert per-read output into ranked abundance tables
  repo: https://github.com/transcript/samsa2
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_006
  - task_004
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:25d7f227e22972b0ab10b64f40c058688c0ca67b7840b6ef8568ae0d117169ef
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# metatranscriptome-read-annotation

## Summary

Annotate preprocessed metatranscriptome mRNA reads by DIAMOND blastx alignment against a local, DIAMOND-indexed protein reference database (RefSeq, SEED Subsystems, or custom FASTA), producing per-read organism and function assignments at BLAST-like speed without any web service.

## When to use

Use when raw metatranscriptome FASTQ reads have been preprocessed (PEAR-merged, Trimmomatic-trimmed, SortMeRNA rRNA-filtered) and must be assigned organism and function labels from a local protein database.

## When NOT to use

- Input is still raw paired-end FASTQ that has not been merged, trimmed, and ribosome-filtered — run SAMSA2 preprocessing first, since rRNA reads do not contribute to the mRNA functional profile
- The goal is 16S rRNA amplicon-style taxonomic profiling rather than expressed-mRNA functional annotation
- Only a web-service workflow is acceptable — this skill requires local, DIAMOND-indexed databases and a cluster/local compute environment

## Inputs

- Preprocessed metatranscriptome reads (PEAR-merged, Trimmomatic-trimmed, SortMeRNA rRNA-filtered FASTQ/FASTA)
- DIAMOND-indexed protein reference database (NCBI RefSeq bacteria, SEED Subsystems, or custom FASTA with ID/function/organism headers)

## Outputs

- Per-read DIAMOND tabular annotation outfile (plaintext, one record per read with read ID, organism, function)
- Ranked abundance summary tables from DIAMOND_analysis_counter.py / DIAMOND_subsystems_analysis_counter.py (organism and function counts, sorted descending, preserving SEED Subsystems levels 1/2/3)

## How to apply

After preprocessing (PEAR merge of paired-end mates, Trimmomatic adapter/quality trimming, SortMeRNA ribosomal read removal), run DIAMOND blastx to translate and align the mRNA reads against a local DIAMOND-indexed database created with 'diamond makedb'; SAMSA2 ships version 0.8.3 and defaults DIAMOND to '--fast' mode, switching to '--sensitive' (lines 162/199 of master_script.bash) only when accuracy matters and extra memory is available. Reference databases should be NCBI RefSeq for organism/specific-function identification and SEED Subsystems for hierarchical functional ontology; custom FASTA databases whose headers encode sequence ID, function, and organism can be indexed and swapped in without reinstalling the pipeline. Parse the DIAMOND tabular output per read, translate the best-matching header into organism and function fields, and feed the line-by-line results to aggregation scripts (DIAMOND_analysis_counter.py) that condense them into ranked abundance summaries. Rationale: DIAMOND runs at 100x–10,000x BLAST speed, so annotation of millions of sequences per sample completes in minutes rather than the months a 10-annotations-per-second BLAST run would require, and local databases make results reproducible and fully under user control.

## Related tools

- **DIAMOND** (Accelerated BLAST-like aligner used for blastx annotation of translated reads against local DIAMOND-indexed databases; version 0.8.3 as distributed with SAMSA2) — https://github.com/bbuchfink/diamond
- **SAMSA2** (Pipeline wrapping the annotation step (example_DIAMOND_annotation_script.bash, master_script.bash) plus aggregation scripts (DIAMOND_analysis_counter.py) that convert per-read output into ranked abundance tables) — https://github.com/transcript/samsa2

## Examples

```
bash example_DIAMOND_annotation_script.bash  # runs DIAMOND (v0.8.3) blastx against a local DIAMOND-indexed database, then DIAMOND_analysis_counter.py for a ranked abundance summary
```

## Evaluation signals

- The per-read outfile is plaintext with one annotation record per input read and non-empty organism and function fields populated from reference database headers
- On MetaSim-simulated gut microbiome metatranscriptomes, genus-level organism accuracy approximates the reported ~95% (vs 91% for SAMSA 1.0), and hierarchy-level functional accuracy exceeds 95%
- Ranked abundance summary output matches the DIAMOND_analysis_counter.py format: feature label plus count, sorted in descending order of abundance
- Annotation runtime scales approximately linearly with input size at fixed resources (30 CPUs, 2 GB RAM each) and RAM speedup plateaus once the ~6 GB nr Bacteria database fits in memory (~10 CPUs / 20 GB RAM minimum)
- Database files are local and DIAMOND-indexed (diamond makedb completed) so no MG-RAST account or web service was contacted

## Limitations

- DIAMOND defaults to --fast mode prioritizing speed; --sensitive mode may significantly increase memory requirements
- Python 2.7 is required for the aggregation scripts and the pipeline was only tested on Linux despite claiming any OS with Python 2.7
- Reference databases must be local and DIAMOND-indexed; full RefSeq bacteria is ~6 GB, so small allocations of RAM or CPUs slow annotation until the database fits in memory

## Evidence

- [readme] DIAMOND integration, allowing for SAMSA2 to be run without ever needing an MG-RAST account.: "DIAMOND integration, allowing for SAMSA2 to be run without ever needing an MG-RAST account."
- [readme] use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed): "use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed)"
- [abstract] SAMSA2 is faster due to the use of the DIAMOND aligner, and more flexible and reproducible because it uses local databases.: "SAMSA2 is faster due to the use of the DIAMOND aligner, and more flexible and reproducible because it uses local databases."
- [readme] This downloads the full RefSeq bacteria database, for organism and specific functional results, and the SEED Subsystems database, which is used for hierarchical functional ontology.: [evidence span withheld — anchor status: external_doc]
