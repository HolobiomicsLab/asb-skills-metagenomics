---
name: assembly-contig-sd-representation-quantification
description: Use when comparing how completely two de novo genome assembly versions
  represent annotated repeat/segmental duplication sequences, or when auditing whether
  an assembler upgrade improved recovery of known genomic sequence.
when_to_use_negative:
- The goal is assembly base-level or structural error assessment (e.g. substitution,
  copy number, or structural error rates vs a reference) rather than sequence-recovery
  completeness of annotated regions
- Input is unassembled read data rather than contigs from a completed assembly — the
  metric is defined on aligned assembly contigs
- No reference annotation of the target repeat class (e.g. hg19 segmental duplication
  track) exists for the genome in question
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
tools:
- name: SOAPdenovo2
  role: De novo short-read assembler used to produce the updated YH v2 assembly whose
    contigs are aligned to the SD sequences; improved SD representation (~99%) is
    attributed to its improved contig assembly, scaffolding, and GapCloser modules
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo (v1)
  role: Original assembler that produced the first-version (2008) YH assembly whose
    contigs serve as the baseline for the 21.5% SD representation rate
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:970ba47864587ef4724aa01d68c58a4ecee9144c893bafdcd15f7feec9de504d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/SOAPdenovo2
  tools:
  - SOAPdenovo (v1)
  - SOAPdenovo2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# assembly-contig-sd-representation-quantification

## Summary

Quantifies how completely a de novo assembly represents known human segmental duplication (SD) sequences by aligning assembly contigs to a reference SD set (134 Mb of hg19 SD sequences) and computing the fraction of SDs sufficiently covered (≥90% of each sequence). Used in the SOAPdenovo2 paper to show the updated YH v2 assembly represents up to 99% of published SDs versus only 21.5% for the first (2008) YH version.

## When to use

Use when comparing how completely two de novo genome assembly versions represent annotated repeat/segmental duplication sequences, or when auditing whether an assembler upgrade improved recovery of known genomic sequence.

## When NOT to use

- The goal is assembly base-level or structural error assessment (e.g. substitution, copy number, or structural error rates vs a reference) rather than sequence-recovery completeness of annotated regions
- Input is unassembled read data rather than contigs from a completed assembly — the metric is defined on aligned assembly contigs
- No reference annotation of the target repeat class (e.g. hg19 segmental duplication track) exists for the genome in question

## Inputs

- Contig FASTA files from each assembly version being compared (e.g. 2008 YH v1 and SOAPdenovo2 YH v2 contigs)
- 134 Mb of published human segmental duplication (SD) sequences from the hg19 UCSC segmental duplication annotation track

## Outputs

- Per-SD coverage table (each SD sequence with fraction covered by aligned contigs, per assembly)
- Fraction of SD sequences sufficiently represented (≥90% coverage) per assembly, tabulated side by side (YH v1: 21.5%; YH v2: ~99%)

## How to apply

Retrieve the 134 Mb of published human SD sequences from the hg19 segmental duplication annotation (UCSC track). Extract the contigs (not scaffolds) from each assembly version under comparison — here the first-version (2008) YH assembly and the SOAPdenovo2-updated YH v2 assembly. Align the contigs of each version against the SD sequences with a short-sequence aligner under identical settings, recording per-SD coverage. Define an SD as 'sufficiently represented' when ≥90% of its sequence is covered by aligned contigs, then compute the fraction of sufficiently represented SDs per assembly and tabulate the two rates side by side (reported: 21.5% for YH v1 vs ~99% for YH v2). Interpret the contrast alongside complementary assembly-quality metrics such as contig/scaffold N50 and reference-genome coverage.

## Related tools

- **SOAPdenovo2** (De novo short-read assembler used to produce the updated YH v2 assembly whose contigs are aligned to the SD sequences; improved SD representation (~99%) is attributed to its improved contig assembly, scaffolding, and GapCloser modules) — https://github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo (v1)** (Original assembler that produced the first-version (2008) YH assembly whose contigs serve as the baseline for the 21.5% SD representation rate)

## Examples

```
${bin} all -s config_file -K 63 -R -o graph_prefix 1>ass.log 2>ass.err
```

## Evaluation signals

- Output table contains both representation rates and they reproduce the reported contrast: ~21.5% for YH v1 vs ~99% for YH v2
- Every one of the SD sequences from the 134 Mb hg19 SD set carries a computed coverage value from both assemblies
- The ≥90% per-SD coverage threshold is applied identically to both assembly versions (identical aligner settings)
- Contig (not scaffold) sequences are used as alignment queries, matching the paper's procedure of aligning 'the contigs of the first version and the new version'
- Direction of the result is consistent with supporting metrics: genome coverage increased from 81.16% to 93.91% and contig/scaffold N50 rose from 3-fold and 50-fold longer in YH v2

## Limitations

- Representation (coverage of known SDs) does not measure base-level or structural accuracy; high coverage can coexist with assembly errors, so pair it with error-rate and N50 metrics
- The metric is bounded by the reference SD annotation (hg19 UCSC track); SDs absent from the annotation are not assessed
- Sensitivity depends on the short-sequence aligner's ability to map short contigs to the SD set; misassembled or fragmented contigs can deflate measured representation

## Evidence

- [results] we aligned the contigs of the first version and the new version to 134 Mb of published human SD sequences: "we aligned the contigs of the first version and the new version to 134 Mb of published human SD sequences"
- [results] by aligning them to the NCBI human reference genome hg19: "by aligning them to the NCBI human reference genome hg19"
- [abstract] The genome coverage increased from 81.16% to 93.91%: "The genome coverage increased from 81.16% to 93.91%, and memory consumption was ~2/3 lower during the point of largest memory consumption."
- [abstract] the contig and scaffold N50 of the YH genome were ~20.9 kbp and ~22 Mbp, respectively: "the contig and scaffold N50 of the YH genome were ~20.9 kbp and ~22 Mbp, respectively, which is 3-fold and 50-fold longer than the first published version."
- [readme] ${bin} all -s config_file -K 63 -R -o graph_prefix: [evidence span withheld — anchor status: external_doc]
