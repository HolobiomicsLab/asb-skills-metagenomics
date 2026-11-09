---
name: metagenome-short-read-assembly
description: Use when the input is interleaved paired-end Illumina FASTQ from a metagenome
  or mock community and the goal is de novo reconstruction of individual strain-level
  genomes without a reference.
when_to_use_negative:
- Input reads are long third-generation (PacBio/ONT) reads — the article notes the
  high error rate of third-generation reads introduces unresolved issues, and StrainXpress
  targets Illumina-type short reads
- A high-quality reference genome is available and a reference-assisted assembly is
  preferred — StrainXpress is a de novo method aimed at avoiding reference bias
- Input FASTQ is not interleaved — the tool requires mate pairs in a single interleaved
  file of the form @S0R0/1 ... @S0R0/2
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: StrainXpress
  role: Primary de novo strain-aware metagenome assembler producing final_contigs.fasta
    from interleaved FASTQ
  repo: https://github.com/HaploKit/StrainXpress
- name: minimap2
  role: Computes all-vs-all suffix-prefix read overlaps whose CIGAR strings feed the
    weighted overlap score
  repo: https://github.com/lh3/minimap2
- name: SPAdes
  role: Competing assembler used as benchmark; optionally pre-processed with StrainXpress
    XC clustering
- name: IDBA-UD
  role: Competing assembler used as benchmark in Quast evaluations
- name: Minia3
  role: Competing assembler used as benchmark in Quast evaluations
- name: MEGAHIT
  role: Competing assembler used as benchmark in Quast evaluations
- name: Quast
  role: Evaluates assemblies, recording genome fraction, N50 and identity per sample
    per assembler
- name: metaQuast
  role: Evaluates spike-in Salmonella assemblies against known references
- name: HaploConduct
  role: Dependency of StrainXpress installed alongside minimap2
  repo: https://github.com/HaploConduct/HaploConduct
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:74aa5cc62b0d3cc1088e6395c5befe774303035fbbddee82fed4a6e8762cf5f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - HaploConduct
  - IDBA-UD
  - MEGAHIT
  - Minia3
  - Minimap2
  - Pandas
  - Python
  - Quast
  - SPAdes
  - StrainXpress
  - metaQuast
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# metagenome-short-read-assembly

## Summary

De novo, strain-aware metagenome assembly of interleaved short-read FASTQ data using StrainXpress, an overlap-layout-consensus (OLC) method that reconstructs individual strain genomes and outperforms SPAdes, IDBA-UD, Minia3 and MEGAHIT in genome fraction.

## When to use

Use when the input is interleaved paired-end Illumina FASTQ from a metagenome or mock community and the goal is de novo reconstruction of individual strain-level genomes without a reference.

## When NOT to use

- Input reads are long third-generation (PacBio/ONT) reads — the article notes the high error rate of third-generation reads introduces unresolved issues, and StrainXpress targets Illumina-type short reads
- A high-quality reference genome is available and a reference-assisted assembly is preferred — StrainXpress is a de novo method aimed at avoiding reference bias
- Input FASTQ is not interleaved — the tool requires mate pairs in a single interleaved file of the form @S0R0/1 ... @S0R0/2

## Inputs

- Interleaved paired-end FASTQ read file (all_reads.fq) from a metagenome, mock community (e.g. Bmock12, NWCs, Gut Metagenome PRJNA379884), or spike-in dataset (Salmonella PRJNA272371)

## Outputs

- stageb/final_contigs.fasta — assembled strain-specific contigs
- Quast/metaQuast metrics table (genome fraction, N50, identity) per sample per assembler, e.g. real_data_benchmark.csv or coverage_vs_accuracy.csv

## How to apply

Install StrainXpress in a conda environment (python=3.6, scipy, pandas, minimap2) and compile with 'sh install.sh'. Run 'python strainxpress.py -fq <reads>.fq' on interleaved FASTQ input; for large datasets add the '-fast' flag, which switches to the fast cluster method. Internally the pipeline proceeds through Clustering Reads, Local Assembly, and Global Assembly stages, using Minimap2 all-vs-all overlaps whose CIGAR strings are scored as D = 0.9*i + 0.1*o/((r1+r2)/2), filtering overlaps shorter than 30 bp or with identity below 0.9. Collect the assembly from the stageb folder (stageb/final_contigs.fasta). Optionally apply StrainXpress's XC clustering as a pre-processing step for competing assemblers (SPAdes, IDBA-UD, Minia3, MEGAHIT), and evaluate all assemblies with Quast or metaQuast, recording genome fraction, N50 and identity per sample per assembler.

## Related tools

- **StrainXpress** (Primary de novo strain-aware metagenome assembler producing final_contigs.fasta from interleaved FASTQ) — https://github.com/HaploKit/StrainXpress
- **minimap2** (Computes all-vs-all suffix-prefix read overlaps whose CIGAR strings feed the weighted overlap score) — https://github.com/lh3/minimap2
- **SPAdes** (Competing assembler used as benchmark; optionally pre-processed with StrainXpress XC clustering)
- **IDBA-UD** (Competing assembler used as benchmark in Quast evaluations)
- **Minia3** (Competing assembler used as benchmark in Quast evaluations)
- **MEGAHIT** (Competing assembler used as benchmark in Quast evaluations)
- **Quast** (Evaluates assemblies, recording genome fraction, N50 and identity per sample per assembler)
- **metaQuast** (Evaluates spike-in Salmonella assemblies against known references)
- **HaploConduct** (Dependency of StrainXpress installed alongside minimap2) — https://github.com/HaploConduct/HaploConduct

## Examples

```
python strainxpress.py -fq all_reads.fq -fast
```

## Evaluation signals

- stageb/final_contigs.fasta exists and is valid FASTQ/FASTA for every completed run
- Overlap pairs survive filtering with overlap length >= 30 bp, identity >= 0.9, and weighted score D in [0, 1]
- Quast genome fraction for StrainXpress on Bmock12 approaches 99.04%, at least 3.7% above all other assemblers
- On spike-in Salmonella data, genome fraction should reach ~70.26% at 5x coverage and ~84.29% from 15x coverage onwards
- Benchmark CSV contains one row per assembler-dataset combination (or per 5x–40x coverage level for the sweep) with all eight coverage levels present

## Limitations

- The short length of Illumina-type reads constrains contig length, preventing full-length strain-specific contigs that span the genomes
- No software version numbers are stated for StrainXpress or the compared tools, creating a reproducibility gap
- High error rates of third-generation sequencing reads remain unresolved by this method
- Large datasets require the '-fast' flag; the default clustering path may be too slow at scale

## Evidence

- [readme] StrainXpress is a de novo assembly method which base on overlap-layout-consensus (OLC) paradigm and can fast and accurately assemble high complexity metagenome sequencing data at strain resolution.: "StrainXpress is a de novo assembly method which base on overlap-layout-consensus (OLC) paradigm and can fast and accurately assemble high complexity metagenome sequencing"
- [readme] conda install -c bioconda python=3.6 scipy pandas minimap2: "conda install -c bioconda python=3.6 scipy pandas minimap2"
- [methods] StrainXpress computes an overlap score for each Minimap2-identified suffix-prefix read overlap directly from the CIGAR string output by Minimap2 (equation 1, a weighted combination of overlap identity and relative overlap length): [evidence span withheld — anchor status: not_found]
- [abstract] StrainXpress reconstructs strain-specific genomes from metagenomes that involve up to >1000 strains and proves to successfully deal with poorly covered strains.: "StrainXpress reconstructs strain-specific genomes from metagenomes that involve up to >1000 strains and proves to successfully deal with poorly covered strains."
- [discussion] the short length of Illumina type reads puts constraints on the length of the contigs a strain aware assembler can compute.: "the short length of Illumina type reads puts constraints on the length of the contigs a strain aware assembler can compute"
