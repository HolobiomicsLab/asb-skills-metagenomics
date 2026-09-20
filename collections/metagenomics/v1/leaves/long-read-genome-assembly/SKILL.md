---
name: long-read-genome-assembly
description: Use when you have long reads (e.g. Nanopore FASTQ) plus optionally paired
  short reads from a bacterial isolate and need a preliminary long-read-only assembly
  to classify contigs and strip chromosomal reads before plasmid-focused hybrid assembly.
when_to_use_negative:
- When you only need a complete chromosome assembly (long-read-first plus polishing)
  and no plasmid recovery — Flye alone suffices without the read-removal step
- When no long reads are available and a short-read-only assembly (e.g. Unicycler
  Illumina mode) is required
- When long-read coverage is too sparse for Flye to produce a reliable initial assembly
  of the chromosome
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3168
tools:
- name: Flye
  role: Performs the initial long-read-only assembly in nanopore high-quality mode
    (>=2.9); Raven >=1.8 is the optional alternative assembler
  repo: https://github.com/mikolmogorov/Flye
- name: minimap2
  role: Maps both long- and short-read sets back to the Flye assembly so reads can
    be classified by contig of origin (>=2.11)
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Extracts reads mapping to non-chromosome contigs and unmapped reads, which
    are combined into the filtered plasmid read set (>=0.15.0)
  repo: https://github.com/samtools/samtools
- name: Plassembler
  role: Wraps the assembly-and-read-removal workflow, using the '-c' chromosome length
    threshold to classify contigs and emitting filtered_plasmid_reads.fastq.gz
  repo: https://github.com/gbouras13/plassembler
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:cb2f041859185537f3e29e746326aa5ee64f3667c9bddd4b8953f333522f506f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikolmogorov/Flye
  tools:
  - Flye
  - Plassembler
  - SAMtools
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# long-read-genome-assembly

## Summary

Produce a long-read-only genome assembly (as Plassembler does with Flye) and then use Minimap2/SAMtools to remove chromosomal reads, keeping unmapped reads plus reads on non-chromosome contigs as putative plasmid reads for downstream hybrid assembly.

## When to use

Use when you have long reads (e.g. Nanopore FASTQ) plus optionally paired short reads from a bacterial isolate and need a preliminary long-read-only assembly to classify contigs and strip chromosomal reads before plasmid-focused hybrid assembly.

## When NOT to use

- When you only need a complete chromosome assembly (long-read-first plus polishing) and no plasmid recovery — Flye alone suffices without the read-removal step
- When no long reads are available and a short-read-only assembly (e.g. Unicycler Illumina mode) is required
- When long-read coverage is too sparse for Flye to produce a reliable initial assembly of the chromosome

## Inputs

- Long-read FASTQ (e.g. Nanopore reads)
- Paired short-read FASTQ files (Illumina, optional for the assembly step but required for hybrid-read filtering)
- Chromosome length threshold (-c) for contig classification
- Thread count

## Outputs

- Flye long-read-only assembly (assembly_info/contigs FASTA) classified into chromosomal vs non-chromosome (putative plasmid) contigs
- filtered_plasmid_reads.fastq.gz containing combined reads mapping to non-chromosome contigs plus unmapped reads
- BAM/SAM alignments of long and short reads against the Flye assembly

## How to apply

Assemble the long reads with Flye in nanopore high-quality mode (`flye --nano-hq <input fastq> --out-dir <dir> --threads <n>`; Raven >=1.8 is an optional alternative, Flye >=2.9 is the tested dependency). Classify the resulting contigs by length against the chromosome threshold (Plassembler's '-c' parameter) to designate chromosomal versus non-chromosome (putative plasmid) contigs. Map both the long-read FASTQ and paired short-read FASTQs to the Flye assembly with Minimap2 (>=2.11). With SAMtools (>=0.15.0), extract reads mapping to non-chromosome contigs together with unmapped reads — the latter recover plasmids Flye may have missed — and combine them into filtered_plasmid_reads.fastq.gz. The rationale is that removing chromosomal reads before Unicycler hybrid assembly gives increased accuracy and computational efficiency versus assembling the full read set.

## Related tools

- **Flye** (Performs the initial long-read-only assembly in nanopore high-quality mode (>=2.9); Raven >=1.8 is the optional alternative assembler) — https://github.com/mikolmogorov/Flye
- **minimap2** (Maps both long- and short-read sets back to the Flye assembly so reads can be classified by contig of origin (>=2.11)) — https://github.com/lh3/minimap2
- **SAMtools** (Extracts reads mapping to non-chromosome contigs and unmapped reads, which are combined into the filtered plasmid read set (>=0.15.0)) — https://github.com/samtools/samtools
- **Plassembler** (Wraps the assembly-and-read-removal workflow, using the '-c' chromosome length threshold to classify contigs and emitting filtered_plasmid_reads.fastq.gz) — https://github.com/gbouras13/plassembler

## Examples

```
flye --nano-hq longreads.fastq --out-dir flye_out --threads 8
```

## Evaluation signals

- Flye completes and produces an assembly whose longest contig(s) exceed the '-c' chromosome length threshold, enabling correct chromosomal vs non-chromosome classification
- The filtered FASTQ passes a validate_fastq-style check: sequence and quality line lengths match for every read
- filtered_plasmid_reads.fastq.gz contains reads aligned to putative plasmid contigs plus unmapped reads, and chromosomally mapping reads are excluded
- Downstream hybrid assembly (Unicycler) succeeds on the extracted reads and recovers plasmids, including small ones (e.g. the 1934 bp K. variicola INF345 plasmid missed by Unicycler on full read sets)
- Minimap2 and SAMtools versions satisfy the minimums (>=2.11 and >=0.15.0) documented in the Plassembler methods

## Limitations

- Unmapped reads must be retained because the initial long-read-only assembly may miss plasmids entirely; filtering only by mapping would drop them
- Flye's '--deterministic' flag does not yield identical assemblies across different thread counts, so results can vary between runs
- Contig classification depends on the chromosome length threshold; small chromosomes or very large plasmids can be misclassified

## Evidence

- [methods] Long-read assembly step with Flye or Raven as the initial draft: "`plassembler` uses Flye (Kolmogorov et al 2019) or optionally Raven (Vaser and Šikić 2021) to create an initial long-read only assembly."
- [methods] Extraction of plasmid and unmapped reads after mapping: "Reads that map to the non-chromosome contigs (i.e. the putative plasmids), and reads that are unmapped  (to recover possible plasmids Flye has missed) are extracted and combined."
- [abstract] Rationale for chromosomal read removal before hybrid assembly: "It achieves increased accuracy and computational efficiency compared to the existing gold standard tool Unicycler by removing chromosomal reads from the input read sets"
- [methods] Minimap2 mapping of both read sets to the assembly: "Following this, minimap2 (Li 2018) is used to map long and short read sets."
- [other] Flye assembly command invoked in the reconstructed workflow: [evidence span withheld — anchor status: not_found]
