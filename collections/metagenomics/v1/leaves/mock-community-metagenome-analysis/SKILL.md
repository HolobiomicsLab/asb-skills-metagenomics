---
name: mock-community-metagenome-analysis
description: Use when Illumina paired-end plus 4 kbp/8 kbp mate-pair reads from a
  known-composition mock metagenome (e.g. DDBJ PRJDB4377) must be assembled with several
  assemblers and compared by NGA50 contiguity.
when_to_use_negative:
- The dataset is real (unknown-composition) metagenomic data with no ground-truth
  reference genomes, so NGA50 cannot be computed and the mock-benchmark validation
  is impossible
- Only short-read paired-end data are available without mate-pair or long-read libraries,
  since the benchmark targets MetaPlatanus's long-range mate-pair scaffolding advantage
- Reference-based or hybrid long-read (PacBio/ONT) assembly is the goal — this study
  notes reference information has not been well examined and its MP benchmark is mate-pair
  based
edam_operation: http://edamontology.org/operation_0523
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MetaPlatanus
  role: Primary assembler under benchmark; run with -IP1 paired-end and -OP mate-pair
    libraries, -t threads and -m memory, producing out_final.fa
  repo: https://github.com/rkajitani/MetaPlatanus
- name: MEGAHIT
  role: Competitor assembler on paired-end reads; also the initial-contig assembler
    combined inside MetaPlatanus (v1.2.3)
  repo: https://github.com/voutcn/megahit
- name: IDBA-UD
  role: Competitor assembler run on the paired-end reads of each mock case
- name: SPAdes
  role: Competitor assembler run on the paired-end reads of each mock case
- name: metaSPAdes
  role: Competitor metagenome assembler run on the paired-end reads of each mock case
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:9dda1cc65e19d4fb513b04a4b5dc77f5626de7f5ade3bffd0347d3f42db475e8
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
  edam_operation: http://edamontology.org/operation_0523
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mock-community-metagenome-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks de novo metagenome assemblers (MetaPlatanus vs IDBA-UD, SPAdes, metaSPAdes, MEGAHIT) on mock human gut bacterial communities sequenced with Illumina paired-end and mate-pair libraries, using NGA50 as the primary contiguity metric to show MetaPlatanus attains the largest NGA50s.

## When to use

Use when Illumina paired-end plus 4 kbp/8 kbp mate-pair reads from a known-composition mock metagenome (e.g. DDBJ PRJDB4377) must be assembled with several assemblers and compared by NGA50 contiguity.

## When NOT to use

- The dataset is real (unknown-composition) metagenomic data with no ground-truth reference genomes, so NGA50 cannot be computed and the mock-benchmark validation is impossible
- Only short-read paired-end data are available without mate-pair or long-read libraries, since the benchmark targets MetaPlatanus's long-range mate-pair scaffolding advantage
- Reference-based or hybrid long-read (PacBio/ONT) assembly is the goal — this study notes reference information has not been well examined and its MP benchmark is mate-pair based

## Inputs

- Illumina HiSeq2500/MiSeq paired-end FASTQ reads for MP mock cases 1-3 (DDBJ BioProject PRJDB4377)
- 4 kbp mate-pair (jumping library) FASTQ reads per mock case
- 8 kbp mate-pair (jumping library) FASTQ reads per mock case
- Reference genomes of the 20 known human gut bacteria for NGA50 computation

## Outputs

- MetaPlatanus assembly FASTA (out_final.fa) per mock case
- Competitor assembly contig/scaffold FASTA files from IDBA-UD, SPAdes, metaSPAdes and MEGAHIT per case
- NGA50 comparison table (nga50_comparison.tsv) mirroring Supplementary Table S14
- Bar chart of NGA50 per assembler per case analogous to Supplementary Figure S5

## How to apply

Download the raw MP mock reads (HiSeq2500/MiSeq paired-end, 4 kbp mate-pair, 8 kbp mate-pair) from DDBJ BioProject PRJDB4377 and organize them per mock case (cases 1–3, 20 human gut bacteria). Run MetaPlatanus per case, supplying the paired-end library with -IP1 and the 4 kbp/8 kbp mate-pair libraries with -OP options, setting -t and -m to available resources (e.g. -t 8 -m 64); internally it executes contig-assembly, scaffolding, gap-closing and binning automatically and uses MEGAHIT for initial contigs. Run IDBA-UD, SPAdes, metaSPAdes and MEGAHIT on the paired-end reads of each case, adding mate-pair libraries where each tool supports them. Evaluate every assembly (MetaPlatanus out_final.fa and each competitor's contig/scaffold FASTA) to compute NGA50 and related contiguity metrics, assemble the per-case NGA50 values into a comparison table mirroring Supplementary Table S14, and plot per-assembler bar charts analogous to Supplementary Figure S5. Validate that the reproduced ranking matches the reported result: MetaPlatanus attains the largest NGA50 for all three mock cases.

## Related tools

- **MetaPlatanus** (Primary assembler under benchmark; run with -IP1 paired-end and -OP mate-pair libraries, -t threads and -m memory, producing out_final.fa) — https://github.com/rkajitani/MetaPlatanus
- **MEGAHIT** (Competitor assembler on paired-end reads; also the initial-contig assembler combined inside MetaPlatanus (v1.2.3)) — https://github.com/voutcn/megahit
- **IDBA-UD** (Competitor assembler run on the paired-end reads of each mock case)
- **SPAdes** (Competitor assembler run on the paired-end reads of each mock case)
- **metaSPAdes** (Competitor metagenome assembler run on the paired-end reads of each mock case)

## Examples

```
metaplatanus -IP1 case1_PE_R1.fq -IP2 case1_PE_R2.fq -OP1 case1_MP4k_R1.fq -OP2 case1_MP4k_R2.fq -t 8 -m 64
```

## Evaluation signals

- MetaPlatanus out_final.fa exists for each of the three mock cases and is a valid FASTA
- Every assembler/case combination yields a computed NGA50 value, and all values are tabulated (nga50_comparison.tsv mirrors Supplementary Table S14)
- Reproduced ranking shows MetaPlatanus with the largest NGA50 for mock cases 1-3, matching Supplementary Table S14 and Supplementary Figure S5
- MetaPlatanus runs used the paired-end library via -IP1 and the 4 kbp/8 kbp mate-pair libraries via -OP options (e.g. -t 8 -m 64)
- Related contiguity metrics (e.g. number of contigs/scaffolds, misassembly counts) are reported alongside NGA50 for each assembly

## Limitations

- Tool versions for compared assemblers (metaFlye, OPERA-MS, MEGAHIT, etc.) are not reported in the article discussion
- The study focuses on de novo methods; reference-based approaches are not well examined
- Interspecies misassemblies remain the key risk when scaffolding multi-species mock communities; MetaPlatanus mitigates but does not eliminate them via coverage depth, k-mer frequencies and binning

## Evidence
