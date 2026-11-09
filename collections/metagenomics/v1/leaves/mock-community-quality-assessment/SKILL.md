---
name: mock-community-quality-assessment
description: Use when comparing de novo short-read assembler performance on a known
  mock community standard with reference genomes available, to judge which assemblers
  produce accurate, contiguous assemblies.
when_to_use_negative:
- The input is a single isolate genome without a mock community reference set — LMAS's
  benchmarking logic assumes defined communities with known replicons
- Strain-level (SNP-level) discrimination is the goal — the article reports meaningful
  strain resolution was not achieved, even by the best assemblers tested
- Long-read or hybrid assembly is required — LMAS evaluates short-read metagenomic
  prokaryotic de novo assembly only
edam_operation: http://edamontology.org/operation_0526
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: LMAS
  role: Nextflow pipeline that runs all assemblers in parallel and computes global
    and per-reference evaluation metrics in one workflow
  repo: https://github.com/B-UMMI/LMAS
- name: ABySS
  role: One of the 11 benchmarked de novo assemblers; found to perform relatively
    poorly on complex samples
  repo: https://github.com/BirolLab/abyss
- name: MetaHipMer2
  role: Benchmarked assembler identified as a poor performer alongside ABySS, minia,
    and VelvetOptimiser
- name: minia
  role: Benchmarked assembler (GATB-Minia Pipeline) performing relatively poorly on
    complex samples
  repo: https://github.com/GATB/gatb-minia-pipeline
- name: VelvetOptimiser
  role: Benchmarked assembler found to perform relatively poorly and recommended for
    use with caution on complex samples
  repo: https://github.com/tseemann/VelvetOptimiser
- name: InSilicoSeq
  role: Recommended simulator for generating in silico mock-community reads (ENN,
    EMS, LNN, LHS datasets)
  repo: https://github.com/HadrienG/InSilicoSeq
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:66942721aa9c40a39069e35dea580b9e8d9d1e5fd2279c486309c192b6e54161
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - ABySS
  - InSilicoSeq
  - LMAS
  - MetaHipMer2
  - MetaHipmer2
  - VelvetOptimiser
  - minia
  edam_operation: http://edamontology.org/operation_0526
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# mock-community-quality-assessment

## Summary

Benchmark de novo metagenomic assemblers on a defined mock community (e.g. the 12-strain BMock12 standard, SRX4901583) using LMAS, a containerized Nextflow workflow that runs assemblers in parallel and scores global (Contigs, Nx, Mapped reads, Misassembled contigs) and per-reference metrics (Contiguity, Multiplicity, Validity, Parsimony, Identity, Breadth of coverage, Lx, NAx, NGx) against a multi-fasta of reference replicons.

## When to use

Use when comparing de novo short-read assembler performance on a known mock community standard with reference genomes available, to judge which assemblers produce accurate, contiguous assemblies.

## When NOT to use

- The input is a single isolate genome without a mock community reference set — LMAS's benchmarking logic assumes defined communities with known replicons
- Strain-level (SNP-level) discrimination is the goal — the article reports meaningful strain resolution was not achieved, even by the best assemblers tested
- Long-read or hybrid assembly is required — LMAS evaluates short-read metagenomic prokaryotic de novo assembly only

## Inputs

- Paired-end Illumina FASTQ reads from a defined mock community standard (e.g. BMock12, accession SRX4901583)
- Concatenated multi-fasta of complete reference replicons for the mock community (12 genomes for BMock12)
- Docker or Singularity container runtime and Nextflow ≥ 21.04.1

## Outputs

- Global assembly metrics table per assembler: Contigs, Basepairs, Nx, Mapped reads, Misassembled contigs, Number of Ns
- Per-reference metrics per assembler: Contiguity, Multiplicity, Validity, Parsimony, Identity, Breadth of coverage, Lx, NAx, NGx, Aligned Basepairs, Number of Ns
- Interactive HTML report summarizing assembler rankings

## How to apply

Install LMAS via conda (`conda install -c bioconda lmas`) or from github.com/B-UMMI/LMAS, ensuring Nextflow ≥ 21.04.1 with Docker or Singularity. Concatenate all complete reference replicons into a single multi-fasta for the --reference parameter, downsample mock-community reads (e.g. BMock12 to 20% of the original HiSeq 2500 reads), and run the workflow with --fastq pointing to the paired-end FASTQ files and all 11 assemblers enabled, using defaults (8 CPUs, 32 GB memory, --minLength 1000, --mapped_reads_threshold 0.75, --n_target 0.5, --l_target 0.9). The paper ran the LMAS workflow 3 times per sample with default parameters; assemblies are evaluated only after filtering contigs shorter than the --minLength cutoff. Extract global metrics and per-reference metrics from the results directory and interactive HTML report, then compare genomic versus metagenomic assemblers; the article found no significant difference between them, particularly after filtering for contigs <1 kbp, so interpret assembler superiority cautiously.

## Related tools

- **LMAS** (Nextflow pipeline that runs all assemblers in parallel and computes global and per-reference evaluation metrics in one workflow) — https://github.com/B-UMMI/LMAS
- **ABySS** (One of the 11 benchmarked de novo assemblers; found to perform relatively poorly on complex samples) — https://github.com/BirolLab/abyss
- **MetaHipMer2** (Benchmarked assembler identified as a poor performer alongside ABySS, minia, and VelvetOptimiser)
- **minia** (Benchmarked assembler (GATB-Minia Pipeline) performing relatively poorly on complex samples) — https://github.com/GATB/gatb-minia-pipeline
- **VelvetOptimiser** (Benchmarked assembler found to perform relatively poorly and recommended for use with caution on complex samples) — https://github.com/tseemann/VelvetOptimiser
- **InSilicoSeq** (Recommended simulator for generating in silico mock-community reads (ENN, EMS, LNN, LHS datasets)) — https://github.com/HadrienG/InSilicoSeq

## Examples

```
nextflow run B-UMMI/LMAS --fastq 'bmock12/*_{1,2}.fq.gz' --reference bmock12_references.fasta --minLength 1000 --mapped_reads_threshold 0.75
```

## Evaluation signals

- The output metrics table contains rows for all 11 assemblers (ABySS, GATB-Minia Pipeline, IDBA-UD, MEGAHIT, MetaHipMer2, metaSPAdes, MINIA, SKESA, SPAdes, Unicycler, VelvetOptimiser) with numeric values in the Contigs, Nx, and Breadth of coverage columns
- Assembler versions and container images used are recorded for reproducibility
- The workflow was run with the paper's default thresholds: --minLength 1000, --mapped_reads_threshold 0.75, --n_target 0.5, --l_target 0.9, 8 CPUs, 32 GB memory
- Per-reference metrics exist for each of the 12 BMock12 reference replicons per assembler
- Results are reproducible across at least 3 workflow runs, as done in the paper

## Limitations

- ABySS failed to produce an assembly for sample ERR2984773 in 1 of the runs, so some assemblers may fail intermittently
- ABySS, MetaHipMer2, minia, and VelvetOptimiser perform relatively poorly and should be used with caution when assembling complex samples
- Assembler usability may be limited with default parameters only; non-default parameter tuning was not explored
- LMAS is computationally intensive due to the de novo assembly process
- Mock communities like ZymoBIOMICS/BMock12 may not represent the metagenomic complexity of most researchers' samples; no single assembler was an undisputed all-purpose choice

## Evidence

- [results] Metagenomic vs genomic assembler comparison on BMock12: "no significant difference was observed between the genomic and metagenomic assemblers, particularly after filtering for contigs <1 kbp"
- [results] Downsampling protocol: "we downsampled to have 20% of the reads available in the original sample and further processed only these"
- [discussion] Single integrated pipeline rationale: "LMAS is the only software-integrating assembly and its evaluation into a single pipeline, guaranteeing the same conditions are met for all tools."
- [results] Repeat runs with defaults: "the LMAS workflow was run 3 times for all samples using default parameters"
- [abstract] Poor performers: "Some assemblers still in use, such as ABySS, MetaHipmer2, minia, and VelvetOptimiser, perform relatively poorly"
