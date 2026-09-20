---
name: nanopore-read-subsampling-coverage-normalization
description: Use when pooled Nanopore FASTQs of uneven depth must be subsampled to
  a fixed coverage (e.g. 50x or 60x) before assembly benchmarking or fair tool comparison.
when_to_use_negative:
- Read sets are already at the desired uniform depth — re-subsampling discards data
  unnecessarily
- The goal is to evaluate assembler performance on full-depth data rather than to
  match a fixed-depth benchmark design
- Reads are short Illumina reads rather than long Nanopore reads; rasusa works generally
  but the barcode-splitting and validate_fastq.py workflow described here is Nanopore-pool
  specific
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: rasusa
  role: Subsamples fixed FASTQ files to a target coverage (50x, then 60x) for normalization
  repo: https://github.com/mbhall88/rasusa
- name: grep
  role: Recovers per-barcode reads from pooled FASTQs to split isolates before subsampling
- name: validate_fastq.py
  role: Repairs and validates FASTQ records after splitting pooled files, yielding
    validated reads for subsampling
  repo: https://github.com/gbouras13/plassembler_simulation_benchmarking
- name: fastq-dl
  role: Downloads raw read data (e.g. PRJNA422511) from ENA prior to processing
  repo: https://github.com/rpetit3/fastq-dl
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:66b379ba470be84b78e174b44233e145d8b6dd70f9eacb6001ad333af6bbae94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbhall88/rasusa
  tools:
  - Flye
  - Plassembler
  - Raven
  - Unicycler
  - fastq-dl
  - grep
  - minimap2
  - rasusa
  - validate_fastq.py
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# nanopore-read-subsampling-coverage-normalization

## Summary

Normalizes pooled Nanopore plasmid-enrichment read sets to a fixed, comparable coverage depth using rasusa (50x per isolate, 60x for benchmarking) so that assembler comparisons like Plassembler vs Unicycler are run on equal-depth inputs matching the reported Wick et al. (2021) real-read benchmark conditions.

## When to use

Use when pooled Nanopore FASTQs of uneven depth must be subsampled to a fixed coverage (e.g. 50x or 60x) before assembly benchmarking or fair tool comparison.

## When NOT to use

- Read sets are already at the desired uniform depth — re-subsampling discards data unnecessarily
- The goal is to evaluate assembler performance on full-depth data rather than to match a fixed-depth benchmark design
- Reads are short Illumina reads rather than long Nanopore reads; rasusa works generally but the barcode-splitting and validate_fastq.py workflow described here is Nanopore-pool specific

## Inputs

- Pooled Nanopore FASTQ files from plasmid-enrichment runs (Small_plasmid_Nanopore_data tarballs, Wick et al. Monash Bridges)
- Per-isolate barcode sequences used with grep to split pooled FASTQs
- Target coverage depth specification (e.g. 50x or 60x)

## Outputs

- Per-isolate validated FASTQ files split from the pooled reads
- Subsampled FASTQ files at fixed coverage (50x / 60x) ready for Plassembler and Unicycler assembly

## How to apply

Download the pooled real-read FASTQs (e.g. the six Wick et al. 2021a isolates from the Small-plasmid-Nanopore deposit) and split them per isolate by recovering barcode sequences from the pooled files using grep. Repair and validate the resulting FASTQs with a validation script (validate_fastq.py) since pooled files can contain malformed records. Then run rasusa (mbhall88/rasusa) to subsample each fixed FASTQ to a target coverage — 50x per file to reduce read sizes, and 60x for the benchmark read sets used to compare Plassembler (Flye >=2.9, Raven >=1.8) and Unicycler (>=0.4.8) against the Trycycler-derived manually curated ground truth. The rationale is that equal, pre-specified depth removes read-depth as a confounder when evaluating genome fraction, indels per 100 kbp, and missed plasmids with QUAST.

## Related tools

- **rasusa** (Subsamples fixed FASTQ files to a target coverage (50x, then 60x) for normalization) — https://github.com/mbhall88/rasusa
- **grep** (Recovers per-barcode reads from pooled FASTQs to split isolates before subsampling)
- **validate_fastq.py** (Repairs and validates FASTQ records after splitting pooled files, yielding validated reads for subsampling) — https://github.com/gbouras13/plassembler_simulation_benchmarking
- **fastq-dl** (Downloads raw read data (e.g. PRJNA422511) from ENA prior to processing) — https://github.com/rpetit3/fastq-dl

## Examples

```
rasusa --coverage 60 --genome-size 5000000 --input pooled.fastq --output subsampled_60x.fastq
```

## Evaluation signals

- Each subsampled FASTQ yields approximately the target coverage (50x or 60x) over the isolate's reference plasmids/chromosome
- Number of reads in the subsampled FASTQ is consistent with rasusa's reported subsample count at the requested depth
- All retained reads pass FASTQ validation (equal sequence/quality lengths, valid 4-line records) after validate_fastq.py
- Downstream QUAST genome-fraction values on Plassembler and Unicycler runs (mean 97.02, median 100) are reproducible on these 60x read sets
- Per-isolate FASTQs contain only the expected isolate's reads (barcode-specific split confirmed)

## Limitations

- Subsampling discards reads, so very low-depth isolates may fall below the target coverage and yield fewer reads than requested
- Pooled-run splitting via grep assumes correct barcode sequences; misbarcoded or chimeric records require the validate_fastq.py repair step
- Flye even with --deterministic is not deterministic between thread counts, so subsampled read sets alone do not guarantee bit-identical assemblies

## Evidence

- [methods] read subsampling to 50x then 60x coverage with rasusa: "I ran [rasusa](https://github.com/mbhall88/rasusa) to get 50x coverage on each fixed fastq file to reduce the read sizes"
- [methods] barcode splitting from pooled FASTQs: "I split the files using grep as follows"
- [methods] FASTQ repair/validation: "Accordingly I wrote a little script called validate_fastq.py to get all the validated fastq reads."
- [methods] ground truth for real-read benchmark: "independent Trycyler derived and manually curated ground truth you can find [here](https://github.com/rrwick/Small-plasmid-Nanopore/blob/main/method.md)"
- [methods] real-read benchmark outcome at fixed depth: "plassembler recovered 2 additional small plasmids missed by Unicycler of lengths 1934 bp  _(_K. variicola_ INF345) and 10697 bp (_K. oxytoca_ MSB1 2C)."
