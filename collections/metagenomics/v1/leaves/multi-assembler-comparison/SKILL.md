---
name: multi-assembler-comparison
description: Use when the goal is to reproduce or extend a contiguity benchmark of
  metagenome assemblers on mock-community reads (Illumina paired-end plus 4 kbp/8
  kbp mate-pair), reporting NGA50 per tool per sample.
when_to_use_negative:
- The datasets are real (unmocked) environmental metagenomes with no reference genomes,
  so NGA50 against known bacteria cannot be computed
- Only one assembler is of interest and no head-to-head contiguity comparison is intended
- Input libraries lack long-range links (mate-pair or long reads), since MetaPlatanus
  requires at least one short-read paired-end library and its advantage lies in using
  long-range links
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MetaPlatanus
  role: Target hybrid assembler run with -IP1 for paired-end and -OP for 4 kbp/8 kbp
    mate-pair libraries; output out_final.fa is benchmarked
  repo: https://github.com/rkajitani/MetaPlatanus
- name: MEGAHIT
  role: Competing short-read metagenome assembler run on the paired-end reads of each
    mock case
  repo: https://github.com/voutcn/megahit
- name: IDBA-UD
  role: Competing assembler run on the paired-end reads of each mock case
- name: SPAdes
  role: Competing assembler run on the paired-end reads of each mock case
- name: metaSPAdes
  role: Competing metagenome assembler run on the paired-end reads of each mock case
- name: minimap2
  role: Aligner used in the MetaPlatanus toolchain for read mapping steps
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:908b0b0585e86c19ad44244f2c73abdc31624af77121fd79787df5f98ddafef4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/rkajitani/MetaPlatanus
  repo_url: https://github.com/rkajitani/MetaPlatanus
  tools:
  - IDBA-UD
  - MEGAHIT
  - MetaPlatanus
  - SPAdes
  - metaSPAdes
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multi-assembler-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks a target metagenome assembler (e.g. MetaPlatanus) against competing assemblers (IDBA-UD, SPAdes, metaSPAdes, MEGAHIT) on the same mock-community read sets and compares assembly contiguity using NGA50, mirroring Supplementary Table S14 and Figure S5 of the MetaPlatanus paper.

## When to use

Use when the goal is to reproduce or extend a contiguity benchmark of metagenome assemblers on mock-community reads (Illumina paired-end plus 4 kbp/8 kbp mate-pair), reporting NGA50 per tool per sample.

## When NOT to use

- The datasets are real (unmocked) environmental metagenomes with no reference genomes, so NGA50 against known bacteria cannot be computed
- Only one assembler is of interest and no head-to-head contiguity comparison is intended
- Input libraries lack long-range links (mate-pair or long reads), since MetaPlatanus requires at least one short-read paired-end library and its advantage lies in using long-range links

## Inputs

- Illumina HiSeq2500/MiSeq paired-end FASTQ reads from MP mock cases (DDBJ BioProject PRJDB4377)
- 4 kbp mate-pair library FASTQ reads per mock case
- 8 kbp mate-pair library FASTQ reads per mock case
- Reference genomes of the 20 known human gut bacteria for NGA50 computation

## Outputs

- MetaPlatanus out_final.fa and competitor contig/scaffold FASTA files per mock case
- Per-case NGA50 comparison table (nga50_comparison.tsv) mirroring Supplementary Table S14
- Bar chart of NGA50 per assembler per mock case analogous to Supplementary Figure S5

## How to apply

Download the MP mock raw reads (HiSeq2500/MiSeq paired-end, 4 kbp and 8 kbp mate-pair libraries) for the 20 human gut bacteria from DDBJ BioProject PRJDB4377 and organize them per mock case. Run MetaPlatanus supplying the paired-end library with -IP1 and the 4 kbp/8 kbp mate-pair libraries with -OP, setting -t and -m to available resources (e.g. -t 8 -m 64). Run IDBA-UD, SPAdes, metaSPAdes and MEGAHIT on the same paired-end reads, adding mate-pair libraries where each tool supports them. Evaluate every output FASTA (MetaPlatanus out_final.fa plus each competitor's contig/scaffold file) to compute NGA50 and related contiguity metrics against the mock reference, then tabulate per-case values (nga50_comparison.tsv mirroring Supplementary Table S14) and plot a bar chart analogous to Supplementary Figure S5. Validate that the reproduced ranking matches the reported result: MetaPlatanus attains the largest NGA50 for mock cases 1-3.

## Related tools

- **MetaPlatanus** (Target hybrid assembler run with -IP1 for paired-end and -OP for 4 kbp/8 kbp mate-pair libraries; output out_final.fa is benchmarked) — https://github.com/rkajitani/MetaPlatanus
- **MEGAHIT** (Competing short-read metagenome assembler run on the paired-end reads of each mock case) — https://github.com/voutcn/megahit
- **IDBA-UD** (Competing assembler run on the paired-end reads of each mock case)
- **SPAdes** (Competing assembler run on the paired-end reads of each mock case)
- **metaSPAdes** (Competing metagenome assembler run on the paired-end reads of each mock case)
- **minimap2** (Aligner used in the MetaPlatanus toolchain for read mapping steps) — https://github.com/lh3/minimap2

## Examples

```
metaplatanus m -IP1 pe_1.fq pe_2.fq -OP mp4k_1.fq mp4k_2.fq mp8k_1.fq mp8k_2.fq -o out -t 8 -m 64
```

## Evaluation signals

- NGA50 for MetaPlatanus is the largest among MetaPlatanus, IDBA-UD, SPAdes, metaSPAdes and MEGAHIT for mock cases 1-3, matching Supplementary Table S14
- The reproduced nga50_comparison.tsv contains one NGA50 value per assembler per mock case with all five tools represented in every case
- Every assembly evaluated is a valid FASTA with contiguity metrics computable against the mock reference (no empty or failed outputs)
- The per-assembler bar chart is consistent with the tabulated NGA50 ranking, analogous to Supplementary Figure S5
- Interspecies misassembly counts remain few for the MetaPlatanus assemblies, consistent with the paper's finding of high contiguity with few interspecies misassemblies

## Limitations

- Tool versions of the compared assemblers (metaFlye, OPERA-MS, MEGAHIT, etc.) are not reported in the discussion, so exact reproducibility of versions is not guaranteed
- Key quantitative evidence (Supplementary Tables S14, S17, S24, S25; Figure 6 / Figure S5) resides in supplementary materials rather than the main text
- The study focuses on de novo methods; reference-based assembly approaches were not well examined
- metaFlye comparisons require additional short-read sequencing and polishing for rigorous downstream analysis

## Evidence

- [results] In the MP mock mate-pair benchmark, MetaPlatanus had the largest NGA50s for all cases (Supplementary Table S14 and Supplementary Figure S5): [evidence span withheld — anchor status: not_found]
