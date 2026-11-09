---
name: bwt-backward-search-context-extraction
description: Use when the input is an eBWT file (e.g. reads.fasta.bwt) of a raw-read
  collection and candidate variant intervals have been found, and read contexts flanking
  each variant must be extracted for variant-fragment assembly without aligning to
  a reference genome.
when_to_use_negative:
- The reads contain 'N' or other non-ACGT symbols and have not been filtered or replaced
  — the eBWT backward-search workflow assumes DNA fragments with only A, C, G, T
- A reference genome is available and a standard aligner-and-caller pipeline is acceptable
  — this skill is specifically for reference-free, alignment-free variant discovery
- Only the parallel pipeline is desired for INDELs — the parallelization described
  in the paper works for SNPs only
edam_operation: http://edamontology.org/operation_3227
tools:
- name: ebwt2InDel
  role: Core tool that runs backward search on the eBWT with the FL mapping to extract
    left/right read contexts and assemble variant fragments into SNP/INDEL calls;
    includes filter_snp for post-processing the .snp file
  repo: github.com/nicolaprezza/ebwt2InDel
- name: BCR
  role: Builds the multi-string eBWT of the raw-read collection that serves as the
    sole input for the backward-search context extraction
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: egsa
  role: Alternative external-memory tool that can output the BWT of the read collection
    as input to ebwt2InDel
  repo: https://github.com/felipelouza/egsa
- name: eGAP
  role: Alternative tool for computing the BWT of the read collection in external
    memory before variant discovery
  repo: https://github.com/felipelouza/egap
- name: ebwt2snp
  role: Older version that needed the eBWT plus LCP and gSA arrays; superseded by
    ebwt2InDel's backward-search approach that works directly on the BWT
  repo: https://github.com/nicolaprezza/ebwt2snp
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03586-3
    title: Variable-order reference-free variant discovery with the Burrows-Wheeler
      Transform
  source_content_hash: sha256:68ef58e267efa0a99930547321a54e9e8c01212f4c9dc2d80535b63c1be5d137
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BCR
  - eGAP
  - ebwt2InDel
  - ebwt2snp
  - egsa
  edam_operation: http://edamontology.org/operation_3227
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# bwt-backward-search-context-extraction

## Summary

Extracts left and right read contexts around candidate variant positions by performing backward search directly on the extended Burrows-Wheeler Transform (eBWT) using the FL mapping, without materializing the suffix array or LCP array. This enables reference-free assembly of variant fragments for SNP/INDEL detection with roughly one order of magnitude less working space than LCP/GSA-based approaches.

## When to use

Use when the input is an eBWT file (e.g. reads.fasta.bwt) of a raw-read collection and candidate variant intervals have been found, and read contexts flanking each variant must be extracted for variant-fragment assembly without aligning to a reference genome.

## When NOT to use

- The reads contain 'N' or other non-ACGT symbols and have not been filtered or replaced — the eBWT backward-search workflow assumes DNA fragments with only A, C, G, T
- A reference genome is available and a standard aligner-and-caller pipeline is acceptable — this skill is specifically for reference-free, alignment-free variant discovery
- Only the parallel pipeline is desired for INDELs — the parallelization described in the paper works for SNPs only

## Inputs

- eBWT file of the input read collection (reads.fasta.bwt), built with BCR_LCP_GSA or eGSA/eGap
- FASTA/FASTQ raw-read collection pre-processed to contain only symbols A, C, G, T (reads with 'N' removed or replaced)
- Minimum support threshold m for post-filtering (paper-suggested m = 5 for coverages >= 25x)

## Outputs

- Output SNP/INDEL file (output.snp) in the format produced by 'ebwt2InDel -1 reads.fasta.bwt -o output.snp'
- Fasta file in KisSNP2 format whose sequences are the contexts (left and right read fragments) surrounding the identified SNPs/indels
- Filtered variant set (output.5.snp) after 'filter_snp output.snp 5' keeps only events covered by at least m reads

## How to apply

First ensure the eBWT of the read collection was built with BCR (BCR_LCP_GSA) or an equivalent tool such as eGSA/eGap, since the input must be a multi-string BWT like reads.fasta.bwt. Detect eBWT clusters as maximal intervals containing no local LCP minimum with lcp >= kmin, computing Kmin, Kright, and localMin bitvectors in a single streaming pass via on-the-fly LCP extraction (simulated suffix-tree traversal) — never materialize the full LCP array or generalized suffix array. For each candidate variant position, use backward search on the eBWT combined with the FL mapping to pull out the left and right read contexts, then assemble the variant fragments. The fragments are emitted as a fasta file in KisSNP2 format (or .snp records via 'ebwt2InDel -1 reads.fasta.bwt -o output.snp'). Downstream, apply filter_snp to keep only events supported by at least m reads (the paper suggests m = 5 for coverages >= 25x). Correctness hinges on the read set containing only 'A','C','G','T' symbols and on adding reverse-complements of reads beforehand to improve sensitivity.

## Related tools

- **ebwt2InDel** (Core tool that runs backward search on the eBWT with the FL mapping to extract left/right read contexts and assemble variant fragments into SNP/INDEL calls; includes filter_snp for post-processing the .snp file) — github.com/nicolaprezza/ebwt2InDel
- **BCR** (Builds the multi-string eBWT of the raw-read collection that serves as the sole input for the backward-search context extraction) — https://github.com/giovannarosone/BCR_LCP_GSA
- **egsa** (Alternative external-memory tool that can output the BWT of the read collection as input to ebwt2InDel) — https://github.com/felipelouza/egsa
- **eGAP** (Alternative tool for computing the BWT of the read collection in external memory before variant discovery) — https://github.com/felipelouza/egap
- **ebwt2snp** (Older version that needed the eBWT plus LCP and gSA arrays; superseded by ebwt2InDel's backward-search approach that works directly on the BWT) — https://github.com/nicolaprezza/ebwt2snp

## Examples

```
ebwt2InDel -1 reads.fasta.bwt -o output.snp; filter_snp output.snp 5 > output.5.snp
```

## Evaluation signals

- output.snp exists and parses as SNP/INDEL records in the format emitted by 'ebwt2InDel -1 reads.fasta.bwt -o output.snp'
- Reported calls are consistent with events supportable by at least m reads (m = 5 for coverages >= 25x) after running 'filter_snp output.snp 5'
- Context sequences in the KisSNP2-format fasta flank each call with plausible left and right read fragments (no array out-of-bounds or empty contexts from FL-mapping traversal)
- Sensitivity benchmark: on a 30x synthetic Human chromosome 1 dataset the pipeline recovers up to 83% of SNPs and 72% of INDELs; substantial deviation indicates misconfigured kmin or corrupted eBWT input
- Working-space invariant: the full LCP array and generalized suffix array were never materialized (memory should be about an order of magnitude below the ebwt2snp v1 footprint, roughly 8x more space-efficient)

## Limitations

- The parallel pipeline works for SNPs only; INDEL support in the parallelized version was deferred to a future update
- Slightly slower than ebwt2snp v1 despite being ~8 times more space-efficient, since LCP values are computed on-the-fly
- Precision estimates on real data are lowered by an approximated ground truth
- Reads containing 'N' must be filtered out or replaced before running, otherwise the eBWT-based analysis is invalid

## Evidence

- [intro] on-the-fly LCP extraction from eBWT avoids materializing arrays: "we overcome the above limitation by using a recent algorithmic result [56] that shows how to extract the LCPon-the-fly from the eBWT"
- [readme] reference-free discovery by analyzing only the BWT: "by just analyzing the Burrows-Wheeler Transform of the dataset. The output is a fasta file (in KisSNP2 format) where sequences are the contexts surrounding the identified SNPs/indels."
- [methods] BWT of input must be built first: "First, the BWT of the input dataset(s) must be built."
- [methods] alphabet restriction for input reads: "the DNA fragments of the input read set must contain only symbols 'A', 'C', 'G', 'T'"
- [methods] post-filtering by minimum read support: "the tool **filter_snp** can be used to filter the .snp file generated by ebwt2InDel so to keep only SNP/indels covered by at least m reads"
- [readme] space efficiency gain of the new version: "The new version detect also InDels and works directly on the input BWT, without building the LCP and GSA arrays. This is about 8 times more space-efficient"
