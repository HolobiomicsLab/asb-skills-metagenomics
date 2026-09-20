---
name: hybrid-metagenome-assembly-execution
description: Use when you have Illumina paired-end metagenomic reads plus at least
  one long-range library (ONT or PacBio long reads, mate pairs, or 10x linked reads)
  and need de novo scaffolds with high contiguity and few interspecies misassemblies.
when_to_use_negative:
- Only short-read data is available — MetaPlatanus requires at least one short-read
  paired-end library and is designed to exploit long-range links; a short-read-only
  assembler such as MEGAHIT is more appropriate
- Reference-guided assembly is desired — the study focuses on de novo methods and
  reference information has not been well examined
- Only HiFi long reads are available and short-read sequencing is not planned — this
  hybrid pipeline is built around accurate short-read contigs plus error-prone long-range
  links
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MetaPlatanus
  role: Hybrid metagenome assembler orchestrating contig-assembly, scaffolding, gap-closing
    and binning in one run
  repo: https://github.com/rkajitani/MetaPlatanus
- name: MEGAHIT
  role: Assembles initial contigs from Illumina short reads (default k-mer coverage
    cutoff 4)
  repo: https://github.com/voutcn/megahit
- name: TGS-GapCloser
  role: Closes N-gaps in scaffolds using error-prone ONT/PacBio long reads (can be
    disabled with -no_tgsgapcloser)
  repo: https://github.com/BGI-Qingdao/TGS-GapCloser
- name: NextPolish
  role: Polishes scaffold base errors (SNV/Indel) using short and/or long reads (can
    be disabled with -no_nextpolish)
  repo: https://github.com/Nextomics/NextPolish
- name: Racon
  role: Consensus module used to polish raw TGS reads when only raw long reads are
    given to TGS-GapCloser
  repo: https://github.com/isovic/racon
- name: minimap2
  role: Read mapping for binning read-mapping identity assessment
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_007
  - task_001
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:1abe0e6de8daaea65db6732fdb2b979038c6c2d42db7aee1a905e6da2a3816e1
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
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# hybrid-metagenome-assembly-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run MetaPlatanus, a hybrid metagenome assembler that combines accurate short-read contig assembly (MEGAHIT) with long-range links (ONT/PacBio long reads or mate pairs), species-specific coverage/k-mer features and binning (MetaBAT2), followed by TGS-GapCloser gap closing and NextPolish polishing, to produce low-misassembly, high-contiguity metagenomic scaffolds.

## When to use

Use when you have Illumina paired-end metagenomic reads plus at least one long-range library (ONT or PacBio long reads, mate pairs, or 10x linked reads) and need de novo scaffolds with high contiguity and few interspecies misassemblies.

## When NOT to use

- Only short-read data is available — MetaPlatanus requires at least one short-read paired-end library and is designed to exploit long-range links; a short-read-only assembler such as MEGAHIT is more appropriate
- Reference-guided assembly is desired — the study focuses on de novo methods and reference information has not been well examined
- Only HiFi long reads are available and short-read sequencing is not planned — this hybrid pipeline is built around accurate short-read contigs plus error-prone long-range links

## Inputs

- Illumina paired-end FASTQ reads (mandatory, supplied via -IP1)
- Oxford Nanopore long-read FASTQ (supplied via -ont)
- PacBio long-read FASTQ (supplied via -p)
- Optional mate-pair or 10x linked-read libraries as alternative long-range links

## Outputs

- Final scaffold FASTA (out_result/out_final.fa)
- Binning cluster assignment files (out_finalClusters, out_finalClusters.tsv)
- Per-module ablation scaffold sets for comparison via QUAST/metaQUAST (NGA50, misassemblies, mean sequence identity)

## How to apply

Run MetaPlatanus in a single invocation supplying the mandatory Illumina paired-end library via -IP1 and long reads via -ont or -p (e.g. metaplatanus -t 8 -m 64 -IP1 PE_1.fq PE_2.fq -ont ONT.fq -p pacbio.fq); contig-assembly, scaffolding, gap-closing and binning then execute automatically, writing out_result/out_final.fa. Contig assembly uses MEGAHIT with a default k-mer coverage cutoff of 4 (2 if MEGAHIT is not used), and binning uses MetaBAT2 with a default minimum mapping identity of 97%. TGS-GapCloser fills N-gaps using the error-prone long reads (polishing raw TGS reads with Racon when only raw long reads are supplied), and NextPolish fixes residual base errors. To attribute performance to individual modules, rerun with -no_tgsgapcloser (disables both gap closing and NextPolish correction, 'w/o closing' mode) or -no_nextpolish (disables only NextPolish, 'w/o correction' mode) and evaluate all scaffold sets with QUAST/metaQUAST for NGA50, misassemblies and mean sequence identity.

## Related tools

- **MetaPlatanus** (Hybrid metagenome assembler orchestrating contig-assembly, scaffolding, gap-closing and binning in one run) — https://github.com/rkajitani/MetaPlatanus
- **MEGAHIT** (Assembles initial contigs from Illumina short reads (default k-mer coverage cutoff 4)) — https://github.com/voutcn/megahit
- **TGS-GapCloser** (Closes N-gaps in scaffolds using error-prone ONT/PacBio long reads (can be disabled with -no_tgsgapcloser)) — https://github.com/BGI-Qingdao/TGS-GapCloser
- **NextPolish** (Polishes scaffold base errors (SNV/Indel) using short and/or long reads (can be disabled with -no_nextpolish)) — https://github.com/Nextomics/NextPolish
- **Racon** (Consensus module used to polish raw TGS reads when only raw long reads are given to TGS-GapCloser) — https://github.com/isovic/racon
- **minimap2** (Read mapping for binning read-mapping identity assessment) — https://github.com/lh3/minimap2

## Examples

```
metaplatanus -t 8 -m 64 -IP1 PE_1.fq PE_2.fq -ont ONT.fq -p pacbio.fq
```

## Evaluation signals

- out_result/out_final.fa exists and is a valid scaffold FASTA after a complete run
- On a mock benchmark (e.g. GIS20), metaQUAST NGA50 exceeds that of metaSPAdes, OPERA-MS (de novo) and metaFlye on the same ONT/PacBio read sets
- Ablation validation: the -no_nextpolish ('w/o correction') run shows increased misassemblies and decreased NGA50 relative to the full run
- Ablation validation: the -no_tgsgapcloser ('w/o closing') run shows markedly fewer misassemblies and higher mean sequence identity than other tools
- Binning used the default 97% minimum mapping identity; contig assembly used the default k-mer coverage cutoff 4 with MEGAHIT

## Limitations

- metaFlye with its default polishing module is a strong long-read-only competitor and may require additional short-read sequencing and polishing for rigorous downstream analysis; MetaPlatanus's advantage rests on hybrid inputs
- Reference-based (reference-guided) methods were not well examined in this study, so this skill covers de novo assembly only
- Only fasta format of TGS reads is acceptable as input to TGS-GapCloser; 10x linked-read support is experimental
- Key quantitative evidence (e.g. Supplementary Tables S8-S10, S15-S17, S22, S24, S25) resides in supplementary materials and must be consulted for expected metric ranges

## Evidence

- [readme] Only fasta format of TGS reads is acceptable.: [evidence span withheld — anchor status: external_doc]
