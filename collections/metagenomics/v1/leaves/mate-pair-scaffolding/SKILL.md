---
name: mate-pair-scaffolding
description: Use when assembling metagenome mock or real communities from Illumina
  paired-end reads plus mate-pair libraries (e.g. 4 kbp/8 kbp jumping libraries),
  and NGA50 contiguity versus short-read-only assemblers must be quantified.
when_to_use_negative:
- Only short paired-end reads are available and no mate-pair/jumping library or other
  long-range information exists — mate-pair scaffolding links cannot be exploited
- The analysis is reference-based; the study focuses on de novo methods and reference
  information has not been well examined
- Aiming to assess assembly of biologically important elements (genes, viral sequences,
  near-complete genomes) on nanopore long-read data rather than mate-pair benchmarks
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MetaPlatanus
  role: hybrid metagenome assembler that runs contig-assembly, scaffolding, gap-closing
    and binning using mate-pair long-range links
  repo: https://github.com/rkajitani/MetaPlatanus
- name: MEGAHIT
  role: competitor short-read assembler (also combined by MetaPlatanus to assemble
    initial contigs, like v1.2.3)
  repo: https://github.com/voutcn/megahit
- name: minimap2
  role: dependency of MetaPlatanus used for read mapping supporting binning and link
    usage
  repo: https://github.com/lh3/minimap2
- name: IDBA-UD
  role: compared assembler on the MP mock paired-end and mate-pair reads
- name: SPAdes
  role: compared assembler on the MP mock paired-end and mate-pair reads
- name: metaSPAdes
  role: compared assembler on the MP mock paired-end and mate-pair reads
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:1551a8afe4e804357f9577f30d9a92875ad3e321aaa6ae392e96ec5da6aaf2c0
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
yes_rate: 0.7
n_claims: 10
---

# mate-pair-scaffolding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scaffold metagenome contigs using Illumina mate-pair (jumping library) long-range links while suppressing interspecies misjoins, as implemented in MetaPlatanus by iteratively combining long-range links, coverage depth, k-mer/species-specific composition and binning.

## When to use

Use when assembling metagenome mock or real communities from Illumina paired-end reads plus mate-pair libraries (e.g. 4 kbp/8 kbp jumping libraries), and NGA50 contiguity versus short-read-only assemblers must be quantified.

## When NOT to use

- Only short paired-end reads are available and no mate-pair/jumping library or other long-range information exists — mate-pair scaffolding links cannot be exploited
- The analysis is reference-based; the study focuses on de novo methods and reference information has not been well examined
- Aiming to assess assembly of biologically important elements (genes, viral sequences, near-complete genomes) on nanopore long-read data rather than mate-pair benchmarks

## Inputs

- Illumina paired-end FASTQ library (required, passed with -IP1)
- 4 kbp mate-pair FASTQ library (passed with -OP)
- 8 kbp mate-pair FASTQ library (passed with -OP)
- Raw reads from DDBJ Sequence Read Archive BioProject PRJDB4377 (MP mock samples)

## Outputs

- MetaPlatanus final scaffold FASTA (out_final.fa)
- Per-assembler NGA50 and contiguity metrics for each mock case
- Comparison table (nga50_comparison.tsv) mirroring Supplementary Table S14
- Bar chart of NGA50 per assembler per case analogous to Supplementary Figure S5

## How to apply

Download raw reads (e.g. HiSeq2500/MiSeq paired-end plus 4 kbp and 8 kbp mate-pair libraries from DDBJ BioProject PRJDB4377) and run MetaPlatanus supplying the paired-end library via -IP1 and the mate-pair libraries via -OP, with -t for threads and -m for memory (e.g. -t 8 -m 64). MetaPlatanus automatically executes contig-assembly, scaffolding, gap-closing and binning at once, using long-range mate-pair links together with coverage depth, k-mer frequencies and binning results to join sequences while avoiding false interspecies links. Run competitors (IDBA-UD, SPAdes, metaSPAdes, MEGAHIT) on the paired-end reads, supplying mate-pair libraries where each tool supports them. Evaluate every output (MetaPlatanus out_final.fa and each competitor's contig/scaffold FASTA) with an assembly evaluator to compute NGA50 and related contiguity metrics, then tabulate and plot per-case NGA50 to mirror Supplementary Table S14 and Supplementary Figure S5.

## Related tools

- **MetaPlatanus** (hybrid metagenome assembler that runs contig-assembly, scaffolding, gap-closing and binning using mate-pair long-range links) — https://github.com/rkajitani/MetaPlatanus
- **MEGAHIT** (competitor short-read assembler (also combined by MetaPlatanus to assemble initial contigs, like v1.2.3)) — https://github.com/voutcn/megahit
- **minimap2** (dependency of MetaPlatanus used for read mapping supporting binning and link usage) — https://github.com/lh3/minimap2
- **IDBA-UD** (compared assembler on the MP mock paired-end and mate-pair reads)
- **SPAdes** (compared assembler on the MP mock paired-end and mate-pair reads)
- **metaSPAdes** (compared assembler on the MP mock paired-end and mate-pair reads)

## Examples

```
metaplatanus.pl -IP1 pe_1.fq pe_2.fq -OP mp4_1.fq mp4_2.fq mp8_1.fq mp8_2.fq -t 8 -m 64
```

## Evaluation signals

- MetaPlatanus attains the largest NGA50 for mock cases 1-3 in the reproduced comparison table matching Supplementary Table S14
- Reproduced bar chart ranking matches Supplementary Figure S5 ordering (MetaPlatanus > IDBA-UD, SPAdes, metaSPAdes, MEGAHIT)
- Interspecies misassemblies remain few, consistent with the reported 'few interspecies misassemblies' benchmark result
- Final scaffold FASTA (out_final.fa) is produced for all three MP mock cases and is evaluable by the NGA50 metric
- All mate-pair libraries (4 kbp and 8 kbp) were supplied via -OP so long-range links contributed to scaffolding, not only the -IP1 paired-end library

## Limitations

- MetaPlatanus requires at least one short-read paired-end library; mate-pair links alone are insufficient
- False interspecies links in mate-pair data must be removed; without species-specific composition/coverage checks, joining a small number of links risks misjoins for low-abundance species
- 10x linked-read support in MetaPlatanus is marked as experimental
- No tool versions were reported for the compared assemblers (metaFlye, OPERA-MS, MEGAHIT, etc.), so version-specific reproducibility is uncertain

## Evidence
