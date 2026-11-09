---
name: cross-assembly-completeness-comparison
description: Use when two assembly versions of the same genome exist and you need
  a head-to-head completeness metric against a curated reference sequence set, such
  as segmental duplications, to quantify assembly improvement.
when_to_use_negative:
- You only have one assembly and no second version to compare against — this skill
  is inherently a two-assembly contrast
- The goal is generic contiguity metrics (N50, genome coverage) rather than representation
  of a specific curated reference sequence set
- Target sequences come from a different species or assembly version than the reference
  used to define them (e.g., non-hg19 SD annotations), which would invalidate the
  coverage baseline
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SOAPdenovo2
  role: Assembler that produced the updated YH v2 assembly whose contigs are aligned
    against the SD sequences in the comparison
  repo: github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo
  role: Assembler that produced the earlier YH v1 (2008) assembly used as the baseline
    in the comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:29a7bd15c8e4427b7735432d221ca81b64269897d0d08414144bda350e090cfb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SOAPdenovo
  - SOAPdenovo2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# cross-assembly-completeness-comparison

## Summary

Quantitatively compares how completely two versions of a genome assembly represent a reference target sequence set (e.g., 134 Mb of human segmental duplication sequences from hg19), by aligning contigs from each assembly and computing the fraction of target sequences sufficiently covered. Used to demonstrate assembly improvement, e.g., YH v1 (21.5% of SDs covered) vs SOAPdenovo2 YH v2 (~99%).

## When to use

Use when two assembly versions of the same genome exist and you need a head-to-head completeness metric against a curated reference sequence set, such as segmental duplications, to quantify assembly improvement.

## When NOT to use

- You only have one assembly and no second version to compare against — this skill is inherently a two-assembly contrast
- The goal is generic contiguity metrics (N50, genome coverage) rather than representation of a specific curated reference sequence set
- Target sequences come from a different species or assembly version than the reference used to define them (e.g., non-hg19 SD annotations), which would invalidate the coverage baseline

## Inputs

- Contig (or scaffold) FASTA files from each assembly version being compared (e.g., YH v1 2008 contigs and SOAPdenovo2 YH v2 contigs)
- Reference target sequences in FASTA, e.g., 134 Mb of human segmental duplication sequences extracted from the hg19 UCSC segmental duplication annotation track

## Outputs

- Per-SD coverage records from short-sequence alignment for each assembly
- A table of the fraction of target SD sequences sufficiently represented (≥90% coverage) per assembly, e.g., 21.5% (YH v1) vs ~99% (YH v2)

## How to apply

Retrieve the reference target sequences — here, 134 Mb of published human SD sequences from the hg19 UCSC segmental duplication track. Align the contigs of the first assembly version against the SD sequences with a short-sequence aligner and record per-SD coverage; a sequence counts as 'sufficiently represented' when ≥90% of its length is covered by aligned contigs. Repeat the identical alignment with the contigs of the updated assembly (e.g., SOAPdenovo2-built YH v2) under the same settings. Compute, per assembly, the fraction of SD sequences with ≥90% coverage and tabulate the two rates side by side. Validate that the output table contains both rates and reproduces the expected contrast (reported: 21.5% for YH v1 vs approximately 99% for YH v2).

## Related tools

- **SOAPdenovo2** (Assembler that produced the updated YH v2 assembly whose contigs are aligned against the SD sequences in the comparison) — github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo** (Assembler that produced the earlier YH v1 (2008) assembly used as the baseline in the comparison)

## Examples

```
SOAPdenovo2 all -s config_file -K 63 -R -o graph_prefix 1>ass.log 2>ass.err; aligner contigs.graph_prefix.contig vs hg19_SD.fa; compute fraction of SDs with >=90% coverage per assembly
```

## Evaluation signals

- The output table contains both assembly versions' representation rates, not just one
- The computed rates reproduce the reported contrast: 21.5% for YH v1 vs approximately 99% for YH v2
- Per-SD coverage was computed with a consistent 'sufficiently represented' threshold of ≥90% of each SD sequence across both assemblies
- Both alignments used identical settings, so rate differences are attributable to assembly quality, not alignment parameters
- Sanity check against independent metrics: YH v2 also shows ~3-fold contig N50 (~20.9 kbp) and 50-fold scaffold N50 (~22 Mbp) improvement and coverage increase from 81.16% to 93.91% over YH v1

## Limitations

- The ≥90%-coverage criterion is a single threshold; SD sequences partially but substantially represented fall on the wrong side of a binary call
- Results depend on the definition of the reference target set (134 Mb of hg19 SD sequences); a different annotation or reference build changes the baseline
- The comparison reflects contig-level representation; SOAPdenovo2 scaffolds/gap closing quality must be assessed separately

## Evidence

- [results] we aligned the contigs of the first version and the new version to 134 Mb of published human SD sequences: "we aligned the contigs of the first version and the new version to 134 Mb of published human SD sequences"
- [abstract] The genome coverage increased from 81.16% to 93.91%: "The genome coverage increased from 81.16% to 93.91%"
- [abstract] contig and scaffold N50 of the YH genome were ~20.9 kbp and ~22 Mbp: "the contig and scaffold N50 of the YH genome were ~20.9 kbp and ~22 Mbp, respectively, which is 3-fold and 50-fold longer than the first published version."
- [readme] a. *.contig contig sequences without using mate pair information.: "a. *.contig
  contig sequences without using mate pair information."
