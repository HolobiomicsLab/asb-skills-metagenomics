---
name: metagenomic-binning-and-scaffold-clustering
description: Use when a hybrid metagenome assembly (Illumina paired-end plus ONT/PacBio
  long reads or mate pairs) must be partitioned into species-specific scaffold clusters,
  or when near-complete genomes and biologically important elements must be detected
  per bin.
when_to_use_negative:
- Input is short-read-only data without any long-range links (no long reads, mate
  pairs or linked reads) — the article's binning/scaffolding relies on long-range
  sequence links and another assembler such as MEGAHIT alone may be more appropriate
- Reference-based or strain-resolved analysis is required — the study focuses on de
  novo assembly and reference information has not been well examined
- Single-isolate genome assembly with no mixed species — the binning step targets
  separating species in microbiome samples and adds no value for pure cultures
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaPlatanus
  role: Executes contig-assembly, scaffolding, gap-closing and binning in one run;
    produces out_finalClusters bin assignments
  repo: https://github.com/rkajitani/MetaPlatanus
- name: minimap2
  role: Read mapping used by the binning step (97% minimum identity default) to link
    scaffolds to species
  repo: https://github.com/lh3/minimap2
- name: MEGAHIT
  role: Assembles initial contigs from short reads that are subsequently binned (default
    k-mer coverage cutoff 4)
  repo: https://github.com/voutcn/megahit
- name: metaFlye
  role: Compared long-read assembler run with its default polishing module for benchmarking
    binned assemblies
- name: OPERA-MS
  role: Compared hybrid assembler whose binning-like clustering step is used as a
    benchmark alternative
- name: TGS-GapCloser
  role: Optional gap-closing module integrated into the MetaPlatanus pipeline
  repo: https://github.com/BGI-Qingdao/TGS-GapCloser
- name: NextPolish
  role: Optional polishing module integrated into the MetaPlatanus pipeline
  repo: https://github.com/Nextomics/NextPolish
- name: Racon
  role: Optional consensus/polishing module available in the pipeline
  repo: https://github.com/isovic/racon
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:b5adb788d4737259a158b439f972bc3c9d0c891251d2f9a24f477bcfadfd8909
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# metagenomic-binning-and-scaffold-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Species-level binning of metagenome assembly output using MetaPlatanus, which clusters scaffolds by coverage depth, k-mer/species-specific sequence composition and read-mapping of paired-end and long reads, yielding scaffold-to-bin assignments (out_finalClusters, out_finalClusters.tsv) that also feed back into contiguity improvement.

## When to use

Use when a hybrid metagenome assembly (Illumina paired-end plus ONT/PacBio long reads or mate pairs) must be partitioned into species-specific scaffold clusters, or when near-complete genomes and biologically important elements must be detected per bin.

## When NOT to use

- Input is short-read-only data without any long-range links (no long reads, mate pairs or linked reads) — the article's binning/scaffolding relies on long-range sequence links and another assembler such as MEGAHIT alone may be more appropriate
- Reference-based or strain-resolved analysis is required — the study focuses on de novo assembly and reference information has not been well examined
- Single-isolate genome assembly with no mixed species — the binning step targets separating species in microbiome samples and adds no value for pure cultures

## Inputs

- Illumina paired-end FASTQ reads (e.g. PE_1.fq, PE_2.fq) from metagenome samples
- Oxford Nanopore (ONT PromethION) long-read FASTQ (e.g. ONT.fq; 28-75 Gbp per saliva sample)
- Optional mate-pair or 10x linked-read libraries as alternative long-range links
- Assembled scaffold FASTA (out_final.fa) and bin assignment files (out_finalClusters, out_finalClusters.tsv) for downstream element detection

## Outputs

- Binned scaffold clusters per species (out_finalClusters, out_finalClusters.tsv)
- Final scaffold FASTA (out_final.fa) with bin-aware extensions
- Per-assembler counts of biologically important elements (coding genes, gene clusters, viral sequences, near-complete genomes)
- Nx-length contiguity plots comparing assemblers

## How to apply

Run MetaPlatanus on paired-end reads supplied via -IP1/-IP2 and long reads via -ont (or mate-pair libraries), letting it automatically execute contig-assembly, scaffolding, gap-closing and binning in one pass. The binning step maps reads back to contigs/scaffolds with a minimum identity threshold controlled by -min_map_idt_binning (default 97%) and combines coverage depth and k-mer composition to group sequences belonging to the same species, suppressing false interspecies links. Collect the final scaffold FASTA (out_final.fa) together with the bin assignment files (out_finalClusters, out_finalClusters.tsv) and use bin membership to extend scaffolds within species and to detect near-complete genomes, gene clusters and viral sequences per bin. Benchmark the binned assemblies against MEGAHIT (short-read-only), metaFlye (with its default polishing module) and OPERA-MS by tabulating detected elements and comparing Nx-length contiguity plots.

## Related tools

- **MetaPlatanus** (Executes contig-assembly, scaffolding, gap-closing and binning in one run; produces out_finalClusters bin assignments) — https://github.com/rkajitani/MetaPlatanus
- **minimap2** (Read mapping used by the binning step (97% minimum identity default) to link scaffolds to species) — https://github.com/lh3/minimap2
- **MEGAHIT** (Assembles initial contigs from short reads that are subsequently binned (default k-mer coverage cutoff 4)) — https://github.com/voutcn/megahit
- **metaFlye** (Compared long-read assembler run with its default polishing module for benchmarking binned assemblies)
- **OPERA-MS** (Compared hybrid assembler whose binning-like clustering step is used as a benchmark alternative)
- **TGS-GapCloser** (Optional gap-closing module integrated into the MetaPlatanus pipeline) — https://github.com/BGI-Qingdao/TGS-GapCloser
- **NextPolish** (Optional polishing module integrated into the MetaPlatanus pipeline) — https://github.com/Nextomics/NextPolish
- **Racon** (Optional consensus/polishing module available in the pipeline) — https://github.com/isovic/racon

## Examples

```
metaplatanus -IP1 PE_1.fq PE_2.fq -ont ONT.fq -min_map_idt_binning 97 -min_cov_contig 4 -o out
```

## Evaluation signals

- The binning output files out_finalClusters and out_finalClusters.tsv are produced and every scaffold in out_final.fa has a bin assignment
- Element-detection table over binned assemblies contains 22 near-complete scaffolds assembled exclusively by MetaPlatanus on the four human saliva PromethION samples, consistent with the reported result
- Nx-length contiguity plots reproduce MetaPlatanus dominance over MEGAHIT, metaFlye and OPERA-MS with few interspecies misassemblies
- Read mapping for binning respects the -min_map_idt_binning default of 97% identity, and contig-assembly uses k-mer coverage cutoff 4 with MEGAHIT
- Mock-dataset benchmarks show high contiguity with few interspecies misassemblies relative to compared assemblers

## Limitations

- Key benchmark evidence (Supplementary Tables S17, S22, S24, S25; Figure 6) is deferred to supplementary materials
- No tool versions reported for compared assemblers (metaFlye, OPERA-MS, MEGAHIT)
- metaFlye requires additional short-read sequencing and polishing for rigorous downstream analysis, complicating fair comparison
- Reference-based methods were not well examined in this study; binning performance on reference-guided workflows is unknown
- There is no comprehensive list of the types of local structures in assembly graphs, so misassembly classification is not exhaustive

## Evidence
