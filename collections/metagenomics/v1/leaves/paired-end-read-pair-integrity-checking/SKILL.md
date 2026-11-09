---
name: paired-end-read-pair-integrity-checking
description: Use when a paired-end QC/trimming step (e.g. fastp) has produced cleaned_1.fastq.gz
  and cleaned_2.fastq.gz and you must confirm the pair is synchronized, non-empty,
  and valid gzip FASTQ before hybrid assembly.
when_to_use_negative:
- Input is single-end FASTQ or long-read (Nanopore/PacBio) data — pairing integrity
  is undefined; long reads are filtered with chopper in Plassembler instead
- Reads have already been mapped and split (e.g. chromosomal reads removed by the
  Plassembler minimap2 mapping approach), where mate synchronization is re-established
  by the mapping step itself
- You only need QC statistics (Q20/Q30, adapter content) rather than pair integrity
  — fastp's JSON/HTML report already provides those
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: fastp
  role: Performs the paired-end filtering/trimming whose cleaned R1/R2 outputs are
    validated for read-pair integrity; >=0.24.2 required by Plassembler
  repo: https://github.com/OpenGene/fastp
- name: Plassembler
  role: Pipeline whose short-read QC stage (skippable via --skip_qc) consumes the
    validated read pairs for hybrid plasmid assembly
  repo: https://github.com/gbouras13/plassembler
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:68e04c2727a0cb7afc3384cb4f8a7db024d32c67703f6885565d75a5a7dd2e89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenGene/fastp
  tools:
  - Plassembler
  - fastp
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# paired-end-read-pair-integrity-checking

## Summary

Validate that quality-controlled paired-end short-read FASTQs remain synchronized: both cleaned R1/R2 gzip FASTQ outputs exist, parse as valid gzip FASTQ, and contain matching numbers of reads. This guards the Plassembler short-read QC stage (fastp) so downstream hybrid assembly receives intact read pairs.

## When to use

Use when a paired-end QC/trimming step (e.g. fastp) has produced cleaned_1.fastq.gz and cleaned_2.fastq.gz and you must confirm the pair is synchronized, non-empty, and valid gzip FASTQ before hybrid assembly.

## When NOT to use

- Input is single-end FASTQ or long-read (Nanopore/PacBio) data — pairing integrity is undefined; long reads are filtered with chopper in Plassembler instead
- Reads have already been mapped and split (e.g. chromosomal reads removed by the Plassembler minimap2 mapping approach), where mate synchronization is re-established by the mapping step itself
- You only need QC statistics (Q20/Q30, adapter content) rather than pair integrity — fastp's JSON/HTML report already provides those

## Inputs

- cleaned_1.fastq.gz (R1 output of fastp paired-end QC)
- cleaned_2.fastq.gz (R2 output of fastp paired-end QC)
- original paired-end Illumina FASTQ pair (e.g. *.fastq.gz R1/R2 from Zenodo record 7996690)

## Outputs

- Read-count comparison for R1 vs R2 (must be equal)
- Gzip/FASTQ validity confirmation for both cleaned files
- Pass/fail verdict on read-pair integrity before downstream assembly

## How to apply

After running fastp (>=0.24.2) in paired-end mode with gzip-compressed output on an Illumina R1/R2 pair (e.g. Acinetobacter_baumannii_1.fastq.gz / Acinetobacter_baumannii_2.fastq.gz from Zenodo record 7996690), check that both cleaned outputs exist and are valid gzip files that decompress without error. Parse each file as 4-line FASTQ records and count reads in R1 and R2; the counts must match exactly, since fastp's default paired-end processing keeps mates synchronized via its --unpaired1/--unpaired2 handling of orphans. Verify records are non-empty and have consistent ID, sequence, and quality-line lengths. A mismatch in read counts indicates filter/trimming dropped one mate without its partner, and the QC stage must be re-run before Plassembler's minimap2 mapping and Unicycler hybrid assembly steps.

## Related tools

- **fastp** (Performs the paired-end filtering/trimming whose cleaned R1/R2 outputs are validated for read-pair integrity; >=0.24.2 required by Plassembler) — https://github.com/OpenGene/fastp
- **Plassembler** (Pipeline whose short-read QC stage (skippable via --skip_qc) consumes the validated read pairs for hybrid plasmid assembly) — https://github.com/gbouras13/plassembler

## Examples

```
gzip -t cleaned_1.fastq.gz && gzip -t cleaned_2.fastq.gz && [ $(zcat cleaned_1.fastq.gz | wc -l) -eq $(zcat cleaned_2.fastq.gz | wc -l) ] && echo 'read pair integrity OK'
```

## Evaluation signals

- Both cleaned_1.fastq.gz and cleaned_2.fastq.gz exist and `gzip -t` passes on each
- Read counts in R1 and R2 are exactly equal (e.g. counted via line count / 4)
- Every FASTQ record has 4 lines with matching read-ID prefixes and equal sequence/quality string lengths
- Both files are non-empty after decompression and contain no truncated final record
- Downstream Plassembler QC stage (fastp) completes without reporting unpaired-read leakage into --unpaired1/--unpaired2 outputs

## Limitations

- fastp can route singleton reads to unpaired outputs under its default quality/length filters, so a count mismatch signals orphaned mates rather than a counting error
- The skill validates pairing integrity only, not read quality — fastp's per-base/per-read quality trimming still needs its JSON/HTML report reviewed separately
- Plassembler's entire QC stage (including fastp) can be skipped via --skip_qc, in which case untrimmed input pairs must be checked instead

## Evidence

- [methods] Plassembler's short-read QC stage filters and trims the input paired-end FASTQ files using fastp: [evidence span withheld — anchor status: not_found]
- [methods] fastp >=0.24.2 requirement: "* [fastp](https://github.com/OpenGene/fastp) >=0.24.2"
- [methods] Validation requirement from source workflow: [evidence span withheld — anchor status: not_found]
- [readme] fastp README on filtering out bad reads: [evidence span withheld — anchor status: external_doc]
