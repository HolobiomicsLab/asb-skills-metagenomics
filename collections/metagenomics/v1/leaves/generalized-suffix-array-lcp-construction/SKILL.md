---
name: generalized-suffix-array-lcp-construction
description: Use when a reference-free variant-discovery workflow (e.g. ebwt2snp-style
  positional clustering) requires the LCP array and generalized Suffix Array of a
  FASTA/FASTQ read collection, or when benchmarking disk/RAM footprint of GSA+LCP
  pipelines against BWT-only ones.
when_to_use_negative:
- The downstream tool is ebwt2InDel, which works directly on the BWT alone and computes
  the LCP on-the-fly — building the GSA and LCP array is unnecessary and costs roughly
  one order of magnitude more disk space (73 GB vs 9.5 GB on a 20x real dataset)
- Reads contain unknown characters 'N' — the input fragments must contain only 'A','C','G','T',
  so filtering or randomizing N-carrying reads must happen first
edam_operation: http://edamontology.org/operation_0227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3316
tools:
- name: egsa
  role: External-memory construction of the generalized suffix array, LCP-array, and
    (optionally) the BWT of a collection of K strings
  repo: https://github.com/felipelouza/egsa
- name: egap
  role: External-memory BWT and LCP computation for sequence collections, invoked
    with --lcp and a memory limit (e.g. -m 4096)
  repo: https://github.com/felipelouza/egap
- name: BCR_LCP_GSA
  role: Multi-string BWT computation (BCR) with related structures; used to build
    the eBWT of the read collection for ebwt2snp and to build the BWT-only input for
    ebwt2InDel
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: ebwt2snp
  role: 'Consumer of the GSA and LCP arrays: the old positional-clustering SNP caller
    requires the eBWT, LCP, and gSA arrays as input'
  repo: https://github.com/nicolaprezza/ebwt2snp
- name: ebwt2InDel
  role: 'BWT-only comparator: consumes only reads.fasta.bwt and computes LCP on-the-fly,
    used to quantify the space reduction versus the GSA/LCP pipeline'
  repo: https://github.com/nicolaprezza/ebwt2InDel
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03586-3
    title: Variable-order reference-free variant discovery with the Burrows-Wheeler
      Transform
  source_content_hash: sha256:b54b6b2c2bd65a8f576379856f6d4155b1ddfd9715926468ffbfa791cbb752de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/felipelouza/egsa
  tools:
  - BCR
  - BCR_LCP_GSA
  - ebwt2InDel
  - ebwt2snp
  - egap
  - egsa
  edam_operation: http://edamontology.org/operation_0227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# generalized-suffix-array-lcp-construction

## Summary

Build the generalized Suffix Array (GSA) and LCP array of a collection of sequencing reads using external-memory tools (egsa, egap, or BCR_LCP_GSA), as required by the old ebwt2snp variant-discovery pipeline and to quantify its space cost relative to BWT-only approaches.

## When to use

Use when a reference-free variant-discovery workflow (e.g. ebwt2snp-style positional clustering) requires the LCP array and generalized Suffix Array of a FASTA/FASTQ read collection, or when benchmarking disk/RAM footprint of GSA+LCP pipelines against BWT-only ones.

## When NOT to use

- The downstream tool is ebwt2InDel, which works directly on the BWT alone and computes the LCP on-the-fly — building the GSA and LCP array is unnecessary and costs roughly one order of magnitude more disk space (73 GB vs 9.5 GB on a 20x real dataset)
- Reads contain unknown characters 'N' — the input fragments must contain only 'A','C','G','T', so filtering or randomizing N-carrying reads must happen first

## Inputs

- Collection of K DNA reads (reads.fasta / reads.fastq) with only 'A','C','G','T' symbols, optionally including reverse-complements
- Number of strings K in the collection (or K=0 to treat the whole file as input)

## Outputs

- Generalized Suffix Array file (GSA) for the read collection
- LCP array file (e.g. reads.fasta.2.lcp from eGAP)
- BWT file (reads.fasta.bwt), produced optionally by egsa or as part of BCR_LCP_GSA / eGAP output
- memory_comparison.csv tabulating peak memory per tool (bytes and n-bit footprint) when used for benchmarking

## How to apply

Concatenate the read collection (after filtering reads containing 'N' so fragments contain only 'A','C','G','T', and adding reverse-complements to improve sensitivity) into a single FASTA file. Run egsa with the number of strings K (K=0 treats all strings as input) or eGAP with the --lcp flag and a RAM limit (e.g. -m 4096), each producing a .bwt file plus an LCP array (.lcp); BCR_LCP_GSA can alternatively build the BWT together with LCP and GSA. These arrays are mandatory inputs for the ebwt2snp pipeline, whereas the newer ebwt2InDel computes the LCP on-the-fly from the eBWT and needs only the .bwt file. When benchmarking, record peak resident memory and disk for the GSA/LCP path and compare with the BWT-only path (expected ~7n bits for n total nucleotides); the paper reports 73 GB vs 9.5 GB on the same 20x real dataset.

## Related tools

- **egsa** (External-memory construction of the generalized suffix array, LCP-array, and (optionally) the BWT of a collection of K strings) — https://github.com/felipelouza/egsa
- **egap** (External-memory BWT and LCP computation for sequence collections, invoked with --lcp and a memory limit (e.g. -m 4096)) — https://github.com/felipelouza/egap
- **BCR_LCP_GSA** (Multi-string BWT computation (BCR) with related structures; used to build the eBWT of the read collection for ebwt2snp and to build the BWT-only input for ebwt2InDel) — https://github.com/giovannarosone/BCR_LCP_GSA
- **ebwt2snp** (Consumer of the GSA and LCP arrays: the old positional-clustering SNP caller requires the eBWT, LCP, and gSA arrays as input) — https://github.com/nicolaprezza/ebwt2snp
- **ebwt2InDel** (BWT-only comparator: consumes only reads.fasta.bwt and computes LCP on-the-fly, used to quantify the space reduction versus the GSA/LCP pipeline) — https://github.com/nicolaprezza/ebwt2InDel

## Examples

```
./eGap --lcp -m 4096 dataset/reads.fastq
```

## Evaluation signals

- Both .bwt and .lcp (and/or GSA) files exist and are non-empty after egsa/eGAP completes; egsa's -c option can check SA and LCP consistency
- Disk footprint of the GSA+LCP path is roughly one order of magnitude larger than the BWT-only path (73 GB vs 9.5 GB on the paper's 20x real dataset)
- Peak resident memory of ebwt2snp on the GSA/LCP input, tabulated in memory_comparison.csv, is far larger than ebwt2InDel on the .bwt-only input
- The BWT-only working-space footprint matches the expected 7n bits, where n is the total number of nucleotides in the collection
- Downstream ebwt2snp runs to completion on the constructed arrays, confirming the GSA/LCP files are valid for the same read collection (with reverse-complements appended)

## Limitations

- The GSA and LCP arrays dominate the working space: the ebwt2snp pipeline needs 73 GB of disk versus 9.5 GB for the BWT-only ebwt2InDel pipeline on the same 20x real dataset, making this construction the wrong choice for large collections when a BWT-only alternative exists
- Input DNA fragments must contain only 'A','C','G','T'; reads containing 'N' must be filtered out or have N replaced by random nucleotides before construction
- ebwt2snp is explicitly marked as an old, unsupported version (users are pointed to ebwt2snp-v2 / ebwt2InDel)

## Evidence

- [methods] you can use https://github.com/felipelouza/egsa: "you can use https://github.com/felipelouza/egsa"
- [methods] https://github.com/giovannarosone/BCR_LCP_GSA, or https://github.com/felipelouza/egap: "https://github.com/giovannarosone/BCR_LCP_GSA, or https://github.com/felipelouza/egap"
- [readme] This is about 8 times more space-efficient (but slightly slower) than version 1.: "The new version detect also InDels and works directly on the input BWT, without building the LCP and GSA arrays. This is about 8 times more space-efficient (but slightly slower) than version 1."
- [readme] egsa outputs the GSA, LCP-array and optional BWT: "Given a collection of K strings, eGSA outputs the:

* Generalized suffix array 
* LCP-array 
* Burrows-Wheeler transform (optional)"
- [methods] the DNA fragments of the input read set must contain only symbols 'A', 'C', 'G', 'T': "the DNA fragments of the input read set must contain only symbols 'A', 'C', 'G', 'T'. It is suggested either to filter out reads containing unknown characters 'N', or to remove them, or to replace"
- [intro] we overcome the above limitation by using a recent algorithmic result [56] that shows how to extract the LCP on-the-fly from the eBWT: "we overcome the above limitation by using a recent algorithmic result [56] that shows how to extract the LCPon-the-fly from the eBWT"
