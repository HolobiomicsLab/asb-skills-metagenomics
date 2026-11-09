---
name: metagenomic-read-recruitment-analysis
description: Use when evaluating or benchmarking a reference gene catalogue's sequence-capture
  efficiency on shotgun metagenomic FASTQ samples, or comparing a new catalogue (e.g.
  VMGC-90) against an incumbent (e.g. VIRGO) via per-sample mapping rates.
when_to_use_negative:
- Input is 16S/ITS amplicon data — catalogue read recruitment assumes shotgun reads
  and will not yield comparable mapping rates
- Samples are from non-vaginal body sites — the VMGC/VIRGO comparison is calibrated
  on vaginal metagenomes from 32 studies and may not transfer to gut or oral microbiome
  data
- The goal is taxonomic abundance profiling rather than capture-efficiency benchmarking
  — the VMGC project uses customized Kraken2/Bracken databases from the 786 SGBs for
  that purpose instead
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Bowtie2
  role: Aligns each sample's clean reads end-to-end (--mm --fast) against the VMGC-90
    and VIRGO gene catalogues to produce per-sample mapping rates
- name: GNU parallel 20201122
  role: Parallelizes the Bowtie2/bwa-mem2 read-recruitment jobs across the thousands
    of samples
  repo: https://github.com/RChGO/VMGC/tree/main/Pipelines
- name: bwa-mem2 2.2.1
  role: Alternative read-recruitment aligner used in the VMGC binning pipeline, with
    a perl filter retaining reads of identity>=97 before depth calculation
  repo: https://github.com/RChGO/VMGC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:af1d5b1ed462690d96e36d45e76587a2bb8593a5f381d34a225fdb68bf9a175c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bowtie2
  - GNU parallel 20201122
  - bwa-mem2 2.2.1
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# metagenomic-read-recruitment-analysis

## Summary

Recruits metagenomic reads against a reference gene/genome catalogue to quantify how much microbial sequence the catalogue captures. Used here to benchmark the VMGC-90 vaginal gene catalogue (74.5% average mapping rate) against the VIRGO catalogue (71.7%) across 4,472 vaginal metagenomes.

## When to use

Use when evaluating or benchmarking a reference gene catalogue's sequence-capture efficiency on shotgun metagenomic FASTQ samples, or comparing a new catalogue (e.g. VMGC-90) against an incumbent (e.g. VIRGO) via per-sample mapping rates.

## When NOT to use

- Input is 16S/ITS amplicon data — catalogue read recruitment assumes shotgun reads and will not yield comparable mapping rates
- Samples are from non-vaginal body sites — the VMGC/VIRGO comparison is calibrated on vaginal metagenomes from 32 studies and may not transfer to gut or oral microbiome data
- The goal is taxonomic abundance profiling rather than capture-efficiency benchmarking — the VMGC project uses customized Kraken2/Bracken databases from the 786 SGBs for that purpose instead

## Inputs

- Reference gene catalogue (VMGC-90, from Zenodo record 10457006) in FASTA
- Comparison gene catalogue (VIRGO non-redundant gene catalogue, 0.95 million genes) in FASTA
- Clean paired-end shotgun metagenomic reads (e.g. clean_reads/*.1.fq.gz, clean_reads/*.2.fq.gz) for 4,472 vaginal metagenomic samples from Supplementary Table 2

## Outputs

- Per-sample mapping rate (percentage of reads aligned) against each catalogue, extracted from the Bowtie2 alignment summary
- Average mapping rate per catalogue (VMGC-90 vs VIRGO) supporting a Fig. 6c-style comparison

## How to apply

Download the reference catalogues — VMGC-90 from Zenodo record 10457006 and the VIRGO non-redundant gene catalogue (0.95 million genes) — plus the clean reads of the 4,472 publicly available vaginal metagenomic samples from 32 studies listed in Supplementary Table 2. Build Bowtie2 indices for both catalogues, then align each sample's paired clean reads against each catalogue with Bowtie2 in end-to-end mode using --mm and --fast, parallelizing over samples with GNU parallel. Parse each Bowtie2 alignment summary to compute the per-sample mapping rate (percentage of reads aligned) and average it per catalogue; VMGC-90 should reach ~74.5% average versus ~71.7% for VIRGO (Fig. 6c). As a stronger capture check, the paper reports a median mapping rate of 91.7% when mapping the same 4,472 samples (from 14 countries) against the full VMGC genome collection, and the VMGC pipeline itself uses bwa-mem2 with an identity≥97 read filter for depth-sensitive recruitment.

## Related tools

- **Bowtie2** (Aligns each sample's clean reads end-to-end (--mm --fast) against the VMGC-90 and VIRGO gene catalogues to produce per-sample mapping rates)
- **GNU parallel 20201122** (Parallelizes the Bowtie2/bwa-mem2 read-recruitment jobs across the thousands of samples) — https://github.com/RChGO/VMGC/tree/main/Pipelines
- **bwa-mem2 2.2.1** (Alternative read-recruitment aligner used in the VMGC binning pipeline, with a perl filter retaining reads of identity>=97 before depth calculation) — https://github.com/RChGO/VMGC

## Examples

```
find clean_reads/*.1.fq.gz | sed 's/.1.fq.gz//' | parallel -j 10 "bowtie2 --mm --fast -x VMGC-90 -1 clean_reads/{}.1.fq.gz -2 clean_reads/{}.2.fq.gz > maps/{}.sam 2> maps/{}.bowtie2.log"
```

## Evaluation signals

- Per-sample mapping rates can be re-derived from the Bowtie2 stderr alignment summaries for both catalogues and averaged
- Average mapping rate to VMGC-90 should be ~74.5%, exceeding the ~71.7% observed for VIRGO (Fig. 6c benchmark)
- Across all 4,472 samples, per-catalogue mapping-rate distributions should show VMGC-90 ≥ VIRGO for the vast majority of vaginal samples
- As a genome-level cross-check, mapping the same samples against the full VMGC reference (786 SGBs + 11 fungi + 4,263 vOTUs) yields a median mapping rate of 91.7%
- Recruited-read counts should be symmetric across mates and consistent between --fast runs and a re-run with default sensitivity on a small sample subset

## Limitations

- Clean reads quality and host-read removal in the 32 public studies affect mapping rates, and the article notes potential contamination from experimental operations or kits in the public vaginal datasets used to build VMGC
- The VMGC's microbial sequence capture is incomplete, especially for vaginal samples from non-western populations, so low mapping rates for such samples may reflect catalogue gaps rather than sequencing issues
- Using Bowtie2 --fast trades alignment sensitivity for speed, which can slightly depress mapping rates relative to default or --very-sensitive modes

## Evidence

- [abstract] We collected a total of 4,472 publicly available vaginal metagenomic samples sourced from the human vagina, spanning 32 studies: "We collected a total of 4,472 publicly available vaginal metagenomic samples sourced from the human vagina, spanning 32 studies"
- [discussion] surpassing the VIRGO database, VMGC offers an extensive expansion of microbial gene sequences: "surpassing the VIRGO database10, VMGC offers an extensive expansion of microbial gene sequences"
- [readme] README claims median 91.7% mapping rate: "with a median mapping rate of 91.7% across 4,472 vaginal metagenomic samples obtained from 14 countries"
- [readme] README read-recruitment identity filter: "$identity=$match_counts/$cov_length*100; print "$_\n" if $identity>=97"
