---
name: global-assembly-metrics-computation
description: Use when you have paired-end metagenomic FASTQ reads from a defined mock
  community plus reference replicons, and need comparable global assembly quality
  metrics across multiple assemblers.
when_to_use_negative:
- You need per-reference-replicon metrics (Contiguity, Multiplicity, Validity, Lx,
  NAx, NGx) rather than sample-level global metrics — use LMAS's per-reference module
  instead
- Assemblies were produced outside LMAS with heterogeneous parameters — comparisons
  would not satisfy LMAS's guarantee that 'the same conditions are met for all tools'
- You are evaluating long-read or hybrid assemblies; LMAS benchmarks short-read de
  novo assembly only
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: LMAS
  role: Nextflow workflow that runs all assemblers in parallel and computes global
    metrics (Contigs, Nx, Mapped reads, Misassembled contigs) under identical conditions
  repo: https://github.com/B-UMMI/LMAS
- name: ABySS
  role: One of 11 assemblers benchmarked; produces the assembly whose global metrics
    are computed (noted as a poor performer)
  repo: https://github.com/BirolLab/abyss
- name: minia
  role: Assembler (via GATB-Minia Pipeline) whose output contigs feed the global metrics
    computation
  repo: https://github.com/GATB/gatb-minia-pipeline
- name: VelvetOptimiser
  role: Assembler benchmarked; its assemblies are evaluated with the global metrics
    but excluded from subsequent analyses due to poor performance
  repo: https://github.com/tseemann/VelvetOptimiser
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:5e4d88ea9e4c70db4423d0ed6a23c1ad47398000b38e8ea6971d9a550c61913b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - ABySS
  - LMAS
  - MetaHipmer2
  - VelvetOptimiser
  - minia
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# global-assembly-metrics-computation

## Summary

Compute assembly-wide (global) benchmarking metrics — Contigs, Basepairs, Nx, Mapped reads, Misassembled contigs, and Number of Ns — for each de novo assembler run within the LMAS Nextflow pipeline, enabling side-by-side comparison of traditional vs metagenomic assemblers on mock communities such as BMock12.

## When to use

Use when you have paired-end metagenomic FASTQ reads from a defined mock community plus reference replicons, and need comparable global assembly quality metrics across multiple assemblers.

## When NOT to use

- You need per-reference-replicon metrics (Contiguity, Multiplicity, Validity, Lx, NAx, NGx) rather than sample-level global metrics — use LMAS's per-reference module instead
- Assemblies were produced outside LMAS with heterogeneous parameters — comparisons would not satisfy LMAS's guarantee that 'the same conditions are met for all tools'
- You are evaluating long-read or hybrid assemblies; LMAS benchmarks short-read de novo assembly only

## Inputs

- Paired-end Illumina FASTQ reads from a mock community (e.g., BMock12 accession SRX4901583, downsampled to 20% of original reads)
- Multi-fasta reference file of concatenated complete replicon/genome sequences passed via --reference

## Outputs

- Per-assembler global metrics table: Contigs, Basepairs, Nx, Mapped reads, Misassembled contigs, Number of Ns
- Assembler FASTA assemblies filtered at --minLength 1000 bp
- Interactive HTML report with global and reference-specific metrics

## How to apply

Run LMAS (via `conda install -c bioconda lmas` or the github.com/B-UMMI/LMAS Nextflow workflow, requiring Nextflow ≥ 21.04.1 with Docker or Singularity) with `--fastq` pointing to the paired-end reads and `--reference` to a multi-fasta of concatenated complete genomes, using default parameters (8 CPUs, 32 GB memory, --minLength 1000, --mapped_reads_threshold 0.75). Assemblies are produced in parallel by up to 11 assemblers (ABySS, GATB-Minia Pipeline, IDBA-UD, MEGAHIT, MetaHipMer2, metaSPAdes, MINIA, SKESA, SPAdes, Unicycler, VelvetOptimiser), each in its own container, so all tools run under identical conditions. For each assembler output the workflow computes global metrics — Contigs, Basepairs, Nx, Mapped reads, Misassembled contigs, Number of Ns — where contigs shorter than --minLength (1000 bp by default) are excluded and a read is considered mapped when at least the --mapped_reads_threshold fraction (0.75) aligns to a contig. Collect the global metrics for every assembler from the results directory and interactive HTML report, and repeat the run (the paper ran the workflow 3 times per sample) to assess run-to-run stability.

## Related tools

- **LMAS** (Nextflow workflow that runs all assemblers in parallel and computes global metrics (Contigs, Nx, Mapped reads, Misassembled contigs) under identical conditions) — https://github.com/B-UMMI/LMAS
- **ABySS** (One of 11 assemblers benchmarked; produces the assembly whose global metrics are computed (noted as a poor performer)) — https://github.com/BirolLab/abyss
- **minia** (Assembler (via GATB-Minia Pipeline) whose output contigs feed the global metrics computation) — https://github.com/GATB/gatb-minia-pipeline
- **VelvetOptimiser** (Assembler benchmarked; its assemblies are evaluated with the global metrics but excluded from subsequent analyses due to poor performance) — https://github.com/tseemann/VelvetOptimiser

## Examples

```
nextflow run B-UMMI/LMAS --fastq 'BMock12_20pct/*_{1,2}.fq.gz' --reference bmock12_references.fasta --minLength 1000 --mapped_reads_threshold 0.75 -with-docker
```

## Evaluation signals

- Output table contains a row for each of the 11 assemblers with numeric values in Contigs, Basepairs, Nx, Mapped reads, Misassembled contigs, and Number of Ns columns
- Contig counts reflect the --minLength 1000 filter (contigs <1000 bp excluded)
- Mapped reads reflect the --mapped_reads_threshold 0.75 alignment criterion
- Assembler versions and container identifiers are recorded for reproducibility
- Workflow repeated 3 times per sample yields stable metric values across runs

## Limitations

- ABySS failed to produce an assembly for one sample (ERR2984773) in 1 of 3 runs, so global metrics can be missing for some assembler/sample combinations
- Global metrics alone cannot resolve strain-level (SNP-level) differences; no assembler achieved meaningful strain resolution
- Results reflect default LMAS parameters only; assembler performance under non-default parameter tuning was not explored
- Misassembled-contig classification is categorical (chimera, insertion, deletion, inversion, rearrangement, translocation, duplication) and computationally intensive

## Evidence

- [results] the LMAS workflow was run 3 times for all samples using default parameters: "the LMAS workflow was run 3 times for all samples using default parameters"
- [methods] min contig filter: "The **X** in the **Contigs > X** section corresponds to the minimum contig size as defined in the ``--minLength`` parameter"
- [methods] mapped reads threshold: "--mapped_reads_threshold                Value for the minimum percentage of a read aligning to the"
- [discussion] single pipeline guarantee: "LMAS is the only software-integrating assembly and its evaluation into a single pipeline, guaranteeing the same conditions are met for all tools."
- [results] ABySS failure: "ABySS failed to produce an assembly for sample ERR2984773 for 1 of the runs"
