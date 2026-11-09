---
name: containerized-bioinformatics-pipeline
description: Use when benchmarking multiple de novo metagenomic assemblers on a defined
  mock community (e.g. BMock12 SRX4901583) and reproducibility across platforms via
  containerization is required.
when_to_use_negative:
- You need strain-level (SNP-level) discrimination from assembler-generated contigs
  alone — the study found this highly unlikely.
- Your goal is assembly of a single bacterial isolate with hybrid long-short reads
  — Unicycler-style isolate pipelines are outside LMAS's metagenomic mock-community
  scope.
- Assemblies are already produced and only need external evaluation without running
  assemblers under identical containerized conditions.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: LMAS
  role: Nextflow-based workflow that orchestrates parallel containerized de novo assembly
    by 11 assemblers and integrated evaluation on mock communities
  repo: https://github.com/B-UMMI/LMAS
- name: InSilicoSeq
  role: recommended simulator for generating in silico mock-community sequencing data
    as pipeline input
  repo: https://github.com/HadrienG/InSilicoSeq
- name: Shifter
  role: container runtime enabling Docker-image-based execution of the pipeline on
    HPC systems
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:69a4888c5986b775a61dbf7d34988d8c8e341a689308ec5595469e1d3b61d5b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - ABySS
  - InSilicoSeq
  - LMAS
  - MetaHipmer2
  - Shifter
  - VelvetOptimiser
  - minia
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# containerized-bioinformatics-pipeline

## Summary

Reproduce the LMAS approach of running 11 de novo assemblers (e.g. ABySS, MEGAHIT, metaSPAdes, SKESA, Unicycler, VelvetOptimiser) on a mock-community metagenome inside Docker/Singularity containers orchestrated by Nextflow ≥ 21.04.1, so that assembly and its evaluation occur in one reproducible pipeline under identical conditions for all tools.

## When to use

Use when benchmarking multiple de novo metagenomic assemblers on a defined mock community (e.g. BMock12 SRX4901583) and reproducibility across platforms via containerization is required.

## When NOT to use

- You need strain-level (SNP-level) discrimination from assembler-generated contigs alone — the study found this highly unlikely.
- Your goal is assembly of a single bacterial isolate with hybrid long-short reads — Unicycler-style isolate pipelines are outside LMAS's metagenomic mock-community scope.
- Assemblies are already produced and only need external evaluation without running assemblers under identical containerized conditions.

## Inputs

- paired-end Illumina FASTQ reads (e.g. BMock12 SRX4901583 downsampled to 20%)
- concatenated multi-fasta of complete reference replicon sequences
- Docker or Singularity container runtime; Nextflow ≥ 21.04.1

## Outputs

- per-assembler assemblies filtered at --minLength 1000 bp
- global metrics (Contigs, Basepairs, Nx, Mapped reads, Misassembled contigs, Number of Ns)
- per-reference replicon metrics (Contiguity, Multiplicity, Validity, Parsimony, Identity, Breadth of coverage, Lx, NAx, NGx, Aligned Basepairs)
- interactive HTML report

## How to apply

Install LMAS via conda (`conda install -c bioconda lmas`) or clone github.com/B-UMMI/LMAS, ensuring Nextflow ≥ 21.04.1 and a container engine (Docker or Singularity) are available. Provide paired-end FASTQ reads via --fastq and a concatenated multi-fasta of complete reference genomes via --reference; run the 11 assemblers in parallel in their own containers with default parameters (8 CPUs, 32 GB memory, --minLength 1000, --mapped_reads_threshold 0.75, --n_target 0.5, --l_target 0.9). Downsampling a large sample such as BMock12 to 20% of reads is recommended before execution. Rationale: modularity and containerization guarantee the same conditions for all tools and reproducibility across platforms, and the workflow was run 3 times per sample in the study to confirm stability.

## Related tools

- **LMAS** (Nextflow-based workflow that orchestrates parallel containerized de novo assembly by 11 assemblers and integrated evaluation on mock communities) — https://github.com/B-UMMI/LMAS
- **InSilicoSeq** (recommended simulator for generating in silico mock-community sequencing data as pipeline input) — https://github.com/HadrienG/InSilicoSeq
- **Shifter** (container runtime enabling Docker-image-based execution of the pipeline on HPC systems)

## Examples

```
conda install -c bioconda lmas && nextflow run B-UMMI/LMAS --fastq 'reads/*_{1,2}.fastq.gz' --reference references/bmock12_all.fasta -profile docker
```

## Evaluation signals

- The interactive HTML report is produced with both global and reference-specific metrics for all 11 assemblers.
- Result tables contain numeric values for Contigs, Nx, and Breadth of coverage for every assembler run.
- Assembler software versions and container identifiers are recorded for reproducibility across the Docker/Singularity containers.
- Repeated runs (the paper ran the workflow 3 times per sample with default parameters) give consistent assemblies, e.g. flagging runs where ABySS failed on ERR2984773.

## Limitations

- LMAS is computationally intensive because de novo assembly by many assemblers in parallel is expensive.
- Assembler usability may be limited with the default parameters used; non-default tuning was not explored.
- Results from ZymoBIOMICS and similar mock communities may not generalize to the metagenomic complexity of most researchers' samples.
- Individual assemblers can fail intermittently (e.g. ABySS failed to produce an assembly for ERR2984773 in one of three runs).

## Evidence

- [discussion] LMAS was built with modularity and containerization as keystones, leveraging the parallelization of processes and guaranteeing reproducibility across platforms.: "LMAS was built with modularity and containerization as keystones, leveraging the parallelization of processes and guaranteeing reproducibility across platforms."
- [discussion] LMAS is the only software-integrating assembly and its evaluation into a single pipeline, guaranteeing the same conditions are met for all tools.: "LMAS is the only software-integrating assembly and its evaluation into a single pipeline, guaranteeing the same conditions are met for all tools."
- [methods] The input sequencing data is assembled in parallel, resources permitting, by the set of assemblers in LMAS: "The input sequencing data is assembled in parallel, resources permitting, by the set of assemblers in LMAS"
- [abstract] the workflow and its documentation are available at https://github.com/B-UMMI/LMAS and https://lmas.readthedocs.io/, respectively.: "the workflow and its documentation are available at https://github.com/B-UMMI/LMAS and https://lmas.readthedocs.io/, respectively."
- [results] we downsampled to have 20% of the reads available in the original sample and further processed only these: "we downsampled to have 20% of the reads available in the original sample and further processed only these"
