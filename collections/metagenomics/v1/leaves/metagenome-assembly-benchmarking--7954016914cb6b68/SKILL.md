---
name: metagenome-assembly-benchmarking
description: Use when comparing metagenome assemblers on mock-community datasets (MP
  mock, GIS20) where reference genomes are known and NGA50, misassembly, and interspecies-misassembly
  metrics must be reproduced.
when_to_use_negative:
- Real environmental/human microbiome data with no reference genomes — metaQUAST-based
  NGA50 and interspecies misassembly counts cannot be computed without mock references
- Reference-based (mapping) assembly workflows — the article notes 'this study focuses
  on de novo methods, and reference information has not been well examined'
- Datasets lacking a short-read paired-end library — MetaPlatanus requires at least
  one short-read paired-end library
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MetaPlatanus
  role: Hybrid assembler under test; assembles short-read contigs with MEGAHIT, then
    scaffolds/gap-closes/bins using long-range links, species-specific compositions
    and coverage depth
  repo: https://github.com/rkajitani/MetaPlatanus
- name: MEGAHIT
  role: Assembler under test; also the default contig-assembly engine inside MetaPlatanus
    (k-mer coverage cutoff 4)
  repo: https://github.com/voutcn/megahit
- name: metaQUAST
  role: Evaluation of all assemblies against mock reference genomes to compute NGA50,
    misassemblies, and interspecies misassemblies
- name: metaFlye
  role: Long-read assembler comparator; its polishing module is executed as the default
    action
- name: OPERA-MS
  role: Hybrid assembler comparator run in de novo mode on GIS20
- name: TGS-GapCloser
  role: Gap closing step integrated in the MetaPlatanus pipeline
  repo: https://github.com/BGI-Qingdao/TGS-GapCloser
- name: NextPolish
  role: Polishing step integrated in the MetaPlatanus pipeline
  repo: https://github.com/Nextomics/NextPolish
- name: minimap2
  role: Read mapping supporting MetaPlatanus scaffolding/binning
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:75038cb55ed614bcb683af13e477478ed67b05f01f13f950c5e9b88c88decad2
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
  - NextPolish
  - OPERA-MS
  - SPAdes
  - TGS-GapCloser
  - metaFlye
  - metaQUAST
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
yes_rate: 0.6428571428571429
n_claims: 14
---

# metagenome-assembly-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark de novo metagenome assemblers (MetaPlatanus vs IDBA-UD, SPAdes, metaSPAdes, MEGAHIT, metaFlye, OPERA-MS) on mock communities with known references, using NGA50 and misassembly metrics from QUAST/metaQUAST. Rationale: contiguity alone is insufficient — interspecies misassemblies must be quantified to detect false joins.

## When to use

Use when comparing metagenome assemblers on mock-community datasets (MP mock, GIS20) where reference genomes are known and NGA50, misassembly, and interspecies-misassembly metrics must be reproduced.

## When NOT to use

- Real environmental/human microbiome data with no reference genomes — metaQUAST-based NGA50 and interspecies misassembly counts cannot be computed without mock references
- Reference-based (mapping) assembly workflows — the article notes 'this study focuses on de novo methods, and reference information has not been well examined'
- Datasets lacking a short-read paired-end library — MetaPlatanus requires at least one short-read paired-end library

## Inputs

- Illumina paired-end FASTQ reads of mock communities (HiSeq2500/MiSeq)
- Mate-pair FASTQ libraries (4 kbp and 8 kbp, MP mock)
- ONT and PacBio long-read FASTQ (GIS20)
- Reference genomes of the known mock bacteria for metaQUAST
- Assembled contig/scaffold FASTA files from each assembler (e.g. MetaPlatanus out_result/out_final.fa)

## Outputs

- Per-assembler NGA50 comparison table (e.g. nga50_comparison.tsv / gis20_benchmark_quast.tsv mirroring Supplementary Table S14)
- metaQUAST reports with misassembly counts, mean sequence identity, and interspecies misassembly counts
- Bar chart of NGA50 per assembler per mock case (analogous to Supplementary Figure S5)

## How to apply

Download mock reads (e.g. MP mock from DDBJ BioProject PRJDB4377; GIS20 Illumina paired-end plus ONT and PacBio long reads) and organize per case. Run MetaPlatanus supplying paired-end libraries with -IP1, mate-pair (4/8 kbp) libraries with -OP, and long reads with -ont/-p (e.g. -t 8 -m 64), which automatically performs MEGAHIT-based contig assembly, scaffolding, MetaBAT2 binning (default min mapping identity 97%), TGS-GapCloser gap closing and NextPolish polishing into out_final.fa. Run competitors (IDBA-UD, SPAdes, metaSPAdes, MEGAHIT on paired-end reads; metaFlye with default polishing; OPERA-MS de novo mode) on identical reads. Evaluate all assemblies with metaQUAST against the known mock references, extract NGA50, misassembly count, sequence identity and interspecies misassemblies, and compare against Supplementary Tables S8-S10 and S14.

## Related tools

- **MetaPlatanus** (Hybrid assembler under test; assembles short-read contigs with MEGAHIT, then scaffolds/gap-closes/bins using long-range links, species-specific compositions and coverage depth) — https://github.com/rkajitani/MetaPlatanus
- **MEGAHIT** (Assembler under test; also the default contig-assembly engine inside MetaPlatanus (k-mer coverage cutoff 4)) — https://github.com/voutcn/megahit
- **metaQUAST** (Evaluation of all assemblies against mock reference genomes to compute NGA50, misassemblies, and interspecies misassemblies)
- **metaFlye** (Long-read assembler comparator; its polishing module is executed as the default action)
- **OPERA-MS** (Hybrid assembler comparator run in de novo mode on GIS20)
- **TGS-GapCloser** (Gap closing step integrated in the MetaPlatanus pipeline) — https://github.com/BGI-Qingdao/TGS-GapCloser
- **NextPolish** (Polishing step integrated in the MetaPlatanus pipeline) — https://github.com/Nextomics/NextPolish
- **minimap2** (Read mapping supporting MetaPlatanus scaffolding/binning) — https://github.com/lh3/minimap2

## Examples

```
metaplatanus -t 8 -m 64 -IP1 PE_1.fq PE_2.fq -OP mp4kb_1.fq mp4kb_2.fq mp8kb_1.fq mp8kb_2.fq
```

## Evaluation signals

- Reproduced ranking must match the published result: MetaPlatanus attains the largest NGA50 for mock cases 1-3 in Supplementary Table S14 and Supplementary Figure S5
- On GIS20, MetaPlatanus should achieve the highest NGA50 for both ONT and PacBio long-read datasets compared to metaSPAdes, OPERA-MS and metaFlye
- Interspecies misassembly counts for MetaPlatanus should be low relative to competitors, consistent with 'high contiguity ... with a few interspecies misassemblies'
- All assemblies must be evaluated on identical read sets with the same metaQUAST reference genomes; NGA50 values should populate a complete per-assembler per-case table with no missing cases

## Limitations

- No tool versions reported for compared assemblers (metaFlye, OPERA-MS, MEGAHIT, etc.) in the discussion section
- Key quantitative evidence is deferred to supplementary materials (Supplementary Tables S14, S17, S24, S25; Figure 6); raw reads of MP mock are under DDBJ BioProject PRJDB4377, and assemblies are on FigShare doi:10.6084/m9.figshare.15090603.v1
- Reference-based methods were not well examined; benchmark conclusions are limited to de novo assemblers
- metaFlye 'requires additional short-read sequencing and polishing for rigorous downstream analysis', which complicates fair comparison

## Evidence

- [results] In the MP mock mate-pair benchmark, MetaPlatanus had the largest NGA50s for all cases (Supplementary Table S14 and Supplementary Figure S5).: [evidence span withheld — anchor status: not_found]
