---
name: hg19-annotation-retrieval
description: Use when an analysis requires a curated set of human segmental duplication
  sequences from hg19, e.g. to measure how completely a de novo assembly represents
  published SD sequences.
when_to_use_negative:
- The reference genome of interest is not hg19/human — the annotation source and 134
  Mb SD set are human-specific.
- The goal is de novo identification of segmental duplications rather than retrieval
  of previously published (annotated) SD sequences.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: SOAPdenovo2
  role: Assembler whose YH v2 contigs (and, for comparison, the 2008 YH v1 contigs)
    are aligned against the retrieved 134 Mb SD sequences to compute representation
    rates
  repo: github.com/aquaskyline/SOAPdenovo2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:c5ac9e37560f26125f9b565c10386384d033745fa5d6e19d7540ff8de7775a7e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SOAPdenovo2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# hg19-annotation-retrieval

## Summary

Retrieves the 134 Mb of published human segmental duplication (SD) sequences from the hg19 (NCBI human reference genome) segmental duplication annotation (UCSC track) to serve as the reference set for assessing assembly representation of SDs.

## When to use

Use when an analysis requires a curated set of human segmental duplication sequences from hg19, e.g. to measure how completely a de novo assembly represents published SD sequences.

## When NOT to use

- The reference genome of interest is not hg19/human — the annotation source and 134 Mb SD set are human-specific.
- The goal is de novo identification of segmental duplications rather than retrieval of previously published (annotated) SD sequences.

## Inputs

- hg19 segmental duplication annotation (UCSC track) for the NCBI human reference genome hg19
- 134 Mb of published human SD sequences derived from that annotation (FASTA)

## Outputs

- FASTA file of 134 Mb of published human segmental duplication sequences
- Per-SD coverage table from contig alignments, yielding per-assembly fraction of SDs with >=90% sequence coverage

## How to apply

Retrieve the hg19 segmental duplication annotation from the UCSC track for the NCBI human reference genome hg19, extracting the full 134 Mb of published human SD sequences as FASTA. These sequences act as the fixed reference against which contigs from different assembly versions (e.g. the 2008 YH v1 and the SOAPdenovo2-built YH v2) are independently aligned. Per-SD coverage from each alignment is then tabulated, and an SD is called 'sufficiently represented' when at least 90% of its sequence is covered by aligned contigs; the fraction of sufficiently represented SDs is compared side by side across assemblies (reported: 21.5% for YH v1 vs up to 99% for YH v2). Validation consists of confirming the output table contains both representation rates and reproduces the reported contrast.

## Related tools

- **SOAPdenovo2** (Assembler whose YH v2 contigs (and, for comparison, the 2008 YH v1 contigs) are aligned against the retrieved 134 Mb SD sequences to compute representation rates) — github.com/aquaskyline/SOAPdenovo2

## Examples

```
SOAPdenovo2 contig-align: align YH v1/v2 contigs to the 134 Mb hg19 SD FASTA and compute the fraction of SDs with >=90% coverage per assembly
```

## Evaluation signals

- The retrieved SD set totals approximately 134 Mb of sequence, matching the published human SD complement.
- The output table contains representation rates for both YH v1 (21.5%) and YH v2 (~99%).
- Per-SD coverage is computed with a >=90% of sequence covered threshold before calling an SD sufficiently represented.
- Both assembly versions are aligned against the SD set under identical aligner settings so the rates are directly comparable.

## Limitations

- Representation is assessed only against SDs annotated in hg19; SDs absent from the annotation cannot be evaluated.
- The 'sufficient representation' definition (>=90% of each SD sequence covered by aligned contigs) is a coarse binary criterion that does not capture partial or fragmented representation below the cutoff.
- SOAPdenovo2 targets Illumina GA short reads on 64-bit Linux, requiring ~5 GB minimum and ~150 GB memory for human-sized genomes, which constrains where the downstream assembly/alignment workflow can run.

## Evidence

- [results] Retrieving SD sequences from hg19 annotation and measuring assembly representation: "we aligned the contigs of the first version and the new version to 134 Mb of published human SD sequences"
- [results] Per-SD coverage threshold used to define sufficient representation: [evidence span withheld — anchor status: not_found]
- [results] Reference genome used as the annotation source: "by aligning them to the NCBI human reference genome hg19"
- [readme] Read formats accepted by the assembler producing the contigs compared against the SD set: [evidence span withheld — anchor status: external_doc]
