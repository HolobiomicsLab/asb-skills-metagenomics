---
name: reference-based-assembly-evaluation
description: Use when benchmarking de novo metagenomic assembly performance on a defined
  mock community where complete reference genomes/replicons are known and per-reference
  accuracy, contiguity, and completeness metrics are required.
when_to_use_negative:
- The community is not a defined mock community and no complete reference genomes/replicons
  are available — reference-based metrics (Contiguity, Validity, Identity, Breadth
  of coverage) cannot be computed
- The goal is assembling a eukaryotic genome or a bacterial isolate rather than benchmarking
  metagenomic short-read de novo assemblers
- SNP-level strain resolution is the target — the article found meaningful strain
  resolution at the single-nucleotide polymorphism level was not achieved, even by
  the best assemblers tested
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: LMAS
  role: Automated Nextflow workflow that runs the assemblers in parallel, aligns contigs
    to the concatenated reference, computes global and per-reference metrics, and
    generates the interactive HTML report
  repo: https://github.com/B-UMMI/LMAS
- name: MetaQUAST
  role: 'Prior reference-based evaluation approach cited by the article: assembly
    evaluation based on aligning contigs to a reference'
- name: ABySS
  role: One of the assemblers benchmarked within LMAS; performs relatively poorly
    and was excluded from subsequent analyses
  repo: https://github.com/BirolLab/abyss
- name: VelvetOptimiser
  role: One of the assemblers benchmarked within LMAS; performs relatively poorly
    on complex samples
  repo: https://github.com/tseemann/VelvetOptimiser
- name: InSilicoSeq
  role: Recommended tool for generating simulated sequencing data from mock community
    references when real reads are unavailable
  repo: https://github.com/HadrienG/InSilicoSeq
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:55ed139ec2da59e924d249a805941b7c1ebab28339a857f96564adff21c94369
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - ABySS
  - InSilicoSeq
  - LMAS
  - MetaHipmer2
  - MetaQUAST
  - VelvetOptimiser
  - minia
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# reference-based-assembly-evaluation

## Summary

Evaluates de novo metagenomic assemblies against known reference replicons using LMAS, which aligns assembler contigs to a concatenated multi-fasta reference and reports global and per-reference metrics (Contiguity, Multiplicity, Validity, Identity, Breadth of coverage, Lx, NAx, NGx) for benchmarking assemblers on defined mock communities such as BMock12.

## When to use

Use when benchmarking de novo metagenomic assembly performance on a defined mock community where complete reference genomes/replicons are known and per-reference accuracy, contiguity, and completeness metrics are required.

## When NOT to use

- The community is not a defined mock community and no complete reference genomes/replicons are available — reference-based metrics (Contiguity, Validity, Identity, Breadth of coverage) cannot be computed
- The goal is assembling a eukaryotic genome or a bacterial isolate rather than benchmarking metagenomic short-read de novo assemblers
- SNP-level strain resolution is the target — the article found meaningful strain resolution at the single-nucleotide polymorphism level was not achieved, even by the best assemblers tested

## Inputs

- Paired-end Illumina FASTQ reads from a mock community standard (e.g. BMock12, accession SRX4901583)
- Complete reference replicon sequences concatenated into a single multi-fasta file
- Assembler selection (e.g. all 11 LMAS assemblers)

## Outputs

- Global assembly metrics per assembler: Contigs, Basepairs, Nx, Mapped reads, Misassembled contigs, Number of Ns
- Per-reference metrics per assembler against each replicon: Contiguity, Multiplicity, Validity, Parsimony, Identity, Breadth of coverage, Lx, NAx, NGx, Aligned Basepairs, Number of Ns
- Interactive HTML report with global and reference replicon-specific performance metrics

## How to apply

Run the LMAS Nextflow workflow (Nextflow >= 21.04.1 with Docker or Singularity) passing paired-end FASTQ reads via --fastq and all complete reference replicons concatenated into a single multi-fasta file via --reference, enabling all 11 assemblers (ABySS, GATB-Minia Pipeline, IDBA-UD, MEGAHIT, MetaHipMer2, metaSPAdes, MINIA, SKESA, SPAdes, Unicycler, VelvetOptimiser) which assemble in parallel in their own containers. Apply default filters: --minLength 1000 to exclude contigs below 1 kbp, --mapped_reads_threshold 0.75 for read-to-contig mapping, and per-reference thresholds --n_target 0.5 and --l_target 0.9. LMAS aligns contigs to the reference and computes global metrics (Contigs, Basepairs, Nx, Mapped reads, Misassembled contigs, Number of Ns) plus per-reference metrics, classifying misassemblies into categories such as chimera, insertion, deletion, inversion, rearrangement, translocation, and duplication. Results are written to the results directory and an interactive HTML report from which per-assembler, per-replicon metric tables can be extracted (e.g. into a CSV) for downstream comparison of metagenomic-specific versus traditional genomic assemblers.

## Related tools

- **LMAS** (Automated Nextflow workflow that runs the assemblers in parallel, aligns contigs to the concatenated reference, computes global and per-reference metrics, and generates the interactive HTML report) — https://github.com/B-UMMI/LMAS
- **MetaQUAST** (Prior reference-based evaluation approach cited by the article: assembly evaluation based on aligning contigs to a reference)
- **ABySS** (One of the assemblers benchmarked within LMAS; performs relatively poorly and was excluded from subsequent analyses) — https://github.com/BirolLab/abyss
- **VelvetOptimiser** (One of the assemblers benchmarked within LMAS; performs relatively poorly on complex samples) — https://github.com/tseemann/VelvetOptimiser
- **InSilicoSeq** (Recommended tool for generating simulated sequencing data from mock community references when real reads are unavailable) — https://github.com/HadrienG/InSilicoSeq

## Examples

```
nextflow run B-UMMI/LMAS --fastq 'bmock12_20pct_R{1,2}.fastq.gz' --reference bmock12_references.fasta -profile docker
```

## Evaluation signals

- Output metric table contains rows for all enabled assemblers (11 by default) with numeric values in the Contigs, Nx, and Breadth of coverage columns
- Assembler versions and container images are recorded for reproducibility of every assembly
- Per-reference metrics are present for each assembler against each reference replicon in the multi-fasta (e.g. 12 replicons for BMock12)
- Contigs shorter than --minLength (default 1000 bp) are excluded from the reported metrics
- Workflow is run 3 times with default parameters to check reproducibility of the metrics across runs

## Limitations

- Assemblers such as ABySS, MetaHipmer2, minia, and VelvetOptimiser perform relatively poorly and should be used with caution when assembling complex samples; ABySS failed to produce an assembly for sample ERR2984773 in 1 of 3 runs
- No single assembler appeared an ideal choice for short-read metagenomic prokaryote replicon assembly; results may not generalize across community compositions
- Meaningful strain resolution at the single-nucleotide polymorphism level was not achieved, even by the best assemblers tested
- LMAS is computationally intensive due to the de novo assembly process, and assembler usability may be limited with the default parameters used

## Evidence

- [intro] MetaQUAST extends the original software by performing assembly evaluation based on aligning contigs to a reference: "MetaQUAST [11], a modification of QUAST, extends the original software by performing assembly evaluation based on aligning contigs to a reference"
- [discussion] Misassembled contigs are classified into 6 main categories: "Misassembled contigs are classified into 6 main categories: chimera, insertion, deletion, inversion, rearrangement, translocation, and duplication"
- [discussion] Interactive report supports global and reference-specific metric exploration: "The interactive report provides an intuitive platform for data exploration, allowing the user to easily sift through global and reference-specific performance metrics for each sample"
- [results] BMock12 downsampled to 20% of reads: "we downsampled to have 20% of the reads available in the original sample and further processed only these"
- [methods] --minLength defines the minimum contig size filter: "The **X** in the **Contigs > X** section corresponds to the minimum contig size as defined in the ``--minLength`` parameter"
