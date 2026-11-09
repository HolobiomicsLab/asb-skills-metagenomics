---
name: plasmid-read-extraction
description: Use when a bacterial hybrid-assembly workflow needs to remove chromosomal
  reads from FASTQ inputs to speed up plasmid assembly, retaining unmapped reads and
  reads on putative plasmid contigs.
when_to_use_negative:
- Input is a long-read-only (kmer_mode) Plassembler run with no short-read set — the
  hybrid read-extraction design described here presumes both long- and short-read
  inputs
- The sample has no expected plasmids or the assembly consists of a single chromosomal
  contig — extraction would reduce to retaining unmapped reads only
- Reads are from eukaryotic or metagenomic samples, outside Plassembler's bacterial-isolate
  hybrid assembly scope
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Flye
  role: Creates the initial long-read-only assembly that read mapping is performed
    against
  repo: https://github.com/mikolomogorov/Flye
- name: minimap2
  role: Maps the long and short read sets to the long-read-only assembly (version
    >=2.11)
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Extracts reads mapping to non-chromosome (putative plasmid) contigs and unmapped
    reads (version >=0.15.0)
  repo: https://github.com/samtools/samtools
- name: Plassembler
  role: Implements the full chromosomal-read-removal module and writes the filtered
    read set
  repo: https://github.com/gbouras13/plassembler
- name: Raven
  role: Optional alternative (>=1.8) to Flye for the initial long-read-only assembly
  repo: https://github.com/lbcb-sci/raven
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:691b2a68841e367923020d9f898b9aea793afbf844e285cf309400354a02843d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikolomogorov/Flye
  tools:
  - Flye
  - Plassembler
  - Raven
  - SAMtools
  - minimap2
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# plasmid-read-extraction

## Summary

Reconstructs the Plassembler chromosomal-read-removal step: map long- and short-read sets to a Flye (or Raven) long-read-only assembly with minimap2, then use SAMtools to extract and combine reads mapping to non-chromosome (putative plasmid) contigs plus unmapped reads, so only chromosomal reads are removed before hybrid assembly.

## When to use

Use when a bacterial hybrid-assembly workflow needs to remove chromosomal reads from FASTQ inputs to speed up plasmid assembly, retaining unmapped reads and reads on putative plasmid contigs.

## When NOT to use

- Input is a long-read-only (kmer_mode) Plassembler run with no short-read set — the hybrid read-extraction design described here presumes both long- and short-read inputs
- The sample has no expected plasmids or the assembly consists of a single chromosomal contig — extraction would reduce to retaining unmapped reads only
- Reads are from eukaryotic or metagenomic samples, outside Plassembler's bacterial-isolate hybrid assembly scope

## Inputs

- Long-read FASTQ (e.g. Nanopore) input read set
- Paired short-read (Illumina) FASTQ read set
- Flye or Raven long-read-only draft assembly (FASTA) for contig classification
- Chromosome length threshold (-c) used to designate chromosomal vs non-chromosome contigs

## Outputs

- filtered_plasmid_reads.fastq.gz — combined plasmid-mapped plus unmapped long- and short-reads
- minimap2 alignment (SAM/BAM) of both read sets against the long-read-only assembly

## How to apply

First create a long-read-only assembly with Flye (--nano-hq mode) or optionally Raven, then classify contigs by length against a '-c' chromosome threshold to designate the largest/chromosomal contig(s) versus putative plasmid contigs. Map both the long-read FASTQ and paired short-read FASTQ sets to this assembly with minimap2 (>=2.11). Using SAMtools (>=0.15.0), extract reads that map to the non-chromosome (putative plasmid) contigs AND reads that are unmapped — the unmapped fraction is deliberately retained to recover possible plasmids the long-read assembly missed — and combine them into filtered_plasmid_reads.fastq.gz. The rationale (from the Plassembler paper) is that discarding only mapped chromosomal reads while keeping unmapped and plasmid-mapped reads yields increased accuracy and computational efficiency versus Unicycler's approach. Validate the resulting FASTQs with a validate_fastq-style check before downstream hybrid assembly with Unicycler.

## Related tools

- **Flye** (Creates the initial long-read-only assembly that read mapping is performed against) — https://github.com/mikolomogorov/Flye
- **minimap2** (Maps the long and short read sets to the long-read-only assembly (version >=2.11)) — https://github.com/lh3/minimap2
- **SAMtools** (Extracts reads mapping to non-chromosome (putative plasmid) contigs and unmapped reads (version >=0.15.0)) — https://github.com/samtools/samtools
- **Plassembler** (Implements the full chromosomal-read-removal module and writes the filtered read set) — https://github.com/gbouras13/plassembler
- **Raven** (Optional alternative (>=1.8) to Flye for the initial long-read-only assembly) — https://github.com/lbcb-sci/raven

## Examples

```
flye --nano-hq <input fastq> --out-dir <output directory> --threads <thread count>
```

## Evaluation signals

- Run a validate_fastq-style check on filtered_plasmid_reads.fastq.gz to confirm the output FASTQ is well-formed (sequence and quality string lengths match per read)
- Confirm the extracted read set contains reads aligned to the non-chromosome (putative plasmid) contigs as well as unmapped reads
- Confirm reads mapping to the designated chromosomal contig(s) are absent from the filtered output
- Both the long-read and paired short-read FASTQs were mapped (minimap2 >=2.11) before extraction, and SAMtools version >=0.15.0 was used
- Downstream hybrid assembly of the filtered reads recovers the small plasmids (e.g. the 1934 bp K. variicola INF345 plasmid) rather than losing them

## Limitations

- Flye --deterministic is not deterministic between thread counts, so the long-read-only assembly (and hence mapping-based extraction) can vary with thread count
- The chromosome/non-chromosome designation depends on the '-c' length threshold; misclassification would contaminate the filtered read set with chromosomal reads or drop plasmid reads
- Unmapped reads are intentionally kept to recover plasmids Flye missed, which means some residual chromosomal signal may pass through the filter

## Evidence

- [methods] Reads that map to the non-chromosome contigs (i.e. the putative plasmids), and reads that are unmapped (to recover possible plasmids Flye has missed) are extracted and combined.: "Reads that map to the non-chromosome contigs (i.e. the putative plasmids), and reads that are unmapped  (to recover possible plasmids Flye has missed) are extracted and combined."
- [methods] Following this, minimap2 (Li 2018) is used to map long and short read sets.: "Following this, minimap2 (Li 2018) is used to map long and short read sets."
- [abstract] It achieves increased accuracy and computational efficiency compared to the existing gold standard tool Unicycler by removing chromosomal reads from the input read sets using a mapping approach: "It achieves increased accuracy and computational efficiency compared to the existing gold standard tool Unicycler by removing chromosomal reads from the input read sets using a mapping approach"
- [methods] `plassembler` uses Flye (Kolmogorov et al 2019) or optionally Raven (Vaser and Šikić 2021) to create an initial long-read only assembly.: "`plassembler` uses Flye (Kolmogorov et al 2019) or optionally Raven (Vaser and Šikić 2021) to create an initial long-read only assembly."
