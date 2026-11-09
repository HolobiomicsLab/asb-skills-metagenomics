---
name: long-read-metagenome-assembly
description: Use when assembling a metagenome from a metagenomic dataset that has
  at least one short-read paired-end Illumina library plus long reads (ONT or PacBio)
  or mate-pair links, and you need NGA50/accuracy benchmarking against assemblers
  like metaFlye or OPERA-MS.
when_to_use_negative:
- Only long reads are available — MetaPlatanus requires at least one short-read paired-end
  library; use a long-read-only assembler such as metaFlye instead
- Reference-based (extract/compare to references) assembly is the goal — the study
  focuses on de novo methods and reference information was not well examined
- No long-range information at all (short reads only) — the skill's advantage comes
  from long-range links, binning, and coverage-depth features, so a pure short-read
  assembler like MEGAHIT alone is more appropriate
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MetaPlatanus
  role: Main hybrid metagenome assembler; chains contig assembly, scaffolding, gap
    closing, and binning automatically
  repo: https://github.com/rkajitani/MetaPlatanus
- name: MEGAHIT
  role: Assembles initial short-read contigs inside the MetaPlatanus pipeline (default
    k-mer coverage cutoff 4)
  repo: https://github.com/voutcn/megahit
- name: minimap2
  role: Read mapping used for long-read linking and binning-based misassembly correction
  repo: https://github.com/lh3/minimap2
- name: TGS-GapCloser
  role: Gap closing on scaffolds using error-prone long reads
  repo: https://github.com/BGI-Qingdao/TGS-GapCloser
- name: NextPolish
  role: Polishing of the gap-closed assembly
  repo: https://github.com/Nextomics/NextPolish
- name: Racon
  role: Optional consensus module invoked when TGS-GapCloser polishes raw TGS reads
  repo: https://github.com/isovic/racon
- name: metaQUAST
  role: Reference-based evaluation of assemblies (NGA50, misassemblies, interspecies
    misassemblies)
- name: metaFlye
  role: Comparator long-read-only assembler (polishing module executed as default
    action)
- name: OPERA-MS
  role: Comparator hybrid assembler run in de novo mode
- name: metaSPAdes
  role: Comparator hybrid short-read assembler
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:9fd7129c2ba03f5e8167037ba1360713b5f396a810d3104e944488efea47f7f8
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
yes_rate: 0.7333333333333333
n_claims: 15
---

# long-read-metagenome-assembly

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Hybrid metagenome assembly that combines accurate Illumina short-read contig assembly with long-range links (ONT/PacBio long reads or mate pairs), species-specific sequence composition, coverage depth, and binning to maximize contiguity while suppressing interspecies misassemblies.

## When to use

Use when assembling a metagenome from a metagenomic dataset that has at least one short-read paired-end Illumina library plus long reads (ONT or PacBio) or mate-pair links, and you need NGA50/accuracy benchmarking against assemblers like metaFlye or OPERA-MS.

## When NOT to use

- Only long reads are available — MetaPlatanus requires at least one short-read paired-end library; use a long-read-only assembler such as metaFlye instead
- Reference-based (extract/compare to references) assembly is the goal — the study focuses on de novo methods and reference information was not well examined
- No long-range information at all (short reads only) — the skill's advantage comes from long-range links, binning, and coverage-depth features, so a pure short-read assembler like MEGAHIT alone is more appropriate

## Inputs

- Illumina paired-end short reads (FASTQ, required; MetaPlatanus requires at least one short-read paired-end library)
- Oxford Nanopore long reads (FASTQ/FASTA)
- PacBio long reads (FASTQ/FASTA)
- Mate-pair or 10x linked-read libraries (optional long-range links)
- Reference genomes of known mock species (for metaQUAST evaluation)

## Outputs

- Final assembled scaffolds: out_result/out_final.fa
- Per-stage intermediate assemblies (contigs, scaffolds, gap-closed, binned scaffolds) from the automatic pipeline
- QUAST/metaQUAST benchmark table (NGA50, misassemblies, mean sequence identity, interspecies misassemblies per assembler)

## How to apply

Run MetaPlatanus in a single invocation (e.g. metaplatanus -t 8 -m 64 -IP1 PE_1.fq PE_2.fq -ont ONT.fq -p pacbio.fq), which automatically chains MEGAHIT-based contig assembly (default k-mer coverage cutoff -min_cov_contig 4 with MEGAHIT, 2 otherwise), scaffolding using long-range links, MetaBAT2 binning (minimum mapping identity -min_map_idt_binning 97), TGS-GapCloser gap closing, and NextPolish polishing, writing out_result/out_final.fa. The rationale is that accurate short reads provide base-level correctness while long reads supply the long-range links needed to join contigs from low-abundance species without interspecies misjoining. For benchmarking, assemble the identical read sets with metaSPAdes (hybrid), OPERA-MS de novo mode, and metaFlye (polishing module executed as default), then evaluate all assemblies with metaQUAST against known mock reference genomes, extracting NGA50, misassembly count, mean sequence identity, and interspecies misassembly count. Correct application is judged by high NGA50 with few interspecies misassemblies relative to comparator assemblers, as reported in the paper's Table 1 and Supplementary Tables S8–S10.

## Related tools

- **MetaPlatanus** (Main hybrid metagenome assembler; chains contig assembly, scaffolding, gap closing, and binning automatically) — https://github.com/rkajitani/MetaPlatanus
- **MEGAHIT** (Assembles initial short-read contigs inside the MetaPlatanus pipeline (default k-mer coverage cutoff 4)) — https://github.com/voutcn/megahit
- **minimap2** (Read mapping used for long-read linking and binning-based misassembly correction) — https://github.com/lh3/minimap2
- **TGS-GapCloser** (Gap closing on scaffolds using error-prone long reads) — https://github.com/BGI-Qingdao/TGS-GapCloser
- **NextPolish** (Polishing of the gap-closed assembly) — https://github.com/Nextomics/NextPolish
- **Racon** (Optional consensus module invoked when TGS-GapCloser polishes raw TGS reads) — https://github.com/isovic/racon
- **metaQUAST** (Reference-based evaluation of assemblies (NGA50, misassemblies, interspecies misassemblies))
- **metaFlye** (Comparator long-read-only assembler (polishing module executed as default action))
- **OPERA-MS** (Comparator hybrid assembler run in de novo mode)
- **metaSPAdes** (Comparator hybrid short-read assembler)

## Examples

```
metaplatanus -t 8 -m 64 -IP1 PE_1.fq PE_2.fq -ont ONT.fq -p pacbio.fq
```

## Evaluation signals

- metaQUAST NGA50 for the MetaPlatanus assembly should exceed that of metaSPAdes, OPERA-MS (de novo), and metaFlye on the same read sets, as reported in Table 1 and Supplementary Tables S8–S10
- Interspecies misassembly count should remain low (a few) compared to comparator assemblers on mock datasets of known bacteria
- mean sequence identity from metaQUAST should not be degraded by the NextPolish polishing step
- out_result/out_final.fa must exist and contain scaffolds longer than the MEGAHIT contig stage, confirming scaffolding and binning extension occurred
- Pipeline log should show all stages (contig-assembly, scaffolding, gap-closing, binning) completed automatically in one invocation

## Limitations

- MetaPlatanus requires at least one short-read paired-end library and cannot run on long reads alone
- No tool versions were reported for the compared assemblers (metaFlye, OPERA-MS, MEGAHIT), complicating exact benchmark reproduction
- Reference-based methods were not well examined in the study; conclusions apply to de novo assembly
- metaFlye comparator requires additional short-read sequencing and polishing for rigorous downstream analysis
- 10x linked-read support is experimental; a comprehensive list of assembly-graph local structures does not exist
- Key benchmark details are deferred to supplementary tables (e.g. S8–S10, S17, S24, S25) and some mock dataset accessions are only cited via the reference list (DDBJ PRJDB4377 for MP mock samples)

## Evidence
