---
name: host-read-alignment-against-human-genome
description: Use when raw paired-end FASTQ files from ribodepleted or poly(A)-enriched
  stool metatranscriptomes must be screened for human-derived reads, e.g. to test
  whether host transcriptomes are recoverable or to quantify host contamination.
when_to_use_negative:
- Input is already a SAMSA/MG-RAST annotation summary file (organism or functional
  tables) — those pipeline outputs contain no raw FASTQ to align against the human
  genome.
- The goal is microbial metatranscriptome annotation, where MG-RAST (or DIAMOND in
  SAMSA2) is used and remaining rRNA reads are discarded instead — host-read quantification
  is a separate alignment step.
- Sequences are short single-end reads under 100 bp — SAMSA requires long reads (>100
  bp) or joined paired-end reads, so raw unjoined short reads cannot be processed
  this way without a merging step.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: SAMSA
  role: Pipeline whose preprocessing stage (cleaning, adapter removal, paired-end
    alignment) prepares the reads before human genome alignment
  repo: https://github.com/transcript/SAMSA
- name: Trimmomatic
  role: Performs cleaning and adapter removal on each raw paired-end FASTQ file prior
    to joining
- name: FLASH
  role: Joins cleaned paired-end reads into longer fragments suitable for alignment
    (SAMSA aligns paired-end files after cleaning)
- name: Python 2.7
  role: Scripting language for the SAMSA pipeline scripts that drive preprocessing
    and count tabulation
  repo: https://github.com/transcript/SAMSA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:f78b3ffe007462222f06a0e9d219a67e34565304155cc8641fa0870a28489b0f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/SAMSA
  tools:
  - FLASH
  - Python 2.7
  - SAMSA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# host-read-alignment-against-human-genome

## Summary

Quantifies host transcript recovery from human stool metatranscriptomes by cleaning FASTQ reads with Trimmomatic, joining paired-end reads with FLASH, aligning them against the human genome reference, and tabulating human-matching reads per replicate as counts and percentages of total reads.

## When to use

Use when raw paired-end FASTQ files from ribodepleted or poly(A)-enriched stool metatranscriptomes must be screened for human-derived reads, e.g. to test whether host transcriptomes are recoverable or to quantify host contamination.

## When NOT to use

- Input is already a SAMSA/MG-RAST annotation summary file (organism or functional tables) — those pipeline outputs contain no raw FASTQ to align against the human genome.
- The goal is microbial metatranscriptome annotation, where MG-RAST (or DIAMOND in SAMSA2) is used and remaining rRNA reads are discarded instead — host-read quantification is a separate alignment step.
- Sequences are short single-end reads under 100 bp — SAMSA requires long reads (>100 bp) or joined paired-end reads, so raw unjoined short reads cannot be processed this way without a merging step.

## Inputs

- Raw paired-end FASTQ files from ribodepleted (P1) stool metatranscriptome samples
- Raw paired-end FASTQ files from poly(A)-enriched (P4) stool metatranscriptome samples
- Human genome reference sequence for read alignment

## Outputs

- human_read_alignment_counts.tsv with per-replicate human-matching read counts (expected: 417, 340, 2273, 2100)
- Per-replicate percentages of total reads aligning to the human genome

## How to apply

Retrieve raw paired-end FASTQ files for each replicate (here: two ribodepleted P1 and two poly(A)-enriched P4 stool metatranscriptome samples). Clean each FASTQ with Trimmomatic for adapter removal and quality trimming as part of SAMSA preprocessing, then join the cleaned paired-end reads with FLASH, since the SAMSA pipeline requires sequences longer than 100 bp (long reads or joined paired-end reads). Align the resulting joined reads against the human genome reference to identify host-derived reads. Count human-matching reads per replicate and express each as a percentage of total reads; in the source study the expected counts were 417 and 340 (0.0019% and 0.0085%) for ribodepleted and 2273 and 2100 (0.116% and 0.097%) for poly(A)-enriched samples, supporting the conclusion that host transcriptomes cannot be reliably extracted from stool metatranscriptomes sequenced by standard methods.

## Related tools

- **SAMSA** (Pipeline whose preprocessing stage (cleaning, adapter removal, paired-end alignment) prepares the reads before human genome alignment) — https://github.com/transcript/SAMSA
- **Trimmomatic** (Performs cleaning and adapter removal on each raw paired-end FASTQ file prior to joining)
- **FLASH** (Joins cleaned paired-end reads into longer fragments suitable for alignment (SAMSA aligns paired-end files after cleaning))
- **Python 2.7** (Scripting language for the SAMSA pipeline scripts that drive preprocessing and count tabulation) — https://github.com/transcript/SAMSA

## Examples

```
python SAMSA_pre_annotation_pipeline.py -i sample_P1_R1.fastq -j sample_P1_R2.fastq -t Trimmomatic-0.36.jar -f flash -o cleaned_sample_P1
```

## Evaluation signals

- Tabulated human-matching counts must match the reported values: 417 and 340 for the two ribodepleted replicates, 2273 and 2100 for the two poly(A)-enriched replicates.
- Computed percentages must be in the reported ranges: ~0.0019–0.0085% (ribodepleted) and ~0.097–0.116% (poly(A)-enriched) of total reads.
- Qualitative criterion: only a tiny fraction of total stool reads align to the human genome under both library-preparation conditions, confirming host transcriptomes are not reliably recoverable.
- Joined read lengths must satisfy SAMSA's requirement of being longer than 100 bp before alignment.

## Limitations

- Host transcriptomes cannot be reliably extracted from stool metatranscriptomes sequenced with standard methods (ribodepletion or poly(A) enrichment), as only ~0.001–0.1% of reads align to the human genome.
- SAMSA's annotation relies on MG-RAST, whose servers require queue waits of days to weeks; MG-RAST is not tailored to work with eukaryotic organisms.
- Only one commercially available ribodepletion kit was tested in the underlying study.
- Each sample needs 40–50 million raw sequences to yield the 5–10 million annotated reads necessary for accurate abundance measures.

## Evidence

- [methods] it performs cleaning and removal of adaptor contamination, and then aligns paired-end files (if specified): "it performs cleaning and removal of adaptor contamination, and then aligns paired-end files (if specified)"
- [abstract] Sequences need to be either long reads (longer than 100 bp) or joined paired-end reads: "Sequences need to be either long reads (longer than 100 bp) or joined paired-end reads."
- [methods] SAMSA has several dependencies, including the programs FLASH and Trimmomatic: "SAMSA has several dependencies, including the programs FLASH and Trimmomatic, Python version 2.7, and R"
- [readme] SAMSA_pre_annotation_pipeline.py - runs Trimmomatic, FLASH (if paired-end files), prepares raw sequences for upload to MG-RAST: "SAMSA_pre_annotation_pipeline.py - runs Trimmomatic, FLASH (if paired-end files), prepares raw sequences for upload to MG-RAST"
- [abstract] Each sample needs 40-50 million raw sequences: "Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures."
