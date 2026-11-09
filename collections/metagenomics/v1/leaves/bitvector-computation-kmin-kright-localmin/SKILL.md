---
name: bitvector-computation-kmin-kright-localmin
description: Use when the input is an eBWT file (e.g. reads.fasta.bwt built with BCR_LCP_GSA)
  of raw sequencing reads and reference-free SNP/INDEL detection requires positional
  clustering without storing LCP or gSA arrays.
when_to_use_negative:
- The variant caller is a mapping-based pipeline expecting a reference genome and
  VCF input rather than an eBWT of raw reads
- The dataset has not been preprocessed (reads still contain N characters or the reverse-complement
  was not added, hurting sensitivity)
- Only the old ebwt2snp workflow is acceptable and the LCP/gSA arrays are available,
  in which case the LCP-materializing version is applicable directly
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3168
tools:
- name: ebwt2InDel
  role: Core tool implementing the on-the-fly LCP extraction and bitvector computation
    on the input BWT to detect SNPs and INDELs
  repo: github.com/nicolaprezza/ebwt2InDel
- name: BCR
  role: Builds the eBWT (reads.fasta.bwt) of the input read collection that the bitvector
    computation consumes
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: eGSA
  role: Alternative external-memory tool to construct the BWT/LCP inputs of a string
    collection
  repo: https://github.com/felipelouza/egsa
- name: eGap
  role: Alternative external-memory BWT (and optional LCP) computation for sequence
    collections
  repo: https://github.com/felipelouza/egap
- name: filter_snp
  role: Post-processing tool to keep only SNP/INDEL events in output.snp covered by
    at least m reads (paper suggests m = 5 for coverages >= 25x)
  repo: github.com/nicolaprezza/ebwt2InDel
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03586-3
    title: Variable-order reference-free variant discovery with the Burrows-Wheeler
      Transform
  source_content_hash: sha256:ca3a0262a88a7737861a31393fbe6a5ac631b2f04ec6ec2504f8d13b6c42c31f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BCR
  - eGSA
  - eGap
  - ebwt2InDel
  - filter_snp
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Bitvector computation (Kmin, Kright, localMin) on the eBWT

## Summary

Computes the Kmin, Kright, and localMin bitvectors in a single streaming pass over the extended Burrows-Wheeler Transform (eBWT) of a read collection, by simulating suffix-tree navigation and extracting local LCP minima on-the-fly. This avoids materializing the full LCP array and generalized suffix array, reducing working space by an order of magnitude.

## When to use

Use when the input is an eBWT file (e.g. reads.fasta.bwt built with BCR_LCP_GSA) of raw sequencing reads and reference-free SNP/INDEL detection requires positional clustering without storing LCP or gSA arrays.

## When NOT to use

- The variant caller is a mapping-based pipeline expecting a reference genome and VCF input rather than an eBWT of raw reads
- The dataset has not been preprocessed (reads still contain N characters or the reverse-complement was not added, hurting sensitivity)
- Only the old ebwt2snp workflow is acceptable and the LCP/gSA arrays are available, in which case the LCP-materializing version is applicable directly

## Inputs

- eBWT file of the input read collection (reads.fasta.bwt), built with BCR_LCP_GSA, eGSA, or eGap
- Read set in FASTA/FASTQ format preprocessed to contain only A, C, G, T (reads containing N filtered out or replaced)

## Outputs

- Kmin, Kright, and localMin bitvectors defining eBWT cluster boundaries
- output.snp: SNP/INDEL events in KisSNP2-compatible fasta/.snp format with contexts surrounding the identified variants

## How to apply

Load the eBWT file of the input read collection, previously built with BCR (or eGSA/eGap). Simulate suffix-tree navigation on the eBWT using the on-the-fly LCP extraction technique, computing the Kmin, Kright, and localMin bitvectors in a single streaming pass without materializing the suffix array. Detect eBWT clusters as maximal intervals of the eBWT containing no local LCP minimum with lcp >= kmin, marking positions where read contexts diverge. For each candidate variant position, extract left and right read contexts using backward search on the eBWT and the FL mapping, assembling variant fragments, and write the detected SNP/INDEL events to output.snp. The rationale is that a recent algorithmic result shows how to extract the LCP on-the-fly from the eBWT, so local LCP minima are computed on-the-fly instead of storing the full LCP array, replacing the generalized suffix array with combinatorial properties of the LF mapping.

## Related tools

- **ebwt2InDel** (Core tool implementing the on-the-fly LCP extraction and bitvector computation on the input BWT to detect SNPs and INDELs) — github.com/nicolaprezza/ebwt2InDel
- **BCR** (Builds the eBWT (reads.fasta.bwt) of the input read collection that the bitvector computation consumes) — https://github.com/giovannarosone/BCR_LCP_GSA
- **eGSA** (Alternative external-memory tool to construct the BWT/LCP inputs of a string collection) — https://github.com/felipelouza/egsa
- **eGap** (Alternative external-memory BWT (and optional LCP) computation for sequence collections) — https://github.com/felipelouza/egap
- **filter_snp** (Post-processing tool to keep only SNP/INDEL events in output.snp covered by at least m reads (paper suggests m = 5 for coverages >= 25x)) — github.com/nicolaprezza/ebwt2InDel

## Examples

```
ebwt2InDel -1 reads.fasta.bwt -o output.snp; filter_snp output.snp 5 > output.5.snp
```

## Evaluation signals

- output.snp exists and parses as SNP/INDEL records with left/right read contexts surrounding each event
- Variant calls are consistent with events supportable by at least m reads; applying 'filter_snp output.snp 5 > output.5.snp' yields a non-empty filtered set at expected coverage (>= 25x)
- Memory usage is consistent with working directly on the BWT (about 8 times more space-efficient than the ebwt2snp version that builds LCP and GSA arrays)
- No suffix array or full LCP array is materialized during the run; bitvectors (Kmin, Kright, localMin) are computed in a single streaming pass
- Sensitivity on a 30x synthetic Human chromosome 1 dataset should be in the reported range (up to 83% SNPs, 72% INDELs) when validated with hap.py against ground truth

## Limitations

- The parallel pipeline works for SNPs only; INDEL support in the parallelized version is deferred to a forthcoming update
- An external pre-processing tool does not take into account the presence of INDELs in the read set
- The bitvector-based version is about 8 times more space-efficient but slightly slower than the LCP/gSA-based version 1 (ebwt2snp)
- Real-data evaluation relies on an approximated ground truth, lowering precision estimates

## Evidence

- [intro] we overcome the above limitation by using a recent algorithmic result [56] that shows how to extract the LCPon-the-fly from the eBWT: "we overcome the above limitation by using a recent algorithmic result [56] that shows how to extract the LCPon-the-fly from the eBWT"
- [readme] The new version works directly on the input BWT, without building the LCP and GSA arrays, about 8 times more space-efficient.: "The new version detect also InDels and works directly on the input BWT, without building the LCP and GSA arrays. This is about 8 times more space-efficient"
- [methods] the tool filter_snp can be used to filter the .snp file generated by ebwt2InDel so to keep only SNP/indels covered by at least m reads: "the tool **filter_snp** can be used to filter the .snp file generated by ebwt2InDel so to keep only SNP/indels covered by at least m reads"
- [abstract] the tool ebwt2InDel is available at github.com/nicolaprezza/ebwt2InDel: "The tool ebwt2InDel is available at github.com/nicolaprezza/ebwt2InDel"
