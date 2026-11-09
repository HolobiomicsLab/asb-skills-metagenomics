---
name: metagenome-de-novo-assembly-pipeline-execution
description: Use when you need to assemble individual strain-level genomes de novo
  from Illumina metagenomic short reads (interleaved FASTQ) and compare reconstruction
  quality against assemblers.
when_to_use_negative:
- Input reads are third-generation (PacBio/ONT) long reads — the article states the
  high Error Rate of third-generation reads introduces unresolved issues
- The goal is reference-assisted assembly rather than de novo, reference-bias-free
  reconstruction
- The input is already assembled contigs rather than raw short reads
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: StrainXpress
  role: Primary de novo strain-aware assembler run on interleaved FASTQ; output contigs
    taken from the stageb folder
  repo: https://github.com/HaploKit/StrainXpress
- name: IDBA-UD
  role: Competitor de novo assembler for benchmarking against StrainXpress
- name: SPAdes
  role: Competitor de novo assembler; second-highest scoring in the benchmark (74.74%
    Genome Fraction on low-complexity 2x250 bp data)
- name: MetaSPAdes
  role: Competitor de novo assembler run on all nine CAMISIM datasets
- name: GATB-Minia
  role: Competitor de novo assembler (Minia3/GATB-Minia) for benchmarking
- name: MEGAHIT
  role: Competitor de novo assembler; noted as outperforming others in contig length
    (N50)
- name: Quast
  role: Evaluation tool computing Genome Fraction, NGA50, N50, Error Rate, misassembled
    contig rate and N rate against the concatenated reference
- name: minimap2
  role: Dependency of StrainXpress installed via conda in the execution environment
  repo: https://github.com/lh3/minimap2
- name: HaploConduct
  role: Dependency of StrainXpress compiled via install.sh
  repo: https://github.com/HaploConduct/HaploConduct
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:40fc14b0a47c85e8fcbfb645df93dc6c1876c271da41e779871439cfc4858ea0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - GATB-Minia
  - HaploConduct
  - IDBA-UD
  - MEGAHIT
  - MetaSPAdes
  - Quast
  - SPAdes
  - StrainXpress
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 20
---

# metagenome-de-novo-assembly-pipeline-execution

## Summary

Executes a de novo, strain-aware metagenome assembly benchmark by running StrainXpress (and competitor assemblers IDBA-UD, SPAdes, MetaSPAdes, GATB-Minia, MEGAHIT) on interleaved Illumina FASTQ inputs and scoring contig sets with Quast metrics such as Genome Fraction, NGA50, and Error Rate.

## When to use

Use when you need to assemble individual strain-level genomes de novo from Illumina metagenomic short reads (interleaved FASTQ) and compare reconstruction quality against state-of-the-art assemblers.

## When NOT to use

- Input reads are third-generation (PacBio/ONT) long reads — the article states the high Error Rate of third-generation reads introduces unresolved issues
- The goal is reference-assisted assembly rather than de novo, reference-bias-free reconstruction
- The input is already assembled contigs rather than raw short reads

## Inputs

- Interleaved FASTQ file of Illumina metagenomic reads (all_reads.fq)
- Concatenated strain-genome reference FASTA for Quast evaluation
- Nine CAMISIM-simulated datasets spanning low/medium/high complexity and three read lengths (e.g. 2x250 bp)

## Outputs

- final_contigs.fasta from the stageb folder per assembler per dataset
- quast_benchmark_table.csv with six Quast metrics (Genome Fraction, NGA50, N50, Error Rate, misassembled contig rate, N rate) for all 6 assemblers x 9 datasets

## How to apply

Set up a conda environment (python 3.6 with scipy, pandas, minimap2), clone the StrainXpress repository and compile it with install.sh, installing boost and g++ >= 5.5.0 from conda-forge if system versions are insufficient. For each of the nine CAMISIM-simulated datasets (low/medium/high complexity x three read lengths), run 'python strainxpress.py -fq all_reads.fq' on the interleaved FASTQ, adding the -fast flag for large, high-complexity datasets, and retrieve final_contigs.fasta from the stageb folder. Run each competitor assembler (IDBA-UD, SPAdes, MetaSPAdes, GATB-Minia, MEGAHIT) on the same nine datasets to obtain comparable contig sets. Then run Quast for every contig set against the concatenated strain-genome reference to compute Genome Fraction, NGA50, N50, Error Rate, misassembled contig rate, and N rate, and aggregate the per-assembler, per-dataset metrics into a single benchmark table (quast_benchmark_table.csv).

## Related tools

- **StrainXpress** (Primary de novo strain-aware assembler run on interleaved FASTQ; output contigs taken from the stageb folder) — https://github.com/HaploKit/StrainXpress
- **IDBA-UD** (Competitor de novo assembler for benchmarking against StrainXpress)
- **SPAdes** (Competitor de novo assembler; second-highest scoring in the benchmark (74.74% Genome Fraction on low-complexity 2x250 bp data))
- **MetaSPAdes** (Competitor de novo assembler run on all nine CAMISIM datasets)
- **GATB-Minia** (Competitor de novo assembler (Minia3/GATB-Minia) for benchmarking)
- **MEGAHIT** (Competitor de novo assembler; noted as outperforming others in contig length (N50))
- **Quast** (Evaluation tool computing Genome Fraction, NGA50, N50, Error Rate, misassembled contig rate and N rate against the concatenated reference)
- **minimap2** (Dependency of StrainXpress installed via conda in the execution environment) — https://github.com/lh3/minimap2
- **HaploConduct** (Dependency of StrainXpress compiled via install.sh) — https://github.com/HaploConduct/HaploConduct

## Examples

```
python strainxpress.py -fq all_reads.fq
```

## Evaluation signals

- quast_benchmark_table.csv contains all six Quast metrics for every assembler x dataset combination (6 assemblers x 9 datasets)
- StrainXpress Genome Fraction on the low-complexity 2x250 bp dataset reaches 93.45%, exceeding SPAdes' 74.74% (second-highest de novo assembler)
- final_contigs.fasta exists in the stageb folder for every StrainXpress run
- StrainXpress ranks first in identity (99.93% vs IDBA-UD 99.91%) on comparable data
- Average improvement in reconstructed strain-specific sequence across all datasets is about 26.75% over state-of-the-art approaches

## Limitations

- Short length of Illumina-type reads constrains contig length, preventing full-length strain-specific contigs that span entire genomes
- High error rate of third-generation sequencing reads remains an unresolved issue
- No specific version numbers for StrainXpress or competitor tools are stated, a reproducibility gap
- The -fast flag is recommended only for large, high-complexity datasets

## Evidence

- [results] Genome fraction on low-complexity data: "StrainXpress: 93.45% versus 74.74% by SPAdes, as the second highest scoring"
- [intro] Assembler comparisons named: "A non-exhaustive selection of state-of-the-art approaches encompasses IDBA-UD (6), SPAdes (7), Minia3 (8) and MEGAHIT (9)."
- [results] Quast used for evaluation: "Quantities referring to the criteria listed in the corresponding Tables were determined using Quast (40)"
- [readme] README installation commands: "conda install -c bioconda python=3.6 scipy pandas minimap2"
- [readme] README example invocation: "python ../scripts/strainxpress.py -fq all_reads.fq"
- [readme] README input format: "The input file must be interleaved FASTQ and format like below:"
- [abstract] Average improvement claim: "The amount of reconstructed strain-specific sequence exceeds that of the current state-of-the-art approaches by on average 26.75% across all data sets"
