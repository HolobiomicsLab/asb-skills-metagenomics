---
name: peak-memory-profiling-of-variant-callers
description: Use when benchmarking the disk/RAM footprint of reference-free variant
  discovery pipelines, e.g. comparing a BWT-only tool (ebwt2InDel, 7n bits expected
  working space) against GSA/LCP-based predecessors (ebwt2snp) on the same FASTQ/FASTA
  reads.
when_to_use_negative:
- Input reads still contain the unknown symbol 'N' and have not been filtered or randomized
  — the tools require fragments with only 'A','C','G','T'
- The parallel pipeline variant is intended for INDELs — it works for SNPs only, so
  INDEL-related memory claims should not be made from the parallel path
- Comparing variant-call quality rather than working-space cost — use hap.py validation
  against ground truth instead
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: ebwt2InDel
  role: BWT-only variant caller whose peak resident memory is measured; expected working
    space is 7n bits
  repo: https://github.com/nicolaprezza/ebwt2InDel
- name: ebwt2snp
  role: Baseline (version 1) pipeline requiring the eBWT, LCP and gSA arrays; its
    memory/disk use is the comparison reference
  repo: https://github.com/nicolaprezza/ebwt2snp
- name: BCR_LCP_GSA
  role: Builds the BWT (with optional RAM setting, e.g. 1024 MB) supplying reads.fasta.bwt
    to the BWT-only path
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: egsa
  role: External-memory builder of the generalized suffix array, LCP-array and (optional)
    BWT for the ebwt2snp baseline
  repo: https://github.com/felipelouza/egsa
- name: egap
  role: Alternative external-memory BWT and LCP computation for the GSA/LCP baseline
    path
  repo: https://github.com/felipelouza/egap
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03586-3
    title: Variable-order reference-free variant discovery with the Burrows-Wheeler
      Transform
  source_content_hash: sha256:d94c32ad87c80cd60f9ca29c73c809d2830212bb13ced5fcc380d5eed00ae928
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nicolaprezza/ebwt2InDel
  tools:
  - BCR
  - BCR_LCP_GSA
  - ebwt2InDel
  - ebwt2snp
  - egap
  - egsa
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# peak-memory-profiling-of-variant-callers

## Summary

Benchmarks the working-space (disk and peak resident memory) footprint of reference-free variant callers, specifically comparing the BWT-only ebwt2InDel pipeline against the older ebwt2snp pipeline that requires the generalized Suffix Array (GSA) and LCP array, on the same read dataset.

## When to use

Use when benchmarking the disk/RAM footprint of reference-free variant discovery pipelines, e.g. comparing a BWT-only tool (ebwt2InDel, 7n bits expected working space) against GSA/LCP-based predecessors (ebwt2snp) on the same FASTQ/FASTA reads.

## When NOT to use

- Input reads still contain the unknown symbol 'N' and have not been filtered or randomized — the tools require fragments with only 'A','C','G','T'
- The parallel pipeline variant is intended for INDELs — it works for SNPs only, so INDEL-related memory claims should not be made from the parallel path
- Comparing variant-call quality rather than working-space cost — use hap.py validation against ground truth instead

## Inputs

- FASTA/FASTQ read collection (raw WGS reads, e.g. 20x or 30x coverage, chromosome 1 or whole-genome) with reads containing 'N' removed or randomized
- Dataset plus its reverse-complement reads
- reads.fasta.bwt produced by BCR_LCP_GSA (BWT-only path)
- BWT + generalized Suffix Array + LCP array produced by egsa or egap (ebwt2snp path)

## Outputs

- memory_comparison.csv tabulating peak memory (bytes and n-bit footprint) per tool
- Disk-footprint figures for .bwt files and GSA/LCP structures per pipeline
- Variant output files (output.snp for ebwt2InDel; ebwt2snp equivalent) confirming both tools ran on the identical read set

## How to apply

Preprocess the read collection so fragments contain only 'A','C','G','T' (remove or randomize reads carrying 'N') and append the reverse-complement of each read, since both pipelines analyze the dataset and its reverse-complement. Build only the BWT with BCR_LCP_GSA (e.g. with a 1024 MB RAM setting) producing reads.fasta.bwt for the ebwt2InDel path; separately build BWT + generalized Suffix Array + LCP array with egsa or egap for the ebwt2snp path. Run ebwt2InDel in single-dataset mode (ebwt2InDel -1 reads.fasta.bwt -o output.snp) and ebwt2snp on its GSA/LCP input, recording each process's peak resident memory. Verify the expected 7n-bit footprint of the BWT-only algorithm (n = total number of nucleotides) and the roughly one-order-of-magnitude reduction (9.5 GB vs 73 GB on a 20x real dataset). Keep BWT construction time/memory separate from the variant-detection cost, as the paper does when separating BCR eBWT build time from EBWT2INDEL runtime.

## Related tools

- **ebwt2InDel** (BWT-only variant caller whose peak resident memory is measured; expected working space is 7n bits) — https://github.com/nicolaprezza/ebwt2InDel
- **ebwt2snp** (Baseline (version 1) pipeline requiring the eBWT, LCP and gSA arrays; its memory/disk use is the comparison reference) — https://github.com/nicolaprezza/ebwt2snp
- **BCR_LCP_GSA** (Builds the BWT (with optional RAM setting, e.g. 1024 MB) supplying reads.fasta.bwt to the BWT-only path) — https://github.com/giovannarosone/BCR_LCP_GSA
- **egsa** (External-memory builder of the generalized suffix array, LCP-array and (optional) BWT for the ebwt2snp baseline) — https://github.com/felipelouza/egsa
- **egap** (Alternative external-memory BWT and LCP computation for the GSA/LCP baseline path) — https://github.com/felipelouza/egap

## Examples

```
ebwt2InDel -1 reads.fasta.bwt -o output.snp
```

## Evaluation signals

- Peak resident memory of the ebwt2InDel process matches the expected 7n-bit working-space bound, where n is the total number of nucleotides in the read collection
- The memory ratio ebwt2snp/ebwt2InDel is roughly one order of magnitude (reported 73 GB vs 9.5 GB on the 20x real dataset; README states 'about 8 times more space-efficient')
- memory_comparison.csv contains one entry per tool with peak memory in bytes and as an n-bit footprint, and the ebwt2InDel entry is markedly smaller
- Both callers ran on the identical preprocessed read set (reads plus reverse-complements, no 'N' symbols), so the footprint comparison is apples-to-apples
- BWT construction (e.g. BCR, which required 23 h 19 min on the 320M-read dataset) is reported separately from the variant-detection step's cost

## Limitations

- The BWT-only version is slightly slower than ebwt2snp despite being ~8x more space-efficient
- eBWT construction cost (e.g. 23 hours 19 minutes by BCR on the real dataset) is excluded from the reported computational cost, so total wall-clock/memory budgets will be larger
- The paper reports both 'about 8 times' (README) and 'one order of magnitude' (abstract) space reduction — treat the ratio as approximate and re-measure on your own data
- No software version numbers are stated for EBWT2INDEL or the predecessor, complicating exact replication of the benchmark

## Evidence

- [abstract] Working space reduced by one order of magnitude using just the BWT: "perform the whole analysis using just the BWT, thus reducing the working space by one order of magnitude"
- [readme] README space-efficiency claim for BWT-only version: "The new version detect also InDels and works directly on the input BWT, without building the LCP and GSA arrays. This is about 8 times more space-efficient"
- [results] Separating BWT build cost from caller cost: "We separate the running time required to build the BWT (by using BCR [42, 43]) from the running time of EBWT2INDEL"
- [methods] Input alphabet preprocessing requirement: "the DNA fragments of the input read set must contain only symbols 'A', 'C', 'G', 'T'"
- [intro] Rationale for replacing GSA/LCP with eBWT-derived LCP: "we overcome the above limitation by using a recent algorithmic result [56] that shows how to extract the LCPon-the-fly from the eBWT"
