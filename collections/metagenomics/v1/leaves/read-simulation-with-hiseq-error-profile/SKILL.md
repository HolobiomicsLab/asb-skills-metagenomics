---
name: read-simulation-with-hiseq-error-profile
description: 'Use when benchmarking a SNP/INDEL caller on synthetic data with a known
  ground truth: you need reads simulated from a bi-allelic genome (e.g. Human chromosome
  1 at 30x, HiSeq error profile) instead of real reads with an approximated truth
  set.'
when_to_use_negative:
- The goal is to evaluate a caller on real sequencing data where an established gold
  standard exists (e.g. NCBI GIAB NA12878_HG001); simulation is the paper's synthetic-data
  route only
- The target workflow needs reads carrying INDELs whose positions the downstream parallel
  pipeline cannot handle (the paper notes the parallel path's external pre-processing
  does not account for INDELs)
- A single-variant (non-bi-allelic) sample is required; the coverage split uniformly
  between two chromosome variants assumes a bi-allelic setup
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: SimSeq
  role: Simulates the synthetic 100 bp reads from the bi-allelic chromosome at the
    chosen coverage using the HiSeq error profile
- name: BCR (BCR_LCP_GSA)
  role: Builds the eBWT of the simulated read collection that ebwt2InDel consumes
    downstream
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: hap.py
  role: Validates the variant calls made on the simulated dataset against the ground-truth
    variant VCF
  repo: Illumina/hap.py
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03586-3
    title: Variable-order reference-free variant discovery with the Burrows-Wheeler
      Transform
  source_content_hash: sha256:c1dd4ae1654a3d87f4ff21241fc55ffc3799f985ad7ef812d8bfbb0b9c51335a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/giovannarosone/BCR_LCP_GSA
  tools:
  - BCR
  - BCR (BCR_LCP_GSA)
  - BCR_LCP_GSA
  - DISCOSNP++
  - HARC
  - SimSeq
  - bwa mem
  - ebwt2InDel
  - filter_snp
  - hap.py
  - pebwt2InDel
  - sam2vcf
  - seqtk
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# read-simulation-with-hiseq-error-profile

## Summary

Generate synthetic short-read datasets with SimSeq from a bi-allelic reference (e.g. Human chromosome 1 carrying 1000 Genomes heterozygous variations), using the HiSeq error profile and coverage distributed uniformly between the two chromosome variants, to produce benchmark data with a known ground-truth variant VCF for evaluating reference-free variant callers such as ebwt2InDel versus DISCOSNP++.

## When to use

Use when benchmarking a SNP/INDEL caller on synthetic data with a known ground truth: you need reads simulated from a bi-allelic genome (e.g. Human chromosome 1 at 30x, HiSeq error profile) instead of real reads with an approximated truth set.

## When NOT to use

- The goal is to evaluate a caller on real sequencing data where an established gold standard exists (e.g. NCBI GIAB NA12878_HG001); simulation is the paper's synthetic-data route only
- The target workflow needs reads carrying INDELs whose positions the downstream parallel pipeline cannot handle (the paper notes the parallel path's external pre-processing does not account for INDELs)
- A single-variant (non-bi-allelic) sample is required; the coverage split uniformly between two chromosome variants assumes a bi-allelic setup

## Inputs

- Reference chromosome sequence (e.g. Human chromosome 1, hg build used with hg38 for VCF conversion)
- Ground-truth heterozygous variant VCF from the 1000 Genomes Project phase 3 release 20130502
- SimSeq simulator with the HiSeq error profile
- Target coverage level (focal 30x; paper sweeps 10x-50x), 100 bp read length

## Outputs

- Synthetic read dataset (FASTA/FASTQ) simulated from the bi-allelic chromosome with coverage split uniformly between the two variants
- Ground-truth variant VCF used later for hap.py validation
- Preprocessed read set (N-containing reads removed, reverse-complements appended) ready for eBWT construction with BCR_LCP_GSA

## How to apply

First build the bi-allelic target sequence: introduce heterozygous variations taken from the 1000 Genomes phase 3 VCF release (ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/release/20130502/) into a reference chromosome (e.g. Human chromosome 1). Then simulate 100 bp paired reads with SimSeq, uniformly distributing the coverage between the two chromosome variants and using the HiSeq error profile; in the paper's benchmark the focal coverage was 30x on Human chromosome 1, with sweeps across 10x-50x also described. Save the simulated FASTA/FASTQ plus the ground-truth variant VCF. Before downstream BWT-based analysis, filter out reads containing the symbol 'N' (the input fragments must contain only 'A','C','G','T') and append the reverse-complement of every read to improve sensitivity. Use coverage sweeps (10x, 20x, 30x, 40x, 50x) to probe how sensitivity changes with depth while keeping the error profile fixed to HiSeq.

## Related tools

- **SimSeq** (Simulates the synthetic 100 bp reads from the bi-allelic chromosome at the chosen coverage using the HiSeq error profile)
- **BCR (BCR_LCP_GSA)** (Builds the eBWT of the simulated read collection that ebwt2InDel consumes downstream) — https://github.com/giovannarosone/BCR_LCP_GSA
- **hap.py** (Validates the variant calls made on the simulated dataset against the ground-truth variant VCF) — Illumina/hap.py

## Evaluation signals

- Coverage must be distributed uniformly between the two chromosome variants; a skew indicates a simulation configuration error
- Read length should be 100 bp matching the HiSeq profile used in the paper's benchmark
- Downstream preprocessing check: no read in the final dataset contains 'N' (only 'A','C','G','T' allowed)
- Sensitivity computed via hap.py on the simulated set should be in the paper's range (~83% SNPs, ~72% INDELs for ebwt2InDel vs ~71%/~51% for DISCOSNP++)
- Coverage sweeps from 10x to 50x should show sensitivity varying with depth while the error profile stays fixed

## Limitations

- Simulated reads must contain only 'A','C','G','T'; reads with 'N' must be filtered out or the characters replaced/removed before BWT construction
- Real-data comparisons use an approximated ground truth with lower precision; the synthetic simulation route avoids this but tests only variants present in the 1000 Genomes phase 3 heterozygous set
- Results depend on the specific error profile chosen (HiSeq); conclusions may not transfer to other sequencing platforms

## Evidence

- [results] The modified bi-allelic chromosome was used to simulate synthetic reads with the tool SimSeq [63]: "The modified bi-allelic chromosome was used to simulate synthetic reads with the tool SimSeq [63]"
- [results] simulate synthetic reads with the tool SimSeq [63], uniformly distributing the coverage among the two chromosome's variants and using the HiSeq error profile: [evidence span withheld — anchor status: not_found]
- [methods] Only ACGT reads allowed downstream: "the DNA fragments of the input read set must contain only symbols 'A', 'C', 'G', 'T'. It is suggested either to filter out reads containing unknown characters 'N'"
- [results] Ground truth variant source: "each heterozygous variation contained in the .vcf file downloaded from ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/release/20130502/"
