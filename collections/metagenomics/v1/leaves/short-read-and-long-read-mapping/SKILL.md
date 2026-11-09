---
name: short-read-and-long-read-mapping
description: Use when you need to partition bacterial long- and short-read FASTQs
  against a draft assembly so only chromosomal reads are discarded and plasmid-derived
  reads (mapped to non-chromosome contigs or unmapped) are retained for downstream
  assembly.
when_to_use_negative:
- Input is already an assembled genome or plasmid FASTA rather than raw read sets
  — no mapping-based read filtering applies
- Only short reads are available with no long-read assembly to map against (Plassembler's
  chromosomal-read removal requires the Flye/Raven long-read-only assembly)
- Reads come from a non-bacterial or multi-isolate metagenome, where a single chromosomal
  contig classification against the '-c' threshold is not meaningful
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
tools:
- name: Flye
  role: Produces the initial long-read-only assembly (>=2.9; e.g. --nano-hq mode)
    that serves as the mapping reference
  repo: https://github.com/fenderglass/Flye
- name: minimap2
  role: Maps both long and short read sets to the Flye assembly (>=2.11)
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Extracts reads mapping to non-chromosome contigs and unmapped reads from the
    alignments, and combines them (>=0.15.0)
  repo: https://github.com/samtools/samtools
- name: Plassembler
  role: Orchestrates the chromosomal-read-removal module and emits filtered_plasmid_reads.fastq.gz
  repo: https://github.com/gbouras13/plassembler
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:c61592e4154766dc69a76089f5e906ba704dce741fb8e73b00956d42f05604bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fenderglass/Flye
  tools:
  - Flye
  - Plassembler
  - SAMtools
  - minimap2
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# short-read-and-long-read-mapping

## Summary

Maps both long-read and paired short-read sets onto a long-read-only assembly (Flye draft) with minimap2, then uses SAMtools to keep reads on non-chromosome (putative plasmid) contigs plus unmapped reads, removing only chromosomal reads before hybrid assembly.

## When to use

Use when you need to partition bacterial long- and short-read FASTQs against a draft assembly so only chromosomal reads are discarded and plasmid-derived reads (mapped to non-chromosome contigs or unmapped) are retained for downstream assembly.

## When NOT to use

- Input is already an assembled genome or plasmid FASTA rather than raw read sets — no mapping-based read filtering applies
- Only short reads are available with no long-read assembly to map against (Plassembler's chromosomal-read removal requires the Flye/Raven long-read-only assembly)
- Reads come from a non-bacterial or multi-isolate metagenome, where a single chromosomal contig classification against the '-c' threshold is not meaningful

## Inputs

- Long-read FASTQ (e.g. Nanopore) from the bacterial isolate
- Paired short-read FASTQ files (Illumina)
- Flye long-read-only assembly (contigs FASTA)
- Chromosome length threshold (-c) for classifying contigs as chromosomal vs putative plasmid

## Outputs

- filtered_plasmid_reads.fastq.gz — combined long- and short-read set containing reads mapping to non-chromosome contigs plus unmapped reads
- SAM/BAM alignments of long and short reads against the Flye assembly
- Per-read FASTQ validation results (sequence/quality length consistency)

## How to apply

First assemble long reads with Flye (e.g. 'flye --nano-hq <input fastq> --out-dir <dir> --threads <n>') and classify contigs by length against the '-c' chromosome threshold to designate the chromosomal contig versus putative plasmid contigs. Map both the long-read FASTQ and the paired short-read FASTQs to this Flye assembly using minimap2 (>=2.11). With SAMtools (>=0.15.0), extract two read classes per read set: reads aligned to the non-chromosome contigs (putative plasmids) and reads that are unmapped — the latter recovers possible plasmids Flye missed. Combine the extracted read sets and write them to filtered_plasmid_reads.fastq.gz for the subsequent Unicycler hybrid assembly; validate FASTQ integrity (sequence and quality length match per read) before proceeding.

## Related tools

- **Flye** (Produces the initial long-read-only assembly (>=2.9; e.g. --nano-hq mode) that serves as the mapping reference) — https://github.com/fenderglass/Flye
- **minimap2** (Maps both long and short read sets to the Flye assembly (>=2.11)) — https://github.com/lh3/minimap2
- **SAMtools** (Extracts reads mapping to non-chromosome contigs and unmapped reads from the alignments, and combines them (>=0.15.0)) — https://github.com/samtools/samtools
- **Plassembler** (Orchestrates the chromosomal-read-removal module and emits filtered_plasmid_reads.fastq.gz) — https://github.com/gbouras13/plassembler

## Examples

```
flye --nano-hq long_reads.fastq --out-dir flye_out --threads 16 && minimap2 -ax flye_out/assembly.fasta long_reads.fastq R1.fastq R2.fastq | samtools view -b - > mapped.bam; samtools view -b mapped.bam nonchrom_contigs.txt > plasmid.bam; samtools view -b -f 4 mapped.bam > unmapped.bam
```

## Evaluation signals

- filtered_plasmid_reads.fastq.gz passes a validate_fastq-style check: sequence length equals quality length for every read
- Retained reads include alignments to non-chromosome (putative plasmid) contigs AND a substantial fraction of unmapped reads, not solely mapped reads
- Reads mapping to the contig(s) designated chromosomal by the '-c' length threshold are absent from the filtered output
- Both the long-read and short-read sets contribute to the combined filtered set (hybrid assembly precondition)
- Chromosome-scale contig identification is stable: re-running contig length classification yields the same chromosomal vs plasmid split (note Flye --deterministic is not fully deterministic across thread counts)

## Limitations

- Flye assemblies are not perfectly reproducible: 'even Flye using --deterministic isn't deterministic between threads either', so the mapped/unmapped read split can vary between runs with different thread counts
- Extracting unmapped reads is a heuristic to recover plasmids Flye missed; it can retain chromosomal-adjacent or contaminant reads that failed to map
- Only one tool in this module (minimap2 >=2.11, SAMtools >=0.15.0) — older versions may not support the needed flags/behaviours

## Evidence

- [methods] minimap2 maps long and short read sets: "Following this, minimap2 (Li 2018) is used to map long and short read sets."
- [methods] extraction of plasmid-mapped and unmapped reads: "Reads that map to the non-chromosome contigs (i.e. the putative plasmids), and reads that are unmapped  (to recover possible plasmids Flye has missed) are extracted and combined."
- [abstract] accuracy gain via mapping-based chromosomal read removal: "It achieves increased accuracy and computational efficiency compared to the existing gold standard tool Unicycler by removing chromosomal reads from the input read sets using a mapping approach"
- [methods] Flye non-determinism across thread counts: "Interestingly, I found out that even Flye using `--deterministic` isn't deterministic between threads either"
- [other] workflow step: classify contigs and extract reads with SAMtools: [evidence span withheld — anchor status: not_found]
