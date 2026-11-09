---
name: segmental-duplication-sequence-alignment
description: Use when comparing two genome assembly versions for how completely they
  represent segmental duplications, or when quantifying SD representation via contig-to-SD
  coverage fractions.
when_to_use_negative:
- The question is overall assembly accuracy (misassemblies, substitution/copy-number
  error) measured against a reference rather than representation of duplicated sequences
- The comparison target is a single genome's coverage of a reference (e.g. hg19 alignment
  for genome coverage), not a side-by-side contrast of two assembly versions against
  SD sequences
- Reads or scaffolds rather than contigs are the input; the reported SD-representation
  metric is computed from contigs
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: SOAPdenovo2
  role: Produces the updated YH v2 assembly whose contigs are aligned against the
    134 Mb SD sequences; the SD representation contrast (21.5% vs ~99%) demonstrates
    its improvement over the 2008 YH version
  repo: github.com/aquaskyline/SOAPdenovo2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:f45702149c54f8194aa7588345c21be1846674a519ab1bb0efcb17cb45337270
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SOAPdenovo2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# segmental-duplication-sequence-alignment

## Summary

A comparison procedure that aligns de novo assembly contigs (YH v1 vs SOAPdenovo2 YH v2) against 134 Mb of published human segmental duplication (SD) sequences from the hg19 UCSC SD track, computing the fraction of SD sequence sufficiently covered (>=90% per SD) to quantify how well an assembly represents duplicated regions.

## When to use

Use when comparing two genome assembly versions for how completely they represent segmental duplications, or when quantifying SD representation via contig-to-SD coverage fractions.

## When NOT to use

- The question is overall assembly accuracy (misassemblies, substitution/copy-number error) measured against a reference rather than representation of duplicated sequences
- The comparison target is a single genome's coverage of a reference (e.g. hg19 alignment for genome coverage), not a side-by-side contrast of two assembly versions against SD sequences
- Reads or scaffolds rather than contigs are the input; the reported SD-representation metric is computed from contigs

## Inputs

- Contig FASTA files from the first-version (2008) YH assembly
- Contig FASTA files from the SOAPdenovo2-updated YH v2 assembly
- 134 Mb of published human SD sequences from the hg19 UCSC segmental duplication annotation

## Outputs

- Per-SD alignment coverage records for each assembly version
- Table of fraction of SD sequences sufficiently represented (>=90% coverage) per assembly: 21.5% (YH v1) vs ~99% (YH v2)

## How to apply

Retrieve the 134 Mb of published human SD sequences from the hg19 segmental duplication annotation (UCSC track). Align the contigs of the first-version (2008) YH assembly against the SD sequences using a short-sequence aligner and record per-SD coverage; the article demonstrated the value of this contrast by first reassembling YH with SOAPdenovo2, whose contig N50 (~20.9 kbp) was 3-fold longer and genome coverage raised from 81.16% to 93.91%. Repeat the alignment with contigs of the updated SOAPdenovo2 YH v2 assembly under identical settings. For each assembly compute the fraction of SD sequences sufficiently represented by aligned contigs (>=90% of each SD sequence) and tabulate the two rates side by side; the paper reports 21.5% for YH v1 versus up to ~99% for YH v2. Verify the output table contains both rates and reproduces the reported contrast.

## Related tools

- **SOAPdenovo2** (Produces the updated YH v2 assembly whose contigs are aligned against the 134 Mb SD sequences; the SD representation contrast (21.5% vs ~99%) demonstrates its improvement over the 2008 YH version) — github.com/aquaskyline/SOAPdenovo2

## Examples

```
SOAPdenovo2 all -s config_file -K 63 -R -o YHv2 1>ass.log 2>ass.err
```

## Evaluation signals

- The output table contains exactly two sufficiently-represented rates, one per assembly version, computed with identical alignment settings
- YH v1 rate falls near the reported 21.5% and YH v2 rate near 99% for SD sequences covered at >=90% of their length
- Per-SD coverage values are in [0, 1] and every one of the 134 Mb SD input sequences has a coverage record
- Re-running the pipeline with the same contig and SD FASTA inputs reproduces the same tabulated rates
- The YH v2 contigs exhibit the expected assembly-quality context: contig N50 ~20.9 kbp and genome coverage of 93.91%

## Limitations

- The sufficiency threshold is fixed at >=90% of each SD sequence covered; SDs below this are counted as insufficiently represented even if partially covered
- Results are specific to the 134 Mb hg19 SD annotation and the YH Asian genome; transfer to other genomes or SD tracks requires revalidation
- SOAPdenovo2 requires substantial compute resources: minimum 5G physical memory, ~150 GB for human-sized genomes, and runs on 64-bit Linux

## Evidence

- [results] we aligned the contigs of the first version and the new version to 134 Mb of published human SD sequences: "we aligned the contigs of the first version and the new version to 134 Mb of published human SD sequences"
- [abstract] updated assembly version of the 2008 Asian (YH) genome: "We also provide an updated assembly version of the 2008 Asian (YH) genome using SOAPdenovo2."
- [abstract] genome coverage increased from 81.16% to 93.91%: "The genome coverage increased from 81.16% to 93.91%, and memory consumption was ~2/3 lower during the point of largest memory consumption."
- [results] SD sequences from hg19 annotation: "by aligning them to the NCBI human reference genome hg19"
- [readme] how to run the assembler end-to-end: [evidence span withheld — anchor status: external_doc]
