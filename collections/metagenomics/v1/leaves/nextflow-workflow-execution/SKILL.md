---
name: nextflow-workflow-execution
description: Use when benchmarking de novo short-read assembly of a defined mock community
  across multiple assemblers under identical conditions, or when reproducing global
  (Contigs, Nx, Mapped reads, Misassembled contigs) and per-reference (Contiguity,
  Multiplicity, Validity, Lx, NGx) assembly metrics from.
when_to_use_negative:
- Assembling eukaryotic genomes or single bacterial isolates — LMAS benchmarks prokaryotic
  metagenomic de novo assembly on defined mock communities with known references
- Strain-level SNP resolution is the goal — the study found meaningful strain resolution
  at the SNP level was not achieved even by the best assemblers tested
- You need long-read or hybrid assembly benchmarking — LMAS targets short-read metagenomic
  assembly
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: LMAS
  role: Nextflow workflow that orchestrates parallel de novo assembly and evaluation
    under identical conditions and generates the HTML metrics report
  repo: https://github.com/B-UMMI/LMAS
- name: ABySS
  role: One of the 11 assemblers executed in its container by the workflow; performed
    relatively poorly on complex samples
  repo: https://github.com/BirolLab/abyss
- name: MetaHipmer2
  role: Metagenomic assembler benchmarked by LMAS; performed relatively poorly on
    complex samples
- name: minia
  role: Assembler executed within the GATB-Minia Pipeline by LMAS; performed relatively
    poorly on complex samples
  repo: https://github.com/GATB/minia
- name: VelvetOptimiser
  role: Assembler benchmarked by LMAS; performed relatively poorly on complex samples
- name: InSilicoSeq
  role: Recommended simulator for generating in silico mock community sequencing data
    as workflow input
  repo: https://github.com/HadrienG/InSilicoSeq
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:e4f579fb4404ed903f5db88be30f8250a81afea742f04daedddd3523a6e21e35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - ABySS
  - InSilicoSeq
  - LMAS
  - MetaHipmer2
  - VelvetOptimiser
  - minia
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# nextflow-workflow-execution

## Summary

Execute the LMAS Nextflow workflow to benchmark metagenomic de novo assemblers on defined mock communities, running all 11 containerized assemblers in parallel and producing a metric-rich interactive HTML report. This skill covers installing LMAS, wiring inputs (FASTQ plus multi-fasta reference), and extracting global and per-reference assembly metrics.

## When to use

Use when benchmarking de novo short-read assembly of a defined mock community across multiple assemblers under identical conditions, or when reproducing global (Contigs, Nx, Mapped reads, Misassembled contigs) and per-reference (Contiguity, Multiplicity, Validity, Lx, NGx) assembly metrics from paired-end FASTQ plus reference replicon FASTA inputs.

## When NOT to use

- Assembling eukaryotic genomes or single bacterial isolates — LMAS benchmarks prokaryotic metagenomic de novo assembly on defined mock communities with known references
- Strain-level SNP resolution is the goal — the study found meaningful strain resolution at the SNP level was not achieved even by the best assemblers tested
- You need long-read or hybrid assembly benchmarking — LMAS targets short-read metagenomic assembly

## Inputs

- Paired-end Illumina FASTQ files of a mock community (e.g. BMock12 SRX4901583, HiSeq 2500 reads, optionally downsampled to 20%)
- Single multi-fasta file of complete reference replicons for the mock community (for --reference)
- Container runtime (Docker or Singularity) and Nextflow >= 21.04.1

## Outputs

- Assembled contigs from 11 assemblers (ABySS, GATB-Minia Pipeline, IDBA-UD, MEGAHIT, MetaHipMer2, metaSPAdes, MINIA, SKESA, SPAdes, Unicycler, VelvetOptimiser)
- Global metrics per assembler: Contigs, Basepairs, Nx, Mapped reads, Misassembled contigs, Number of Ns
- Per-reference metrics per assembler vs each replicon: Contiguity, Multiplicity, Validity, Parsimony, Identity, Breadth of coverage, Lx, NAx, NGx, Aligned Basepairs, Number of Ns
- Interactive HTML report of global and reference replicon-specific performance metrics

## How to apply

Install LMAS via conda (`conda install -c bioconda lmas`) or clone github.com/B-UMMI/LMAS, ensuring Nextflow >= 21.04.1 and Docker or Singularity are available since each assembler runs in its own container. Provide paired-end FASTQ files via --fastq and concatenate all complete reference genomes into a single multi-fasta supplied via --reference; for complex communities such as the 12-strain BMock12 standard (SRX4901583), downsample to 20% of original reads as in the published protocol. Run with default parameters (8 CPUs, 32 GB memory, --minLength 1000, --mapped_reads_threshold 0.75, --n_target 0.5, --l_target 0.9) so that ABySS, GATB-Minia Pipeline, IDBA-UD, MEGAHIT, MetaHipMer2, metaSPAdes, MINIA, SKESA, SPAdes, Unicycler and VelvetOptimiser assemble the data in parallel under the same conditions. Note that filtering contigs <1 kbp attenuated differences between genomic and metagenomic assemblers in the source study, and that ABySS can fail intermittently (one failed run for ERR2984773), so re-running is expected behavior. Extract global metrics and per-reference metrics from the results directory and HTML report, recording assembler versions and container identifiers for reproducibility.

## Related tools

- **LMAS** (Nextflow workflow that orchestrates parallel de novo assembly and evaluation under identical conditions and generates the HTML metrics report) — https://github.com/B-UMMI/LMAS
- **ABySS** (One of the 11 assemblers executed in its container by the workflow; performed relatively poorly on complex samples) — https://github.com/BirolLab/abyss
- **MetaHipmer2** (Metagenomic assembler benchmarked by LMAS; performed relatively poorly on complex samples)
- **minia** (Assembler executed within the GATB-Minia Pipeline by LMAS; performed relatively poorly on complex samples) — https://github.com/GATB/minia
- **VelvetOptimiser** (Assembler benchmarked by LMAS; performed relatively poorly on complex samples)
- **InSilicoSeq** (Recommended simulator for generating in silico mock community sequencing data as workflow input) — https://github.com/HadrienG/InSilicoSeq

## Examples

```
conda install -c bioconda lmas; lmas --fastq bmock12_R1.fq.gz bmock12_R2.fq.gz --reference bmock12_references.fasta --minLength 1000
```

## Evaluation signals

- Output report/CSV contains rows for all 11 assemblers with numeric values in Contigs, Nx, and Breadth of coverage columns
- Assembler versions and container identifiers are recorded alongside results
- Default parameter values are respected: --minLength 1000, --mapped_reads_threshold 0.75, --n_target 0.5, --l_target 0.9
- Workflow completed 3 times reproducibly (the source study ran LMAS 3 times per sample with default parameters); intermittent assembler failures (e.g. ABySS on ERR2984773) are documented rather than silently missing
- Filtered results (contigs <1 kbp) can be compared to unfiltered results, with differences expected to attenuate after filtering

## Limitations

- LMAS is computationally intensive due to the de novo assembly process; 8 CPUs and 32 GB memory are the stated defaults
- Assembler performance conclusions hold for the tested mock communities (e.g. ZymoBIOMICS, BMock12) and may not generalize to more complex metagenomic samples
- Usability findings are limited to default parameters; non-default parameter tuning was not explored
- Some legacy assemblers may fail intermittently and must be excluded from subsequent analyses when they do (ABySS, MetaHipmer2, minia, VelvetOptimiser were excluded after poor performance)

## Evidence

- [abstract] LMAS (Last Metagenomic Assembler Standing) was developed as a flexible platform allowing users to evaluate assembler performance given known standard communities.: "LMAS (Last Metagenomic Assembler Standing) was developed as a flexible platform allowing users to evaluate assembler performance given known standard communities."
- [discussion] LMAS is the only software-integrating assembly and its evaluation into a single pipeline, guaranteeing the same conditions are met for all tools.: "LMAS is the only software-integrating assembly and its evaluation into a single pipeline, guaranteeing the same conditions are met for all tools."
- [discussion] Modularity and containerization: "LMAS was built with modularity and containerization as keystones, leveraging the parallelization of processes and guaranteeing reproducibility across platforms."
- [results] Downsampling BMock12: "we downsampled to have 20% of the reads available in the original sample and further processed only these"
- [results] ABySS intermittent failure: "ABySS failed to produce an assembly for sample ERR2984773 for 1 of the runs"
