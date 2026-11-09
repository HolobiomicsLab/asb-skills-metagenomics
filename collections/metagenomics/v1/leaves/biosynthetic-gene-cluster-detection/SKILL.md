---
name: biosynthetic-gene-cluster-detection
description: Use when comparing metagenome assemblies by biologically meaningful content
  — counting BGCs (alongside Swiss-Prot CDSs, viral sequences, and CheckM/Prokka scaffold
  categories) recovered per assembler per sample.
when_to_use_negative:
- Input is raw reads rather than a scaffold/contig FASTA assembly — BGC detection
  in this skill is applied to cleaned assemblies, not reads
- Assemblies still contain human contaminant scaffolds — the workflow requires human-reference
  mapping and contaminant removal before BGC evaluation
- The analysis goal is assembly accuracy (mismatches, misassemblies) rather than recovery
  of biologically important elements such as gene clusters, viral sequences, or CDSs
edam_operation: http://edamontology.org/operation_0306
tools:
- name: MetaPlatanus
  role: Hybrid assembler producing the scaffolds on which BGC detection is evaluated;
    combines long-range links, species-specific sequence compositions, coverage depth,
    and binning
  repo: https://github.com/rkajitani/MetaPlatanus
- name: metaFlye
  role: Comparator long-read assembler run with its default polishing module to produce
    scaffolds for BGC recovery comparison
- name: OPERA-MS
  role: Comparator hybrid assembler for the short-read + long-read benchmark case
- name: MEGAHIT
  role: Comparator short-read assembler; also used internally by MetaPlatanus to assemble
    initial contigs
  repo: https://github.com/voutcn/megahit
- name: Prokka
  role: Annotation tool used in the parallel evaluation to count Swiss-Prot coding
    sequences on the same assemblies
- name: CheckM
  role: Used with Prokka to categorize scaffolds as over-half or near-complete bacterial
    genomes in the same per-assembler evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:03d2c5af8885acbfa835795214d23a4e715848a4652dbc16c78a59335f61451f
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
  - BWA
  - CheckM
  - MEGAHIT
  - MetaBAT2
  - MetaPlatanus
  - NextPolish
  - OPERA-MS
  - Prokka
  - Racon
  - SAMtools
  - SeqKit
  - TGS-GapCloser
  - metaFlye
  - minimap2
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# biosynthetic-gene-cluster-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects biosynthetic gene clusters (BGCs) in metagenome-assembled scaffolds so that different hybrid metagenome assemblers (e.g. MetaPlatanus, metaFlye, OPERA-MS, MEGAHIT) can be compared on how many biologically important gene-cluster elements they recover from human gut nanopore samples.

## When to use

Use when comparing metagenome assemblies by biologically meaningful content — counting BGCs (alongside Swiss-Prot CDSs, viral sequences, and CheckM/Prokka scaffold categories) recovered per assembler per sample.

## When NOT to use

- Input is raw reads rather than a scaffold/contig FASTA assembly — BGC detection in this skill is applied to cleaned assemblies, not reads
- Assemblies still contain human contaminant scaffolds — the workflow requires human-reference mapping and contaminant removal before BGC evaluation
- The analysis goal is assembly accuracy (mismatches, misassemblies) rather than recovery of biologically important elements such as gene clusters, viral sequences, or CDSs

## Inputs

- Cleaned scaffold FASTA files (human-contaminant scaffolds removed) from each assembler per human gut nanopore sample
- Hybrid input reads: Illumina paired-end FASTQ (R1/R2) and ONT FASTQ (>1 Gbp per sample) from the SRA dataset of reference 22

## Outputs

- Per-sample, per-assembler counts of biosynthetic gene clusters
- Summary tables compiling BGC counts for reproducing Figure 2 and Supplementary Tables S15-S17

## How to apply

First generate per-sample scaffold FASTA assemblies: run MetaPlatanus in hybrid mode on Illumina paired-end plus ONT reads ('metaplatanus -IP1 R1.fq R2.fq -ont ONT.fq') with the default k-mer coverage cutoff for contig assembly (4 with MEGAHIT, 2 otherwise) and default 97% minimum mapping identity for binning, alongside comparator assemblies from metaFlye (with its default polishing module), MEGAHIT, and OPERA-MS. Map reads against the human reference and remove human contaminant scaffolds before evaluation. Then detect biosynthetic gene clusters on each cleaned assembly, alongside counting Swiss-Prot coding sequences with Prokka and categorizing scaffolds with CheckM/Prokka. Rationale: the MetaPlatanus study showed BGC recovery discriminates assemblers on long-read metagenome data, with MetaPlatanus achieving the largest number of top cases for all element criteria in the 18-sample human gut nanopore benchmark.

## Related tools

- **MetaPlatanus** (Hybrid assembler producing the scaffolds on which BGC detection is evaluated; combines long-range links, species-specific sequence compositions, coverage depth, and binning) — https://github.com/rkajitani/MetaPlatanus
- **metaFlye** (Comparator long-read assembler run with its default polishing module to produce scaffolds for BGC recovery comparison)
- **OPERA-MS** (Comparator hybrid assembler for the short-read + long-read benchmark case)
- **MEGAHIT** (Comparator short-read assembler; also used internally by MetaPlatanus to assemble initial contigs) — https://github.com/voutcn/megahit
- **Prokka** (Annotation tool used in the parallel evaluation to count Swiss-Prot coding sequences on the same assemblies)
- **CheckM** (Used with Prokka to categorize scaffolds as over-half or near-complete bacterial genomes in the same per-assembler evaluation)

## Evaluation signals

- Per-assembler BGC counts must match the values reported in Figure 2 and Supplementary Tables S15-S17 for the 18 human gut samples
- Every assembler run must produce a valid final scaffold FASTA per sample before BGC evaluation
- MetaPlatanus is expected to have the largest number of top cases for all element criteria, exceeding other tools by at least seven cases
- Human contaminant scaffolds must be absent from assemblies (confirmed via mapping against the human reference) before detection
- BGC counts should be consistent with parallel element metrics (Swiss-Prot CDS counts via Prokka, viral sequence counts, CheckM/Prokka scaffold categories) compiled per assembler

## Limitations

- The specific BGC-detection software and its version are not reported in the article context; no tool versions are given for the compared assemblers either
- Key supporting evidence (Supplementary Tables S17, S24, S25; Figure 6) is deferred to supplementary materials not included here
- No dataset accessions for the human gut benchmark data are stated in the article discussion; only mock-sample reads (PRJDB4377) and FigShare assemblies (doi:10.6084/m9.figshare.15090603.v1) are deposited

## Evidence
