---
name: bwt-construction-of-read-collections
description: Use when raw NGS reads (FASTA/FASTQ) need to be indexed for reference-free,
  alignment-free variant discovery, and the downstream tool (ebwt2InDel) consumes
  a .bwt file as input.
when_to_use_negative:
- The downstream analysis needs the full generalized suffix array/LCP anyway (e.g.
  running the legacy ebwt2snp pipeline) — use egsa or eGap --lcp instead, but expect
  ~10x more disk (73 GB vs 9.5 GB on a 20x real dataset)
- Reads still contain 'N' symbols and have not been filtered or randomized — the BWT-based
  variant pipeline requires fragments containing only 'A', 'C', 'G', 'T'
- The goal is reference-based variant calling on aligned reads (BAM) — this skill
  indexes raw reads only, not reference-aligned data
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: BCR (BCR_LCP_GSA)
  role: Primary tool used to compute the multi-string eBWT of the read collection,
    producing the .bwt file consumed by ebwt2InDel
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: egsa
  role: External-memory alternative that constructs the generalized suffix array,
    LCP array and (optionally, with the -b flag) the BWT of the read collection
  repo: https://github.com/felipelouza/egsa
- name: egap
  role: External-memory alternative for computing the BWT (and LCP with --lcp) of
    a sequence collection when eGSA-style output is required
  repo: https://github.com/felipelouza/egap
- name: ebwt2InDel
  role: Downstream consumer of the .bwt file; runs in single- or two-dataset mode
    to call SNPs/INDELs directly on the BWT without building LCP/GSA
  repo: https://github.com/nicolaprezza/ebwt2InDel
- name: ebwt2snp
  role: Legacy comparator pipeline that requires the BWT plus GSA and LCP arrays,
    used to quantify the space savings of the BWT-only approach
  repo: https://github.com/nicolaprezza/ebwt2snp
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03586-3
    title: Variable-order reference-free variant discovery with the Burrows-Wheeler
      Transform
  source_content_hash: sha256:242bd483c657378095757b0e55cb88a7b601804723a1ec8443942e7d779faa58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/giovannarosone/BCR_LCP_GSA
  tools:
  - BCR
  - BCR (BCR_LCP_GSA)
  - DISCOSNP++
  - ebwt2InDel
  - ebwt2snp
  - egap
  - egsa
  - filter_snp
  - hap.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# bwt-construction-of-read-collections

## Summary

Build the (extended) Burrows-Wheeler Transform of one or two raw read collections (FASTA/FASTQ) as the sole input structure for reference-free SNP/INDEL discovery with ebwt2InDel, using a multi-string BWT builder such as BCR (BCR_LCP_GSA), eGSA, or eGap instead of constructing the heavier GSA/LCP arrays.

## When to use

Use when raw NGS reads (FASTA/FASTQ) need to be indexed for reference-free, alignment-free variant discovery, and the downstream tool (ebwt2InDel) consumes a .bwt file as input.

## When NOT to use

- The downstream analysis needs the full generalized suffix array/LCP anyway (e.g. running the legacy ebwt2snp pipeline) — use egsa or eGap --lcp instead, but expect ~10x more disk (73 GB vs 9.5 GB on a 20x real dataset)
- Reads still contain 'N' symbols and have not been filtered or randomized — the BWT-based variant pipeline requires fragments containing only 'A', 'C', 'G', 'T'
- The goal is reference-based variant calling on aligned reads (BAM) — this skill indexes raw reads only, not reference-aligned data

## Inputs

- FASTA or FASTQ file of raw sequencing reads (e.g. Human chromosome 1 synthetic reads at 30x, 1000 Genomes HG00096 chr1 reads, or NA12892/NA12878 whole-genome read sets)
- Preprocessed read set containing only A/C/G/T symbols with reverse-complements appended
- Memory bound for the BWT builder (e.g. 1024 MB for BCR_LCP_GSA)

## Outputs

- reads.fasta.bwt — the BWT of the read collection, in ebwt2InDel's expected input format
- Optionally, .lcp and generalized suffix array files if the old ebwt2snp-style pipeline is also being run (via egsa or eGap --lcp)

## How to apply

Preprocess the read set so every fragment contains only the symbols 'A', 'C', 'G', 'T' — filter out reads carrying 'N' or replace 'N' with random nucleotides — and append the reverse-complement of each read to improve downstream sensitivity. Then build the multi-string BWT with BCR from BCR_LCP_GSA, supplying a memory bound in MB (e.g. 'BCR_LCP_GSA reads.fasta reads.fasta.bwt 1024' for a 1024 MB setting); alternatively use egsa with the BWT-output option or eGap. For ebwt2InDel's new BWT-only pipeline, do NOT build the generalized suffix array or LCP array — the algorithm extracts the LCP on-the-fly from the eBWT and uses LF-mapping properties in place of the GSA, which reduces working space to about 7n bits (one order of magnitude less disk than the old ebwt2snp pipeline: 9.5 GB vs 73 GB on a 20x real dataset). The resulting reads.fasta.bwt is passed to ebwt2InDel (single-dataset mode: '-1 reads.fasta.bwt'; two-dataset mode: '-1 reads1.fasta.bwt -2 reads2.fasta.bwt').

## Related tools

- **BCR (BCR_LCP_GSA)** (Primary tool used to compute the multi-string eBWT of the read collection, producing the .bwt file consumed by ebwt2InDel) — https://github.com/giovannarosone/BCR_LCP_GSA
- **egsa** (External-memory alternative that constructs the generalized suffix array, LCP array and (optionally, with the -b flag) the BWT of the read collection) — https://github.com/felipelouza/egsa
- **egap** (External-memory alternative for computing the BWT (and LCP with --lcp) of a sequence collection when eGSA-style output is required) — https://github.com/felipelouza/egap
- **ebwt2InDel** (Downstream consumer of the .bwt file; runs in single- or two-dataset mode to call SNPs/INDELs directly on the BWT without building LCP/GSA) — https://github.com/nicolaprezza/ebwt2InDel
- **ebwt2snp** (Legacy comparator pipeline that requires the BWT plus GSA and LCP arrays, used to quantify the space savings of the BWT-only approach) — https://github.com/nicolaprezza/ebwt2snp

## Examples

```
BCR_LCP_GSA reads.fasta reads.fasta.bwt 1024
```

## Evaluation signals

- The .bwt file is produced and its size equals the total number of input nucleotides plus one end-marker per read, over an alphabet restricted to A/C/G/T plus the implicit distinct per-string sentinel
- ebwt2InDel accepts the .bwt without error in single- or two-dataset mode ('-1 reads1.fasta.bwt -2 reads2.fasta.bwt -o output.snp')
- Peak memory of the BWT-only path matches the expected 7n-bit footprint (n = total nucleotides), roughly one order of magnitude below the ebwt2snp GSA/LCP path on the same reads
- Reverse-complemented, N-filtered input yields higher variant sensitivity in the downstream ebwt2InDel run than the un-preprocessed collection

## Limitations

- Reads containing the symbol 'N' must be removed or randomized before BWT construction — input fragments must contain only 'A', 'C', 'G', 'T'
- The new BWT-only version is about 8 times more space-efficient than ebwt2snp version 1 but slightly slower in CPU time; BWT construction itself can be costly (BCR required 23 hours and 19 minutes on the whole-genome dataset) and is typically excluded from reported tool runtime
- eGap with the LCP option produces extra .lcp output required only by the legacy pipeline, not by the BWT-only ebwt2InDel path

## Evidence

- [readme] README truncation of ebwt2InDel BWT-only design: "The new version detect also InDels and works directly on the input BWT, without building the LCP and GSA arrays."
- [methods] Alphabet restriction before BWT construction: "the DNA fragments of the input read set must contain only symbols 'A', 'C', 'G', 'T'. It is suggested either to filter out reads containing unknown characters 'N'"
- [results] Reverse-complement augmentation rationale: "We added the reverse-complement of the reads to each dataset in order to improve the sensitivity of our tool"
- [readme] BCR multi-string eBWT construction mechanism: [evidence span withheld — anchor status: external_doc]
- [results] BCR as the BWT builder separated from tool runtime: "We separate the running time required to build the BWT (by using BCR [42, 43]) from the running time of EBWT2INDEL"
