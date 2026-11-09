---
name: ont-long-read-scaffolding
description: Use when a metagenome sample has both Illumina paired-end reads and ONT
  long reads (e.g. 28-75 Gbp PromethION) and the goal is highly contiguous scaffolds
  or near-complete genomes for high-abundance species.
when_to_use_negative:
- Only short-read data are available — MetaPlatanus requires at least one short-read
  paired-end library and this skill depends on long-range ONT links
- Reference-based reassembly or strain-typing against known genomes is the goal —
  the study focuses on de novo methods and reference information was not well examined
- Extremely low-coverage long-read data where hybrid assemblers are reported advantageous
  and ONT links are too sparse to scaffold reliably
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: MetaPlatanus
  role: Hybrid metagenome assembler that uses ONT long reads as long-range links for
    scaffolding, gap-closing and binning
  repo: https://github.com/rkajitani/MetaPlatanus
- name: MEGAHIT
  role: Assembles initial short-read contigs inside MetaPlatanus and serves as a short-read-only
    comparator
  repo: https://github.com/voutcn/megahit
- name: minimap2
  role: Maps ONT long reads to contigs/scaffolds for linking and binning support
  repo: https://github.com/lh3/minimap2
- name: metaFlye
  role: Long-read-only comparator assembler run with its default polishing module
- name: OPERA-MS
  role: Hybrid comparator assembler using binning-like clustering of contig connections
- name: TGS-GapCloser
  role: Optional integrated gap-closing module using error-prone long reads
  repo: https://github.com/BGI-Qingdao/TGS-GapCloser
- name: NextPolish
  role: Optional integrated polishing module to fix base errors in scaffolds
  repo: https://github.com/Nextomics/NextPolish
- name: Racon
  role: Optional consensus module used within the gap-closing/polishing chain
  repo: https://github.com/isovic/racon
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:e854264d80ba5af4b5dfef775faf7408e1edf1ae5942bae15578a3d7f16d151e
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
  - MEGAHIT
  - MetaPlatanus
  - NextPolish
  - OPERA-MS
  - Racon
  - TGS-GapCloser
  - metaFlye
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ont-long-read-scaffolding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scaffold short-read metagenomic contigs into near-complete scaffolds using Oxford Nanopore PromethION long reads as long-range links via MetaPlatanus, which combines contig-assembly, scaffolding, gap-closing and binning with species-specific composition and coverage-depth checks to suppress interspecies misjoins.

## When to use

Use when a metagenome sample has both Illumina paired-end reads and ONT long reads (e.g. 28-75 Gbp PromethION) and the goal is highly contiguous scaffolds or near-complete genomes for high-abundance species.

## When NOT to use

- Only short-read data are available — MetaPlatanus requires at least one short-read paired-end library and this skill depends on long-range ONT links
- Reference-based reassembly or strain-typing against known genomes is the goal — the study focuses on de novo methods and reference information was not well examined
- Extremely low-coverage long-read data where hybrid assemblers are reported advantageous and ONT links are too sparse to scaffold reliably

## Inputs

- Illumina paired-end FASTQ read files (forward and reverse)
- Oxford Nanopore PromethION long-read FASTQ files (tens of Gbp per sample)
- Optional: reference-free binning inputs; MetaPlatanus requires at least one short-read paired-end library

## Outputs

- Final scaffold FASTA (MetaPlatanus out_final.fa)
- Bin assignment files (out_finalClusters, out_finalClusters.tsv)
- Nx-length contiguity plots comparing assemblers
- Tabulated counts of biologically important elements (coding genes, gene clusters, viral sequences, near-complete genomes) per assembler

## How to apply

Run MetaPlatanus supplying Illumina paired-end files via -IP1/-IP2 and the ONT read set via -ont (e.g. metaplatanus -IP1 PE_1.fq PE_2.fq -ont ONT.fq), so that contig-assembly, scaffolding, gap-closing and binning execute automatically in one run. Initial contigs are built with MEGAHIT using a k-mer coverage cutoff of 4 (-min_cov_contig default), and read mapping for binning uses a minimum identity of 97% (-min_map_idt_binning default). Long-range ONT links are used iteratively together with species-specific sequence compositions and coverage depth to extend scaffolds while filtering false interspecies links; binning information further improves contiguity. Assemble the same reads with comparators (short-read-only MEGAHIT, metaFlye with its default polishing module, OPERA-MS) to confirm that near-complete scaffolds are recovered uniquely. Collect out_final.fa plus out_finalClusters/out_finalClusters.tsv, then detect coding genes, gene clusters, viral sequences and near-complete genomes and generate Nx contiguity plots to validate the result.

## Related tools

- **MetaPlatanus** (Hybrid metagenome assembler that uses ONT long reads as long-range links for scaffolding, gap-closing and binning) — https://github.com/rkajitani/MetaPlatanus
- **MEGAHIT** (Assembles initial short-read contigs inside MetaPlatanus and serves as a short-read-only comparator) — https://github.com/voutcn/megahit
- **minimap2** (Maps ONT long reads to contigs/scaffolds for linking and binning support) — https://github.com/lh3/minimap2
- **metaFlye** (Long-read-only comparator assembler run with its default polishing module)
- **OPERA-MS** (Hybrid comparator assembler using binning-like clustering of contig connections)
- **TGS-GapCloser** (Optional integrated gap-closing module using error-prone long reads) — https://github.com/BGI-Qingdao/TGS-GapCloser
- **NextPolish** (Optional integrated polishing module to fix base errors in scaffolds) — https://github.com/Nextomics/NextPolish
- **Racon** (Optional consensus module used within the gap-closing/polishing chain) — https://github.com/isovic/racon

## Examples

```
metaplatanus -IP1 PE_1.fq PE_2.fq -ont ONT.fq -o out_saliva -t 32
```

## Evaluation signals

- On the human saliva PromethION benchmark, element detection recovers 22 near-complete scaffolds assembled exclusively by MetaPlatanus, with none produced by MEGAHIT, metaFlye or OPERA-MS
- Nx-length contiguity plots show MetaPlatanus dominating the compared assemblers, consistent with Figures 4-6 and Supplementary Tables S22, S24, S25
- Scaffolds show few interspecies misassemblies, verified against species-specific composition and coverage-depth consistency
- out_finalClusters.tsv assigns contigs to bins with read-mapping identity at or above the 97% binning threshold
- Gap-closed scaffold regions contain few remaining N-gaps after ONT-based gap closing

## Limitations

- Some key benchmark evidence is deferred to supplementary materials (Supplementary Tables S17, S24, S25; Figure 6) and may not be directly checkable from the main text
- No tool versions were reported for the compared assemblers (metaFlye, OPERA-MS, MEGAHIT), complicating exact reproduction
- metaFlye requires additional short-read sequencing and polishing for rigorous downstream analysis, so long-read-only comparator outputs are not directly equivalent
- Long-read-only datasets (e.g. large PacBio cow rumen data) did not cover many regions efficiently obtained from short reads, motivating the hybrid approach rather than long-read-only scaffolding
- There is no comprehensive list of the types of local structures in assembly graphs, so misjoin detection relies on heuristic coverage/composition checks

## Evidence
