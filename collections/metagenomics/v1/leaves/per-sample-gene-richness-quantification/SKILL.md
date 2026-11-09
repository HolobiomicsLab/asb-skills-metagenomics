---
name: per-sample-gene-richness-quantification
description: Use when you need per-sample, per-species gene counts from vaginal metagenomic
  or metatranscriptomic FASTQ data to assess intraspecies gene-content diversity,
  such as testing whether multiple strains of a species co-occur in a community.
when_to_use_negative:
- Input is paired-end data that cannot or should not be merged into single-end reads
  — VIRGO's mapping module only accepts one FASTQ per sample
- The analysis goal is variant detection or nucleotide-level diversity within genes
  — VIRGO does not yet identify nucleotide variants
- The target community is dominated by non-bacterial microbes (viral/fungal) — VIRGO
  has only limited inclusion of viral and fungal gene sequences (10,908 fungal and
  15,965 phage genes)
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: VIRGO
  role: Non-redundant vaginal gene catalog and script suite (runMapping.step1.sh,
    runMapping.step2.sh) used to map reads and produce per-sample gene-count summaries
  repo: github.com/ravel-lab/VIRGO
- name: bowtie
  role: Short-read aligner used inside runMapping.step1.sh with seed length -l 25
    to map merged single-end reads to the VIRGO Bowtie database
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:8de6cd484be653c5f86d7252b4cab16b1865640fac02356013bf621ff543f967
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - VIRGO
  - bowtie
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# per-sample-gene-richness-quantification

## Summary

Quantifies the number of microbial genes per species and per sample in vaginal metagenomes by mapping reads to the VIRGO non-redundant gene catalog with bowtie, then summarizing per-sample gene counts to reveal within-community intraspecies diversity (e.g., L. crispatus metagenomes carrying 3262 ± 586 genes vs 2064 ± 225 in isolate genomes).

## When to use

Use when you need per-sample, per-species gene counts from vaginal metagenomic or metatranscriptomic FASTQ data to assess intraspecies gene-content diversity, such as testing whether multiple strains of a species co-occur in a community.

## When NOT to use

- Input is paired-end data that cannot or should not be merged into single-end reads — VIRGO's mapping module only accepts one FASTQ per sample
- The analysis goal is variant detection or nucleotide-level diversity within genes — VIRGO does not yet identify nucleotide variants
- The target community is dominated by non-bacterial microbes (viral/fungal) — VIRGO has only limited inclusion of viral and fungal gene sequences (10,908 fungal and 15,965 phage genes)

## Inputs

- Single-end merged FASTQ files of vaginal metagenomes (VIRGO does not support paired-end mapping)
- FASTQ or read sets from microbial isolate genomes (e.g., L. crispatus isolates)
- VIRGO non-redundant gene catalog (0.95 million genes) with Bowtie 1 database

## Outputs

- Per-sample .out file listing VIRGO gene IDs, mapped read counts, and gene lengths (sorted by read count)
- summary.Count.txt: number of genes per species per sample
- summary.geneRichness.txt: number of genes per sample
- summary.Abundance.txt: number of reads per species

## How to apply

First run the VIRGO test module (runTesting.step1.sh and runTesting.step2.sh) to confirm the tool and database, checking that expected outputs like summary.Count.txt and summary.geneRichness.txt are produced. Merge paired-end reads into single-end FASTQ files, since VIRGO does not yet support paired-end mapping. For each sample (e.g., the 1,507 vaginal metagenomes and L. crispatus isolate genome read sets), run runMapping.step1.sh against the VIRGO database with bowtie seed length -l 25 and a reads cutoff of 2 for calling a gene hit, producing a per-sample .out file of VIRGO gene IDs, mapped read counts, and gene lengths. Then run runMapping.step2.sh over all .out files to generate summary tables, including summary.Count.txt (genes per species per sample) and summary.geneRichness.txt (genes per sample). Filter mapped genes for presence at >=3 reads per gene, and compare per-sample gene counts between metagenomes and isolate genomes (Welch-type test) to detect strain co-occurrence, e.g., a 1.6-fold excess in L. crispatus (3262 ± 586 vs 2064 ± 225 genes, p < 0.001).

## Related tools

- **VIRGO** (Non-redundant vaginal gene catalog and script suite (runMapping.step1.sh, runMapping.step2.sh) used to map reads and produce per-sample gene-count summaries) — github.com/ravel-lab/VIRGO
- **bowtie** (Short-read aligner used inside runMapping.step1.sh with seed length -l 25 to map merged single-end reads to the VIRGO Bowtie database)

## Examples

```
./runMapping.step1.sh -r sample1.fastq -p samplePrefix -d /full/path/to/VIRGO/ && ./runMapping.step2.sh -p /path/to/output/of/step1/ -d /path/to/VIRGO/
```

## Evaluation signals

- Test-run output directory contains all expected files: summary.Abundance.txt, summary.Percentage.txt, summary.Count.txt, summary.geneRichness.txt, and sampleName.out
- Each sampleName.out file is sorted by mapped-read count with columns VIRGO gene ID, read count, gene length
- Per-sample gene counts are filtered at >=3 reads per gene before calling gene presence
- Group means reproduce published values: 3262 ± 586 genes for metagenome-derived L. crispatus vs 2064 ± 225 for isolate genomes, significant at p < 0.001 (1.6-fold excess)
- summary.geneRichness.txt contains a row with the total number of genes detected per sample

## Limitations

- VIRGO does not yet support paired-end mapping; reads must be merged into a single FASTQ, which loses mate-pair information
- Reference-based approach: metagenomic subspecies may contain non-redundant genes not observed in isolate genomes, and gaps may reflect limited isolate genomes or systematic bias in growth and recovery of vaginal species
- VIRGO does not yet identify nucleotide variants within genes; planned for future versions
- Database is primarily focused on bacteria with limited inclusion of viral and fungal gene sequences

## Evidence

- [results] L. crispatus genes identified in vaginal metagenomes contained on average 1.6 times more genes (3262 ± 586) than L. crispatus isolate genomes (2064 ± 225), a significant excess (p < 0.001): [evidence span withheld — anchor status: not_found]
- [readme] reads cutoff default is 2: "**"reads** = " default value is 2. : *in step 1 scripts, the cutoff of the number of reads for calling a gene hit.**"
- [readme] bowtie seed length default 25: "**"bowtie -l"** default value is 25 : *in step 1 scripts, the -l is the seed length for read mapping. You can go lower to get more hits or higher to get more strict in mapping quality.*"
- [readme] VIRGO does not support paired-end mapping: "*Please note VIRGO does NOT yet support paired-end mapping, please merge the paired end reads alone or together with single-end reads into one fastq file.*"
- [readme] summary.Count.txt is the number of genes per species: "- summary.Count.txt: the number of genes per species"
