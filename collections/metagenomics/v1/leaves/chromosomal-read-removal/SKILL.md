---
name: chromosomal-read-removal
description: Use when a hybrid bacterial plasmid assembly pipeline needs to exclude
  chromosomal reads from FASTQ inputs after a long-read-only assembly has been produced,
  to cut compute cost and avoid chromosomal contigs in plasmid output.
when_to_use_negative:
- There is no long-read-only assembly available from which chromosomal contigs can
  be identified — the mapping-based removal depends on Flye/Raven contigs as a reference.
- The input read sets are already plasmid-only or otherwise filtered, so removing
  chromosomal mappings adds no benefit.
- The isolate's plasmids are already completely and correctly captured in the long-read
  assembly and no hybrid assembly step will follow.
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
tools:
- name: Plassembler
  role: implements the mapping-based chromosomal read removal as part of its automated
    plasmid assembly pipeline
  repo: https://github.com/gbouras13/plassembler
- name: Flye
  role: produces the initial long-read-only assembly whose contigs are classified
    as chromosome vs putative plasmid and used as the mapping reference
  repo: https://github.com/fenderglass/Flye
- name: minimap2
  role: maps both the long-read and short-read sets to the Flye assembly to identify
    chromosomal, plasmid and unmapped reads
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: extracts reads mapping to non-chromosome contigs and unmapped reads, which
    are then combined
  repo: https://github.com/samtools/samtools
- name: Raven
  role: optional alternative to Flye for creating the initial long-read-only assembly
    used as the mapping reference
  repo: https://github.com/lbcb-sci/raven
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:09ddd1d27bf9b5d978448799c44010a613df92cf1ad88eaa43c4d8927bbcd661
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gbouras13/plassembler
  tools:
  - Flye
  - Plassembler
  - Raven
  - SAMtools
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# chromosomal-read-removal

## Summary

A mapping-based read-filtering step from Plassembler that removes chromosomal reads from long- and short-read FASTQ inputs, keeping only reads on putative plasmid contigs plus unmapped reads so that downstream hybrid assembly runs on a much smaller, plasmid-enriched read set.

## When to use

Use when a hybrid bacterial plasmid assembly pipeline needs to exclude chromosomal reads from FASTQ inputs after a long-read-only assembly has been produced, to cut compute cost and avoid chromosomal contigs in plasmid output.

## When NOT to use

- There is no long-read-only assembly available from which chromosomal contigs can be identified — the mapping-based removal depends on Flye/Raven contigs as a reference.
- The input read sets are already plasmid-only or otherwise filtered, so removing chromosomal mappings adds no benefit.
- The isolate's plasmids are already completely and correctly captured in the long-read assembly and no hybrid assembly step will follow.

## Inputs

- raw long-read FASTQ (e.g. Oxford Nanopore) from the bacterial isolate
- paired short-read (Illumina) FASTQ files from the same isolate
- Flye (or Raven) long-read-only assembly contigs in FASTA, partitioned into chromosomal and non-chromosome contigs

## Outputs

- filtered_plasmid_reads.fastq.gz — combined long/short reads mapping to putative plasmid contigs plus unmapped reads
- BAM/SAM alignment files of long and short reads against the Flye assembly

## How to apply

First create a long-read-only assembly with Flye (>=2.9; e.g. nanopore high-quality mode) or optionally Raven (>=1.8). Classify Flye contigs by length against the '-c' chromosome threshold to designate chromosomal versus putative plasmid contigs. Map both the long-read set and the paired short-read set to the Flye assembly with Minimap2 (>=2.11). Then, using SAMtools, extract and combine reads that map to the non-chromosome (putative plasmid) contigs and reads that are unmapped — the unmapped fraction is deliberately retained to recover possible plasmids Flye has missed. Write the combined reads (e.g. filtered_plasmid_reads.fastq.gz) for subsequent hybrid assembly with Unicycler (>=0.4.8).

## Related tools

- **Plassembler** (implements the mapping-based chromosomal read removal as part of its automated plasmid assembly pipeline) — https://github.com/gbouras13/plassembler
- **Flye** (produces the initial long-read-only assembly whose contigs are classified as chromosome vs putative plasmid and used as the mapping reference) — https://github.com/fenderglass/Flye
- **minimap2** (maps both the long-read and short-read sets to the Flye assembly to identify chromosomal, plasmid and unmapped reads) — https://github.com/lh3/minimap2
- **SAMtools** (extracts reads mapping to non-chromosome contigs and unmapped reads, which are then combined) — https://github.com/samtools/samtools
- **Raven** (optional alternative to Flye for creating the initial long-read-only assembly used as the mapping reference) — https://github.com/lbcb-sci/raven

## Examples

```
flye --nano-hq long_reads.fastq --out-dir flye_out --threads 8
```

## Evaluation signals

- Validate the output FASTQ with a validate_fastq-style check: per-read sequence and quality lengths must match.
- The filtered read set should contain reads aligned to putative plasmid contigs plus unmapped reads, and should exclude reads mapping to the designated chromosomal contig(s).
- Non-chromosome contigs are correctly designated by the '-c' chromosome length threshold before extraction.
- Downstream hybrid assembly (Unicycler) recovers plasmids missed by the long-read assembly, since unmapped reads were retained.
- minimap2 >=2.11 and SAMtools versions meet the stated minimums (SAMtools >=0.15.0 in the workflow card).

## Limitations

- Retention of unmapped reads is a heuristic to recover plasmids Flye has missed; it cannot recover plasmids absent from any read mapping signal.
- Flye assembly with --deterministic is still not deterministic between thread counts, so the contig classification underlying the filtering can vary between runs.
- Plassembler requires a long-read-first approach; the article notes existing long-read-first methods can misassemble or miss small plasmids without this removal step, and sample heterogeneity (e.g. barcode08 Serratia marcescens) can defeat assembly entirely.

## Evidence

- [abstract] It achieves increased accuracy and computational efficiency compared to the existing gold standard tool Unicycler by removing chromosomal reads from the input read sets using a mapping approach: "It achieves increased accuracy and computational efficiency compared to the existing gold standard tool Unicycler by removing chromosomal reads from the input read sets using a mapping approach"
- [methods] plassembler uses Flye or Raven to create an initial long-read only assembly: "`plassembler` uses Flye (Kolmogorov et al 2019) or optionally Raven (Vaser and Šikić 2021) to create an initial long-read only assembly."
- [methods] Following this, minimap2 (Li 2018) is used to map long and short read sets.: "Following this, minimap2 (Li 2018) is used to map long and short read sets."
- [methods] Reads that map to the non-chromosome contigs (i.e. the putative plasmids), and reads that are unmapped are extracted and combined.: "Reads that map to the non-chromosome contigs (i.e. the putative plasmids), and reads that are unmapped  (to recover possible plasmids Flye has missed) are extracted and combined."
- [methods] These reads are then assembled using Unicycler (Wick et al 2017).: "These reads are then assembled using Unicycler (Wick et al 2017)."
