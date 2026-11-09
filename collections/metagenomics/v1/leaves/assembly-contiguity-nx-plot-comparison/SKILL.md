---
name: assembly-contiguity-nx-plot-comparison
description: Use when multiple metagenome assemblers have been run on matched ONT
  long-read + Illumina paired-end samples and you need to quantify contiguity differences
  via Nx-length plots and scaffold categories.
when_to_use_negative:
- When the goal is base-level accuracy or interspecies-misasassembly assessment rather
  than contiguity — this skill compares Nx plots and scaffold-length categories, not
  nucleotide correctness.
- When a single assembly is available with no matched comparison assemblers run on
  the same samples, since the Nx-plot comparison requires per-assembler scaffold FASTAs.
- When reads lack a paired-end short-read library, because MetaPlatanus requires at
  least one short-read paired-end library and the comparison is defined over hybrid/short-read
  assembler outputs.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: MetaPlatanus
  role: Hybrid assembler under comparison; produces out_final.fa scaffolds and bin
    files used for Nx-plot and scaffold-category evaluation
  repo: https://github.com/rkajitani/MetaPlatanus
- name: MEGAHIT
  role: Short-read-only comparison arm; also the contig-assembly engine inside MetaPlatanus
    (k-mer coverage cutoff 4)
  repo: https://github.com/voutcn/megahit
- name: metaFlye
  role: Long-read assembler comparison arm, executed with its default polishing module
- name: OPERA-MS
  role: Hybrid assembler comparison arm with a binning-like clustering step utilizing
    contig connections
- name: minimap2
  role: Read mapping against the human reference to remove contaminant scaffolds from
    every assembly before Nx evaluation
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:5d38276a3003eea145216042c59704d821b13a95f85b6990aa699282f698ff08
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
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# assembly-contiguity-nx-plot-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking procedure for metagenome assemblers: the same ONT long-read plus Illumina paired-end samples are assembled with MetaPlatanus, MEGAHIT, metaFlye and OPERA-MS, and contiguity is quantified with Nx-length plots, Nx plot areas, and over-half/near-complete scaffold counts so the dominant assembler can be identified.

## When to use

Use when multiple metagenome assemblers have been run on matched ONT long-read + Illumina paired-end samples and you need to quantify contiguity differences via Nx-length plots and scaffold categories.

## When NOT to use

- When the goal is base-level accuracy or interspecies-misasassembly assessment rather than contiguity — this skill compares Nx plots and scaffold-length categories, not nucleotide correctness.
- When a single assembly is available with no matched comparison assemblers run on the same samples, since the Nx-plot comparison requires per-assembler scaffold FASTAs.
- When reads lack a paired-end short-read library, because MetaPlatanus requires at least one short-read paired-end library and the comparison is defined over hybrid/short-read assembler outputs.

## Inputs

- ONT long-read FASTQ files (e.g. 28-75 Gbp PromethION reads per saliva sample, >1 Gbp per gut sample)
- Illumina paired-end FASTQ files (mandatory paired-end library for MetaPlatanus)
- Final scaffold FASTA from each assembler per sample (e.g. MetaPlatanus out_final.fa)
- MetaPlatanus bin assignment files (out_finalClusters, out_finalClusters.tsv)
- Human reference genome for contaminant-scaffold removal

## Outputs

- Nx-length contiguity plots comparing all assemblers per dataset
- Nx plot areas per assembler and per sample
- CheckM/Prokka scaffold category counts (over-half and near-complete bacterial genomes) per assembler
- Summary tables reproducing Figure 2 and Supplementary Tables S15-S17, S22, S24, S25

## How to apply

Run each assembler per sample on the same inputs: MetaPlatanus in hybrid mode (metaplatanus -IP1 PE_1.fq PE_2.fq -ont ONT.fq), MEGAHIT for short reads only, metaFlye with its default polishing module, and OPERA-MS for the hybrid case; MetaPlatanus automatically executes contig-assembly, scaffolding, gap-closing and binning and emits out_final.fa plus bin files (out_finalClusters, out_finalClusters.tsv). Collect each assembler's final scaffold FASTA, map reads against the human reference and remove human contaminant scaffolds from every assembly before evaluation. Compute Nx-length plots and Nx plot areas for each cleaned assembly, and categorize scaffolds as over-half or near-complete bacterial genomes with CheckM/Prokka. Tabulate per-assembler, per-sample results and verify against the reported dominance of MetaPlatanus contiguity (Figures 2, 4-6; Supplementary Tables S15-S17, S22, S24, S25). The rationale is that contiguity alone is meaningful only if misassemblies are controlled, which MetaPlatanus achieves using long-range links, species-specific sequence compositions, coverage depth and binning.

## Related tools

- **MetaPlatanus** (Hybrid assembler under comparison; produces out_final.fa scaffolds and bin files used for Nx-plot and scaffold-category evaluation) — https://github.com/rkajitani/MetaPlatanus
- **MEGAHIT** (Short-read-only comparison arm; also the contig-assembly engine inside MetaPlatanus (k-mer coverage cutoff 4)) — https://github.com/voutcn/megahit
- **metaFlye** (Long-read assembler comparison arm, executed with its default polishing module)
- **OPERA-MS** (Hybrid assembler comparison arm with a binning-like clustering step utilizing contig connections)
- **minimap2** (Read mapping against the human reference to remove contaminant scaffolds from every assembly before Nx evaluation) — https://github.com/lh3/minimap2

## Examples

```
metaplatanus -IP1 PE_1.fq PE_2.fq -ont ONT.fq
```

## Evaluation signals

- Each assembler produces a valid final scaffold FASTA per sample before Nx computation.
- Nx plot areas per assembler/sample match the values in Supplementary Tables S15-S17 (human gut) and S22 (saliva).
- The Nx plot reproduces the reported dominance of MetaPlatanus contiguity over MEGAHIT, metaFlye and OPERA-MS (Figures 4-6).
- Human contaminant scaffolds are removed from every assembly before evaluation (reads mapped against the human reference).
- CheckM/Prokka over-half and near-complete scaffold counts are tabulated per assembler and consistent with the reported element tables (e.g. 22 near-complete scaffolds assembled only by MetaPlatanus in the saliva benchmark).

## Limitations

- Tool versions for the compared assemblers (metaFlye, OPERA-MS, MEGAHIT) are not reported in the article, complicating exact reproduction.
- metaFlye also requires additional short-read sequencing and polishing for rigorous downstream analysis; the comparison must include its default polishing module to be faithful to the study.
- Key supporting counts (Supplementary Tables S17, S24, S25; Figure 6) are deferred to supplementary materials, and no dataset accessions for the mock or human gut benchmark data are given in the discussion.

## Evidence

- [abstract] Benchmarking using mock datasets consisting of known bacteria with long reads or mate pairs revealed the high contiguity MetaPlatanus with a few interspecies misassemblies.: "Benchmarking using mock datasets consisting of known bacteria with long reads or mate pairs revealed the high contiguity MetaPlatanus with a few interspecies misassemblies."
- [discussion] metaFlye comparison arm context: "Although the metaFlye assembly exhibited high performance with large-scale accuracy and contiguity"
- [discussion] metaFlye polishing caveat: "metaFlye also requires additional short-read sequencing and polishing for rigorous downstream analysis."
- [methods] MetaPlatanus pipeline producing the compared scaffolds: "(3) Contig-assembly, scaffolding, gap-closing and binning are automatically executed at once."
- [readme] Input requirement for the compared hybrid assemblies: "MetaPlatanus requires at least one short-read paired-end library."
