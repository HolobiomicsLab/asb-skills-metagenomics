---
name: interleaved-fastq-read-processing
description: Use when the input is paired-end metagenome sequencing data that must
  be converted to interleaved FASTQ and assembled de novo at strain resolution with
  StrainXpress, or when reproducing StrainXpress benchmark workflows.
when_to_use_negative:
- Input reads are long-read (PacBio/Nanopore) — the article notes the high Error Rate
  of third-generation sequencing reads 'introduce a range of issues that still need
  to be resolved'
- The task requires species-level binning of an existing assembly rather than de novo
  strain-aware assembly from raw reads
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: StrainXpress
  role: Performs the three-stage (Clustering Reads, Local Assembly, Global Assembly)
    strain-aware de novo assembly on interleaved FASTQ input
  repo: https://github.com/HaploKit/StrainXpress
- name: minimap2
  role: Dependency used by StrainXpress, installed via bioconda in the environment
    setup
  repo: https://github.com/lh3/minimap2
- name: Quast
  role: Evaluates final_contigs.fasta against concatenated strain-genome references
    to compute genome fraction, N50, NGA50, Error Rate
- name: metaQuast
  role: Evaluates spike-in Salmonella contigs against known references across 5x–40x
    coverage levels
provenance:
  source_task_ids:
  - task_004
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:ba816892f055d3f2c87341d44763921bb78aa91b0c73e92e1122e00cfd291068
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - GATB-Minia
  - IDBA-UD
  - MEGAHIT
  - MetaSPAdes
  - Minia3
  - Quast
  - SPAdes
  - StrainXpress
  - metaQuast
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# interleaved-fastq-read-processing

## Summary

Format and run StrainXpress on interleaved paired-end FASTQ metagenome read sets (e.g. CAMISIM simulations, Bmock12, NWCs, Gut Metagenome, Salmonella spike-ins) using a conda-installed StrainXpress environment, producing strain-resolved contigs in stageb/final_contigs.fasta.

## When to use

Use when the input is paired-end metagenome sequencing data that must be converted to interleaved FASTQ and assembled de novo at strain resolution with StrainXpress, or when reproducing StrainXpress benchmark workflows.

## When NOT to use

- Input reads are long-read (PacBio/Nanopore) — the article notes the high Error Rate of third-generation sequencing reads 'introduce a range of issues that still need to be resolved'
- The task requires species-level binning of an existing assembly rather than de novo strain-aware assembly from raw reads

## Inputs

- Interleaved paired-end FASTQ file (all_reads.fq) with /1 and /2 read pairs as consecutive records
- Paired-end Illumina read sets to be merged, e.g. Bmock12 (SRX4901583), NWCs, Gut Metagenome (PRJNA379884), Salmonella spike-ins (PRJNA272371)

## Outputs

- stageb/final_contigs.fasta — strain-resolved de novo contigs per dataset
- Quast/metaQuast metrics (genome fraction, N50, NGA50, Error Rate) per assembler-dataset combination for benchmark CSV tables

## How to apply

Set up the environment with conda (python=3.6, scipy, pandas, minimap2 via bioconda), clone the StrainXpress repository, and compile with 'sh install.sh', ensuring g++ >=5.5.0 and boost libraries are available. StrainXpress requires interleaved paired-end FASTQ input (paired reads as consecutive records with /1 and /2 suffixes, e.g. @S0R0/1 followed by @S0R0/2), so merge forward/reverse FASTQ files into a single all_reads.fq before running. Invoke 'python strainxpress.py -fq all_reads.fq' for standard datasets; add the '-fast' flag for large or high-complexity datasets such as the CAMISIM high-complexity set (1057 strains, 376 species) as recommended by the README. Collect the assembly output from the stageb folder as final_contigs.fasta, then validate assembly quality against references with Quast (genome fraction, N50, Error Rate) or metaQuast for the Salmonella spike-in datasets.

## Related tools

- **StrainXpress** (Performs the three-stage (Clustering Reads, Local Assembly, Global Assembly) strain-aware de novo assembly on interleaved FASTQ input) — https://github.com/HaploKit/StrainXpress
- **minimap2** (Dependency used by StrainXpress, installed via bioconda in the environment setup) — https://github.com/lh3/minimap2
- **Quast** (Evaluates final_contigs.fasta against concatenated strain-genome references to compute genome fraction, N50, NGA50, Error Rate)
- **metaQuast** (Evaluates spike-in Salmonella contigs against known references across 5x–40x coverage levels)

## Examples

```
python strainxpress.py -fq all_reads.fq -fast
```

## Evaluation signals

- Output directory contains stageb/final_contigs.fasta after each run
- Input FASTQ is genuinely interleaved: every /1 record is immediately followed by its /2 mate with matching IDs
- On low-complexity 2×250 bp CAMISIM data, Quast genome fraction should reach ~93.45% (vs. 74.74% for SPAdes)
- On Bmock12, genome fraction should approach 99.04%, exceeding other assemblers by ≥3.7%
- Coverage-sweep runs reproduce metaQuast genome fractions such as 70.26% at 5x and ≥84.29% from 15x spike-in coverage

## Limitations

- Python 3.6 environment required; compilation needs g++ >=5.5.0 with boost libraries, and tool version numbers for competitors are not stated in the paper
- Short Illumina read length constrains contig length, preventing full-length strain-specific contigs
- Very large/high-complexity datasets require the -fast clustering flag to be tractable

## Evidence

- [readme] StrainXpress is built for linux-based systems and python3 only: "Please note that StrainXpress is built for linux-based systems and python3 only."
- [readme] conda install python 3.6 scipy pandas minimap2: "conda install -c bioconda python=3.6 scipy pandas minimap2"
- [readme] input must be interleaved FASTQ: "The input file must be interleaved FASTQ and format like below:"
- [readme] fast cluster method for big datasets: "When a data set is big, we recommend to use the fast cluster method:"
- [readme] result in stageb folder: "The result is in the stageb folder:"
- [results] genome fraction 93.45 vs 74.74 SPAdes: "StrainXpress: 93.45% versus 74.74% by SPAdes, as the second highest scoring"
