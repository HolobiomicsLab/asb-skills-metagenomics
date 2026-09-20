---
name: on-the-fly-lcp-extraction-from-bwt
description: Use when the input is only the eBWT file of a raw-read collection (e.g.
  reads.fasta.bwt from BCR) and you need positional-clustering variant detection without
  the memory cost of explicit LCP/GSA arrays.
when_to_use_negative:
- You only have short k-mer/de Bruijn graph machinery or already possess a reference
  genome to map reads against — the paper's method is explicitly reference-free and
  alignment-free
- You need the parallel pipeline for INDEL detection — as a limitation, the parallel
  version works for SNPs only
- The input BWT was not produced by a tool compatible with ebwt2InDel's expected format
  (e.g. reads.fasta.bwt from BCR_LCP_GSA)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3316
tools:
- name: ebwt2InDel
  role: Implements the on-the-fly LCP simulation on the eBWT to detect SNP/INDEL positional
    clusters without materializing LCP/GSA arrays
  repo: github.com/nicolaprezza/ebwt2InDel
- name: BCR (BCR_LCP_GSA)
  role: Builds the eBWT of the read collection that this technique consumes as its
    sole input
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: eGSA/eGap
  role: Alternative external-memory tools for constructing the BWT (and optionally
    LCP) input
  repo: https://github.com/felipelouza/egsa
- name: ebwt2snp
  role: Old positional-clustering version that required the explicit LCP and gSA arrays;
    baseline this technique replaces
  repo: nicolaprezza/ebwt2snp
- name: filter_snp
  role: Post-processing tool that keeps only SNP/indel events from the output covered
    by at least m reads (suggested m = 5)
  repo: github.com/nicolaprezza/ebwt2InDel
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03586-3
    title: Variable-order reference-free variant discovery with the Burrows-Wheeler
      Transform
  source_content_hash: sha256:1eefb8e8f0cb8db7e82c56db1e030b7e949abadfb01c888c2946b0cb04f20391
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BCR
  - BCR (BCR_LCP_GSA)
  - eGSA/eGap
  - ebwt2InDel
  - ebwt2snp
  - filter_snp
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# on-the-fly LCP extraction from the BWT

## Summary

Simulates suffix-tree traversal on the extended Burrows-Wheeler Transform of a read collection, computing local LCP minima on-the-fly (Kmin, Kright, localMin bitvectors) without materializing the full LCP array or generalized suffix array, reducing working space by about one order of magnitude.

## When to use

Use when the input is only the eBWT file of a raw-read collection (e.g. reads.fasta.bwt from BCR) and you need positional-clustering variant detection without the memory cost of explicit LCP/GSA arrays.

## When NOT to use

- You only have short k-mer/de Bruijn graph machinery or already possess a reference genome to map reads against — the paper's method is explicitly reference-free and alignment-free
- You need the parallel pipeline for INDEL detection — as a limitation, the parallel version works for SNPs only
- The input BWT was not produced by a tool compatible with ebwt2InDel's expected format (e.g. reads.fasta.bwt from BCR_LCP_GSA)

## Inputs

- eBWT file of the input read collection (reads.fasta.bwt), previously built with BCR_LCP_GSA, eGSA, or eGap
- FASTA/FASTQ read set preprocessed to contain only A, C, G, T (reads with N filtered out or replaced)
- optional: reverse-complement of the reads added to the dataset to improve sensitivity

## Outputs

- Kmin, Kright, and localMin bitvectors computed in a single streaming pass
- output.snp file of SNP/INDEL events with surrounding read contexts (KisSNP2 fasta format), consumable by filter_snp and snp2vcf

## How to apply

Build the eBWT of the read collection beforehand with BCR (BCR_LCP_GSA) or eGSA/eGap. Load the eBWT file and, in a single streaming pass, use the recent algorithmic result showing how to extract the LCP on-the-fly from the eBWT to compute the Kmin, Kright, and localMin bitvectors, simulating suffix-tree navigation without storing the suffix array. Detect eBWT clusters as maximal intervals containing no local LCP minimum with lcp >= kmin; these mark positions where read contexts diverge (candidate SNPs/INDELs). For each candidate position, extract left and right read contexts via backward search on the eBWT and the FL mapping, assembling variant fragments written to output.snp in ebwt2InDel format. Rationale: the new ebwt2InDel version works directly on the input BWT, without building the LCP and GSA arrays, making it about 8 times more space-efficient than ebwt2snp, at the cost of being slightly slower.

## Related tools

- **ebwt2InDel** (Implements the on-the-fly LCP simulation on the eBWT to detect SNP/INDEL positional clusters without materializing LCP/GSA arrays) — github.com/nicolaprezza/ebwt2InDel
- **BCR (BCR_LCP_GSA)** (Builds the eBWT of the read collection that this technique consumes as its sole input) — https://github.com/giovannarosone/BCR_LCP_GSA
- **eGSA/eGap** (Alternative external-memory tools for constructing the BWT (and optionally LCP) input) — https://github.com/felipelouza/egsa
- **ebwt2snp** (Old positional-clustering version that required the explicit LCP and gSA arrays; baseline this technique replaces) — nicolaprezza/ebwt2snp
- **filter_snp** (Post-processing tool that keeps only SNP/indel events from the output covered by at least m reads (suggested m = 5)) — github.com/nicolaprezza/ebwt2InDel

## Examples

```
ebwt2InDel -1 reads.fasta.bwt -o output.snp; filter_snp output.snp 5 > output.5.snp
```

## Evaluation signals

- output.snp exists and parses as SNP/INDEL records in the format produced by 'ebwt2InDel -1 reads.fasta.bwt -o output.snp'
- Detected calls are consistent with events supported by at least m reads (paper threshold m = 5 for coverages >= 25x, applied via filter_snp)
- Peak memory is roughly one order of magnitude lower than ebwt2snp, which materialized the LCP and GSA arrays (README: about 8 times more space-efficient)
- On a 30x synthetic Human chromosome 1 dataset with HiSeq error profile, sensitivity should approach up to 83% of SNPs and 72% of INDELs, exceeding the de Bruijn graph tool (71% SNPs, 51% INDELs)
- Downstream VCF validation with hap.py against the NCBI GIAB NA12878_HG001 ground truth yields plausible precision/recall after hg38 alignment via the snp2vcf pipeline (seqtk -> bwa-mem -> sam2vcf)

## Limitations

- Slightly slower than ebwt2snp (the LCP/GSA-materializing version), trading speed for space
- The parallel pipeline works for SNPs only; INDEL support in the parallel path was deferred to a forthcoming update
- Real-data evaluation relies on an approximated ground truth, lowering precision estimates
- Input read fragments must contain only A, C, G, T — reads containing N must be filtered or N replaced

## Evidence

- [methods] The tool avoids materializing the full LCP array: local LCP minima are computed on-the-fly by simulating a suffix tree traversal using just the eBWT: [evidence span withheld — anchor status: not_found]
- [intro] we overcome the above limitation by using a recent algorithmic result [56] that shows how to extract the LCPon-the-fly from the eBWT: "we overcome the above limitation by using a recent algorithmic result [56] that shows how to extract the LCPon-the-fly from the eBWT"
- [readme] The new version detect also InDels and works directly on the input BWT, without building the LCP and GSA arrays.: "The new version detect also InDels and works directly on the input BWT, without building the LCP and GSA arrays."
- [results] this is done at post-processing time (using the tool filter_snp) by simply discarding low-covered variants from the output of EBWT2INDEL: "this is done at post-processing time (using the tool filter_snp) by simply discarding low-covered variants from the output of EBWT2INDEL"
- [discussion] As a limitation, our parallel pipeline at the moment works for SNPs only: "As a limitation, our parallel pipeline at the moment works for SNPs only"
