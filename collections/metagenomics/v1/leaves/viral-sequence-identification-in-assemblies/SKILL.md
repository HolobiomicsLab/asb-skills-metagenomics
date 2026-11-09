---
name: viral-sequence-identification-in-assemblies
description: Use when benchmarking metagenome assemblies — e.g. 18 human gut nanopore
  samples assembled with hybrid ONT + Illumina paired-end tools — for near-complete
  viral sequence recovery as a biological evaluation criterion.
when_to_use_negative:
- Input is raw reads rather than cleaned metagenome assembly scaffolds — viral identification
  is an assembly evaluation step, not a read-level step
- The analysis goal is reference-based characterization; this study focuses on de
  novo methods and reference information was not well examined
- Assemblies have not had human contaminant scaffolds removed, since evaluation must
  be performed on cleaned assemblies
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: MetaPlatanus
  role: Hybrid assembler producing the scaffolds evaluated for near-complete viral
    sequences; uses long-range ONT links, coverage depth, and binning to extend scaffolds
  repo: https://github.com/rkajitani/MetaPlatanus
- name: metaFlye
  role: Comparison long-read assembler run per sample with its default polishing module
    so viral recovery can be compared across tools
- name: MEGAHIT
  role: Short-read comparison assembler; also combined to assemble initial contigs
    in the benchmark pipeline
  repo: https://github.com/voutcn/megahit
- name: OPERA-MS
  role: Hybrid comparison assembler (binning-like clustering of contig connections)
    run on the human gut samples
- name: minimap2
  role: Read mapping against the human reference so human contaminant scaffolds can
    be removed before viral sequence evaluation
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:93ca58ca03d7668152cf0a69f7d0548046ee88cef27d919d22f715507508db1f
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
  - MEGAHIT
  - MetaBAT2
  - MetaPlatanus
  - NextPolish
  - OPERA-MS
  - Racon
  - SAMtools
  - SeqKit
  - TGS-GapCloser
  - metaFlye
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# viral-sequence-identification-in-assemblies

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies near-complete viral sequences within metagenome assembly scaffolds to quantify how well different assemblers (MetaPlatanus, metaFlye, MEGAHIT, OPERA-MS) recover biologically important elements from human gut nanopore data.

## When to use

Use when benchmarking metagenome assemblies — e.g. 18 human gut nanopore samples assembled with hybrid ONT + Illumina paired-end tools — for near-complete viral sequence recovery as a biological evaluation criterion.

## When NOT to use

- Input is raw reads rather than cleaned metagenome assembly scaffolds — viral identification is an assembly evaluation step, not a read-level step
- The analysis goal is reference-based characterization; this study focuses on de novo methods and reference information was not well examined
- Assemblies have not had human contaminant scaffolds removed, since evaluation must be performed on cleaned assemblies

## Inputs

- ONT long-read FASTQ files (>1 Gbp per sample) from the 18 human gut nanopore samples (SRA dataset of reference 22)
- Mandatory Illumina paired-end FASTQ files (R1/R2) for the same samples
- Cleaned (human-contaminant-removed) final scaffold FASTA per assembler per sample
- Human reference genome for contaminant read mapping

## Outputs

- Per-assembler, per-sample counts of near-complete viral sequences
- Summary tables reproducing Figure 2 and Supplementary Tables S15-S17
- Valid final scaffold FASTA per assembler per sample

## How to apply

Run each assembler per sample (MetaPlatanus hybrid mode e.g. 'metaplatanus -IP1 R1.fq R2.fq -ont ONT.fq' with default -min_cov_contig of 4 with MEGAHIT and 2 otherwise, and default 97% -min_map_idt_binning; metaFlye with its default polishing module; MEGAHIT for short reads; OPERA-MS for the hybrid case). Before any evaluation, map reads against the human reference and remove human contaminant scaffolds from every assembly. Then evaluate each cleaned assembly, identifying near-complete viral sequences alongside Nx-length plot areas, Swiss-Prot CDS counts via Prokka, biosynthetic gene clusters, and over-half/near-complete bacterial scaffolds categorized with CheckM/Prokka. Compile per-assembler, per-sample viral counts into summary tables reproducing Figure 2 and Supplementary Tables S15-S17; in the published benchmark MetaPlatanus had the largest number of top cases for all element criteria, exceeding other tools by at least seven cases.

## Related tools

- **MetaPlatanus** (Hybrid assembler producing the scaffolds evaluated for near-complete viral sequences; uses long-range ONT links, coverage depth, and binning to extend scaffolds) — https://github.com/rkajitani/MetaPlatanus
- **metaFlye** (Comparison long-read assembler run per sample with its default polishing module so viral recovery can be compared across tools)
- **MEGAHIT** (Short-read comparison assembler; also combined to assemble initial contigs in the benchmark pipeline) — https://github.com/voutcn/megahit
- **OPERA-MS** (Hybrid comparison assembler (binning-like clustering of contig connections) run on the human gut samples)
- **minimap2** (Read mapping against the human reference so human contaminant scaffolds can be removed before viral sequence evaluation) — https://github.com/lh3/minimap2

## Examples

```
metaplatanus -IP1 R1.fq R2.fq -ont ONT.fq -t 32 -m 200
```

## Evaluation signals

- Per-assembler, per-sample near-complete viral counts must match the values reported in Figure 2 and Supplementary Tables S15-S17
- Each assembler must produce a valid final scaffold FASTA per sample before evaluation
- Every assembly must be cleaned of human contaminant scaffolds (via mapping against the human reference) prior to counting viral sequences
- Cross-assembler comparison: MetaPlatanus is expected to hold the largest number of top cases for all element criteria, exceeding other tools by at least seven cases
- Viral counts should be reported alongside the other element criteria (Swiss-Prot CDSs, BGCs, over-half and near-complete scaffolds) for a complete benchmark

## Limitations

- Key supporting evidence (e.g. Supplementary Tables S15-S17, S24, S25; Figure 6) is deferred to supplementary materials not present in the main text
- No tool versions were reported for the compared assemblers (metaFlye, OPERA-MS, MEGAHIT, etc.), affecting reproducibility
- metaFlye requires additional short-read sequencing and polishing for rigorous downstream analysis
- Reference-based methods were not well examined in this study, so viral identification results apply to de novo assemblies only

## Evidence
