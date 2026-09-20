---
name: multi-population-metagenome-coverage-benchmarking
description: Use when you need to quantify what fraction of reads from independent,
  population-diverse metagenomes map to a reference gene catalog versus a smaller
  subset reference, to assess catalog comprehensiveness.
when_to_use_negative:
- The goal is taxonomic/functional profiling of samples rather than benchmarking the
  reference catalog itself — mapping to VIRGO for abundance estimation is a different
  skill
- Your mapper or workflow requires paired-end mapping; VIRGO's runMapping.step1.sh
  does not support paired-end input, so reads must be merged first
- The held-out samples come from a body site or population far outside the catalog's
  scope (VIRGO is vagina-focused and primarily bacterial, with limited viral/fungal
  sequences), making coverage benchmarks uninterpretable
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: VIRGO
  role: Reference gene catalog and mapping/summarization scripts (runMapping.step1.sh,
    runMapping.step2.sh) used to map held-out metagenomes and compute per-sample mapping
    statistics
  repo: github.com/ravel-lab/VIRGO
- name: bowtie
  role: Short-read aligner invoked inside runMapping.step1.sh with default seed length
    -l 25 for read mapping against the VIRGO Bowtie database
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:c1d502bc1cef301bb27a90fcfa9a166c5dd948f43043b6984ae11c8a65b33160
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - VIRGO
  - bowtie
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# multi-population-metagenome-coverage-benchmarking

## Summary

Benchmark the geographic comprehensiveness of a reference gene catalog by mapping held-out metagenomes from multiple populations (North American, African, Chinese vaginal samples) against the full catalog and a subset reference, then comparing read-mapping rates. Used to validate that VIRGO provides >95% coverage of the human vaginal microbiome across continents.

## When to use

Use when you need to quantify what fraction of reads from independent, population-diverse metagenomes map to a reference gene catalog versus a smaller subset reference, to assess catalog comprehensiveness.

## When NOT to use

- The goal is taxonomic/functional profiling of samples rather than benchmarking the reference catalog itself — mapping to VIRGO for abundance estimation is a different skill
- Your mapper or workflow requires paired-end mapping; VIRGO's runMapping.step1.sh does not support paired-end input, so reads must be merged first
- The held-out samples come from a body site or population far outside the catalog's scope (VIRGO is vagina-focused and primarily bacterial, with limited viral/fungal sequences), making coverage benchmarks uninterpretable

## Inputs

- 91 held-out vaginal metagenome samples from North American, African, and Chinese cohorts (paired-end and/or single-end FASTQ)
- Complete VIRGO non-redundant gene database (0.95 million genes) with Bowtie 1 index (0_db)
- Comparison subset reference, e.g., HMP vaginal metagenome contig subset of the VIRGO database

## Outputs

- One sampleName.out file per sample (VIRGO gene ID, mapped read count, gene length) per reference set
- Summary tables from runMapping.step2.sh: summary.Abundance.txt, summary.Count.txt, summary.Percentage.txt, summary.geneRichness.txt
- mapping_rate_table.tsv: per-sample fraction of reads mapped against each reference, grouped by population (North American, African, Chinese)

## How to apply

Convert each held-out metagenome to single-end FASTQ by merging paired-end reads (VIRGO does not support paired-end mapping), optionally pooling with single-end reads. Run VIRGO's runMapping.step1.sh (-r sample.fastq -p samplePrefix -d /path/to/VIRGO) against the complete 0.95-million-gene VIRGO non-redundant database, keeping the default bowtie seed length (-l 25) and the default reads cutoff of 2 reads for calling a gene hit; this yields one sampleName.out file per sample with VIRGO gene ID, mapped read count, and gene length. Repeat the identical mapping for all samples against the comparison subset (e.g., the HMP vaginal metagenome contig subset). Run runMapping.step2.sh (-p /path/to/step1/output -d /path/to/VIRGO) to obtain per-sample summaries (summary.Abundance.txt, summary.Count.txt, summary.Percentage.txt, summary.geneRichness.txt). Finally compute the fraction of reads mapped per sample against each reference, group samples by population, and compare mapping rates between the full catalog and the subset to quantify coverage gains.

## Related tools

- **VIRGO** (Reference gene catalog and mapping/summarization scripts (runMapping.step1.sh, runMapping.step2.sh) used to map held-out metagenomes and compute per-sample mapping statistics) — github.com/ravel-lab/VIRGO
- **bowtie** (Short-read aligner invoked inside runMapping.step1.sh with default seed length -l 25 for read mapping against the VIRGO Bowtie database)

## Examples

```
./runMapping.step1.sh -r sample1.fastq -p samplePrefix -d /path/to/VIRGO/ ; ./runMapping.step2.sh -p /path/to/output/of/step1/ -d /path/to/VIRGO/
```

## Evaluation signals

- North American samples show >99% of reads mapped to the complete VIRGO dataset but only ~55% mapped to the HMP vaginal metagenome contig subset
- African samples reproduce ~96% mapping rate and Chinese samples ~88% against the complete VIRGO set, consistent with VIRGO's claimed >95% coverage across North America, Africa, and Asia
- Each sampleName.out file is sorted by the mapped-read-count column and contains three columns: VIRGO gene ID, number of reads mapped, gene length
- runMapping.step2.sh output directory contains summary.Abundance.txt, summary.Count.txt, summary.Percentage.txt, and summary.geneRichness.txt
- Genes called as hits respect the default reads cutoff of 2 mapped reads and bowtie seed length of 25 unless explicitly overridden

## Limitations

- VIRGO does not support paired-end mapping; paired-end reads must be merged into a single FASTQ before runMapping.step1.sh
- The VIRGO database is primarily focused on bacteria with limited inclusion of viral and fungal gene sequences, which caps attainable mapping rates for non-bacterial content
- VIRGO acknowledges limitations of its reference-based approach; this version focuses on gene-level de-redundancy and characterization of the vaginal microbiome

## Evidence

- [intro] we show that VIRGO provides >95% coverage of the human vaginal microbiome, and it is applicable to populations from North America, Africa, and Asia: "we show that VIRGO provides >95% coverage of the human vaginal microbiome, and it is applicable to populations from North America, Africa, and Asia"
- [readme] VIRGO does NOT yet support paired-end mapping, please merge the paired end reads alone or together with single-end reads into one fastq file: "Please note VIRGO does NOT yet support paired-end mapping, please merge the paired end reads alone or together with single-end reads into one fastq file"
- [readme] the -l is the seed length for read mapping: "**"bowtie -l"** default value is 25 : *in step 1 scripts, the -l is the seed length for read mapping."
- [readme] the cutoff of the number of reads for calling a gene hit: "**"reads** = " default value is 2. : *in step 1 scripts, the cutoff of the number of reads for calling a gene hit."
- [discussion] The database is primarily focused on bacteria with limited inclusion of viral and fungal gene sequences: "The database is primarily focused on bacteria with limited inclusion of viral and fungal gene sequences."
