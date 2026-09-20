---
name: metagenomic-assembly-benchmarking
description: Use when benchmarking multiple de novo assemblers on a known mock-community
  metagenomic sample with reference replicons available, to compare global metrics
  (Nx, contigs, mapped reads) and per-reference metrics (contiguity, multiplicity,
  validity, Lx, NAx, NGx).
when_to_use_negative:
- Assembling a eukaryotic genome or a single bacterial isolate (e.g., Unicycler is
  designed exclusively for bacterial isolates, and LMAS targets metagenomic mock communities)
- You lack known reference replicons for the community and only reference-free contig
  statistics (e.g., QUAST-style) are needed
- You need strain-level SNP resolution from assemblies — the study found 'Meaningful
  strain resolution at the single-nucleotide polymorphism level was not achieved,
  even by the best assemblers tested.'
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: LMAS
  role: Nextflow benchmarking workflow that runs assemblers in parallel, filters contigs
    (BBTools), maps to references (minimap2), and computes metrics
  repo: https://github.com/B-UMMI/LMAS
- name: ABySS
  role: Benchmarked genomic de novo assembler within LMAS
  repo: https://github.com/BirolLab/abyss
- name: MetaHipMer2
  role: Benchmarked metagenomic de novo assembler within LMAS
- name: minia
  role: Benchmarked genomic de novo assembler within LMAS (GATB-Minia Pipeline)
  repo: https://github.com/GATB/gatb-minia-pipeline
- name: VelvetOptimiser
  role: Benchmarked genomic de novo assembler within LMAS
  repo: https://github.com/tseemann/VelvetOptimiser
- name: BBTools
  role: v38.44 sequence-length filter producing the FS contig set at --minLength cutoff
- name: minimap2
  role: v2.22 alignment of FS contigs to reference replicons for reference-dependent
    metrics
- name: InSilicoSeq
  role: Recommended simulator for generating in-silico mock community reads
  repo: https://github.com/HadrienG/InSilicoSeq
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:2f6dec6c49450acce089fc06b92a79a262a3080dbff1b640b4c4cb643f6ebe3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - ABySS
  - BBTools
  - InSilicoSeq
  - LMAS
  - MetaHipMer2
  - MetaHipmer2
  - VelvetOptimiser
  - minia
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# metagenomic-assembly-benchmarking

## Summary

Benchmarks traditional and metagenomic short-read de novo assemblers on defined mock communities (e.g., BMock12 SRX4901583, ZymoBIOMICS) using the LMAS Nextflow workflow, which parallelizes assembly across containerized assemblers and computes global and per-reference quality metrics in an interactive HTML report.

## When to use

Use when benchmarking multiple de novo assemblers on a known mock-community metagenomic sample with reference replicons available, to compare global metrics (Nx, contigs, mapped reads) and per-reference metrics (contiguity, multiplicity, validity, Lx, NAx, NGx).

## When NOT to use

- Assembling a eukaryotic genome or a single bacterial isolate (e.g., Unicycler is designed exclusively for bacterial isolates, and LMAS targets metagenomic mock communities)
- You lack known reference replicons for the community and only reference-free contig statistics (e.g., QUAST-style) are needed
- You need strain-level SNP resolution from assemblies — the study found 'Meaningful strain resolution at the single-nucleotide polymorphism level was not achieved, even by the best assemblers tested.'

## Inputs

- Paired-end Illumina FASTQ reads from a defined mock community (e.g., BMock12 accession SRX4901583, HiSeq 2500; ZymoBIOMICS ERR2984773/ERR2935805; Gut-Mix-RR SRR11487941; Gut-Mix-HiLo SRR11487935)
- Single multi-FASTA file of concatenated complete reference replicon sequences (passed via --reference)
- Optional: user-defined mock community reference genomes and simulated reads generated with InSilicoSeq

## Outputs

- Per-assembler contig FASTA files under results/assembly/<assembler>/
- minimap2 alignment files (one per sample) of BBTools-filtered (FS) contigs against reference replicons
- Interactive HTML report with global metrics (Contigs, Basepairs, Nx, Mapped reads, Misassembled contigs, Number of Ns) and per-reference metrics (Contiguity, Multiplicity, Validity, Parsimony, Identity, Breadth of coverage, Lx, NAx, NGx, Aligned Basepairs)
- Per-assembler performance records (version, container, execution time, CPU/hour, max memory, data read/written) from the report Performance tab

## How to apply

Install LMAS via conda (`conda install -c bioconda lmas`) or from github.com/B-UMMI/LMAS, requiring Nextflow ≥21.04.1 with Docker or Singularity. Provide paired-end FASTQ via --fastq and a single multi-FASTA of concatenated reference replicons via --reference, keeping default thresholds (--minLength 1000, --mapped_reads_threshold 0.75, --n_target 0.5, --l_target 0.9); for BMock12 (SRX4901583), downsample to 20% of reads as in the paper. Assemblers (ABySS, GATB-Minia Pipeline, IDBA-UD, MEGAHIT, MetaHipMer2, metaSPAdes, MINIA, SKESA, SPAdes, Unicycler, VelvetOptimiser) run in parallel in their own containers; contigs are then filtered to the FS set with BBTools v38.44 at the --minLength cutoff and mapped to references with minimap2 v2.22. Extract global metrics (Contigs, Basepairs, Nx, Mapped reads, Misassembled contigs, Number of Ns) and per-reference metrics (Contiguity, Multiplicity, Validity, Parsimony, Identity, Breadth of coverage, Lx, NAx, NGx, Aligned Basepairs) from the results directory and HTML report. Rationale: the paper's protocol ran LMAS 3 times per sample with default parameters, and noted that differences between genomic and metagenomic assemblers tended to be attenuated after filtering contigs <1 kbp.

## Related tools

- **LMAS** (Nextflow benchmarking workflow that runs assemblers in parallel, filters contigs (BBTools), maps to references (minimap2), and computes metrics) — https://github.com/B-UMMI/LMAS
- **ABySS** (Benchmarked genomic de novo assembler within LMAS) — https://github.com/BirolLab/abyss
- **MetaHipMer2** (Benchmarked metagenomic de novo assembler within LMAS)
- **minia** (Benchmarked genomic de novo assembler within LMAS (GATB-Minia Pipeline)) — https://github.com/GATB/gatb-minia-pipeline
- **VelvetOptimiser** (Benchmarked genomic de novo assembler within LMAS) — https://github.com/tseemann/VelvetOptimiser
- **BBTools** (v38.44 sequence-length filter producing the FS contig set at --minLength cutoff)
- **minimap2** (v2.22 alignment of FS contigs to reference replicons for reference-dependent metrics)
- **InSilicoSeq** (Recommended simulator for generating in-silico mock community reads) — https://github.com/HadrienG/InSilicoSeq

## Examples

```
nextflow run B-UMMI/LMAS --fastq 'bmock12/*_{1,2}.fastq.gz' --reference bmock12_references.fasta --minLength 1000 -with-docker
```

## Evaluation signals

- Output report/CSV contains rows for all 11 assemblers with numeric values in Contigs, Nx, and Breadth of coverage columns
- Assembler versions and container identifiers are recorded for every assembler in the results
- A valid minimap2 alignment file exists for every sample, and all retained FS contigs satisfy the --minLength threshold (default 1000 bp)
- Resource table includes all five Performance-tab metrics (Avg Time, CPU/Hour, Max Memory (GB), Average Read (GB), Average Write (GB)) for every assembler
- Repeated runs (paper used 3 runs per sample with default parameters) should give consistent assembly metrics; inconsistent runtime/memory for ABySS and MetaHipMer2 is a known instability flag

## Limitations

- ABySS failed to produce an assembly for sample ERR2984773 in 1 of 3 runs; ABySS and MetaHipMer2 showed inconsistent resource requirements across repeated runs and were excluded from subsequent analyses
- LMAS is computationally intensive due to the de novo assembly process; k-mer De Bruijn graph assemblers outperformed alternatives but at greater computational cost
- Assembler usability may be limited with the default parameters used; non-default parameter tuning was not explored
- ZymoBIOMICS and similar mock communities may not represent the metagenomic complexity of most researchers' samples; no single assembler was an undisputed all-purpose choice

## Evidence

- [abstract] LMAS (Last Metagenomic Assembler Standing) was developed as a flexible platform allowing users to evaluate assembler performance given known standard communities.: "LMAS (Last Metagenomic Assembler Standing) was developed as a flexible platform allowing users to evaluate assembler performance given known standard communities."
- [results] we downsampled to have 20% of the reads available in the original sample: "we downsampled to have 20% of the reads available in the original sample and further processed only these"
- [results] no significant difference was observed between the genomic and metagenomic assemblers, particularly after filtering for contigs <1 kbp: "no significant difference was observed between the genomic and metagenomic assemblers, particularly after filtering for contigs <1 kbp"
- [abstract] k-mer De Bruijn graph assemblers outperformed the alternative approaches but came with a greater computational cost: "Overall, in our test datasets, k-mer De Bruijn graph assemblers outperformed the alternative approaches but came with a greater computational cost."
- [discussion] LMAS is the only software-integrating assembly and its evaluation into a single pipeline: "LMAS is the only software-integrating assembly and its evaluation into a single pipeline, guaranteeing the same conditions are met for all tools."
- [abstract] the workflow and its documentation are available at https://github.com/B-UMMI/LMAS and https://lmas.readthedocs.io/, respectively.: "the workflow and its documentation are available at https://github.com/B-UMMI/LMAS and https://lmas.readthedocs.io/, respectively."
