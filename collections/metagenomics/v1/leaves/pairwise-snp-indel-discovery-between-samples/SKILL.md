---
name: pairwise-snp-indel-discovery-between-samples
description: Use when two raw FASTA/FASTQ read collections from related samples must
  be compared for SNP/INDEL discovery without aligning to a reference genome, and
  sensitivity against a de Bruijn graph baseline (DISCOSNP++) must be quantified.
when_to_use_negative:
- A high-quality reference genome and mature reference-based calling are required
  or preferred — ebwt2InDel is deliberately reference-free/alignment-free
- Only a single sample (one read collection) is available and no pairwise comparison
  is needed
- 'INDEL calls from a parallelized pipeline are required: the parallel pipeline works
  for SNPs only, so sequential ebwt2InDel must be used for INDELs'
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: ebwt2InDel
  role: Core two-dataset mode variant caller that discovers SNPs/INDELs by scanning
    the BWTs of the two read collections
  repo: https://github.com/nicolaprezza/ebwt2InDel
- name: BCR (BCR_LCP_GSA)
  role: Builds the extended BWT of each input read collection, the input format ebwt2InDel
    requires
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: filter_snp
  role: Post-processing tool to discard low-covered variants, keeping only SNP/INDELs
    supported by at least m reads
  repo: https://github.com/nicolaprezza/ebwt2InDel
- name: DISCOSNP++
  role: State-of-the-art de Bruijn graph-based competitor run on the same preprocessed
    read sets for sensitivity comparison
- name: hap.py
  role: Validates the converted VCF calls of both tools against the ground truth to
    compute SNP/INDEL sensitivity
  repo: Illumina/hap.py
- name: snp2vcf (seqtk -> bwa-mem -> sam2vcf)
  role: Pipeline that converts both tools' fasta variant outputs to VCF by aligning
    variant fragments against the hg38 Human assembly
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03586-3
    title: Variable-order reference-free variant discovery with the Burrows-Wheeler
      Transform
  source_content_hash: sha256:7163b7493e2a9b38c771fdfcf0c03b9557204d013798b636ef40b9d4a7151329
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nicolaprezza/ebwt2InDel
  tools:
  - BCR
  - BCR (BCR_LCP_GSA)
  - DISCOSNP++
  - ebwt2InDel
  - filter_snp
  - hap.py
  - snp2vcf (seqtk -> bwa-mem -> sam2vcf)
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pairwise-snp-indel-discovery-between-samples

## Summary

Reference-free, alignment-free discovery of SNPs and INDELs between two read sets (e.g. mother NA12892 and son NA12878 whole-genome data at ~10x each) by running ebwt2InDel on the extended Burrows-Wheeler Transforms of the two collections, then benchmarking sensitivity against DISCOSNP++ with hap.py.

## When to use

Use when two raw FASTA/FASTQ read collections from related samples must be compared for SNP/INDEL discovery without aligning to a reference genome, and sensitivity against a de Bruijn graph baseline (DISCOSNP++) must be quantified.

## When NOT to use

- A high-quality reference genome and mature reference-based calling are required or preferred — ebwt2InDel is deliberately reference-free/alignment-free
- Only a single sample (one read collection) is available and no pairwise comparison is needed
- INDEL calls from a parallelized pipeline are required: the parallel pipeline works for SNPs only, so sequential ebwt2InDel must be used for INDELs

## Inputs

- Two raw read collections (FASTA/FASTQ) from related samples, e.g. 1000 Genomes NA12892 and NA12878 whole-genome reads at ~10x coverage each
- Per-dataset BWT files built with BCR (BCR_LCP_GSA) after N-filtering and reverse-complement augmentation
- Minimum read-support threshold for filter_snp (e.g. 5)
- Ground-truth VCF (NCBI GIAB NA12878_HG001 release or 1000 Genomes phase 3 VCF) and hg38 Human assembly for VCF conversion and validation

## Outputs

- KisSNP2-format .snp fasta file of variant contexts from ebwt2InDel, filtered to events supported by >= m reads
- VCF files for both tools produced by the snp2vcf pipeline, deduplicated across strands
- Per-tool SNP and INDEL sensitivity comparison (e.g. sensitivity_comparison_results.csv) from hap.py validation

## How to apply

Preprocess each read set by removing reads containing 'N' (inputs must contain only A/C/G/T) and append the reverse-complement of every read to improve sensitivity. Build each collection's BWT with BCR (BCR_LCP_GSA), e.g. 'BCR_LCP_GSA reads.fasta reads.fasta.bwt 1024', then run the two-dataset mode of ebwt2InDel ('ebwt2InDel -1 reads1.fasta.bwt -2 reads2.fasta.bwt -o output.snp') and keep only events supported by at least the chosen minimum read count via 'filter_snp output.snp 5 > output.5.snp'. Run DISCOSNP++ on the same preprocessed read sets as the state-of-the-art de Bruijn graph baseline, convert both tools' fasta outputs to VCF with the snp2vcf pipeline (seqtk -> bwa-mem against the hg38 Human assembly -> sam2vcf), and remove duplicate entries arising from forward/reverse strand detection. Validate both VCFs with hap.py against the ground truth (NCBI GIAB NA12878_HG001 release / 1000 Genomes phase 3 VCF) to obtain per-tool SNP and INDEL sensitivity. On the merged NA12892+NA12878 whole-genome experiment the expected outcome is that EBWT2INDEL recovers over 5 times more SNPs and over 2 times more INDELs than DISCOSNP++.

## Related tools

- **ebwt2InDel** (Core two-dataset mode variant caller that discovers SNPs/INDELs by scanning the BWTs of the two read collections) — https://github.com/nicolaprezza/ebwt2InDel
- **BCR (BCR_LCP_GSA)** (Builds the extended BWT of each input read collection, the input format ebwt2InDel requires) — https://github.com/giovannarosone/BCR_LCP_GSA
- **filter_snp** (Post-processing tool to discard low-covered variants, keeping only SNP/INDELs supported by at least m reads) — https://github.com/nicolaprezza/ebwt2InDel
- **DISCOSNP++** (State-of-the-art de Bruijn graph-based competitor run on the same preprocessed read sets for sensitivity comparison)
- **hap.py** (Validates the converted VCF calls of both tools against the ground truth to compute SNP/INDEL sensitivity) — Illumina/hap.py
- **snp2vcf (seqtk -> bwa-mem -> sam2vcf)** (Pipeline that converts both tools' fasta variant outputs to VCF by aligning variant fragments against the hg38 Human assembly)

## Examples

```
BCR_LCP_GSA reads1.fasta reads1.fasta.bwt 1024; ebwt2InDel -1 reads1.fasta.bwt -2 reads2.fasta.bwt -o output.snp; filter_snp output.snp 5 > output.5.snp
```

## Evaluation signals

- ebwt2InDel output contains only A/C/G/T-derived reads: input datasets were N-filtered and reverse-complement augmented as prescribed
- Filtered .snp file contains only events supported by at least the chosen minimum read count (e.g. 5 via filter_snp)
- VCF files for both tools are deduplicated — no repeated entries for variants found on both forward and reverse strands
- hap.py sensitivity tabulation shows EBWT2INDEL reporting >5x more SNPs and >2x more INDELs than DISCOSNP++ on the merged NA12892/NA12878 whole-genome dataset
- Both tools were run on identically preprocessed read sets so the sensitivity comparison is apples-to-apples

## Limitations

- The parallel pipeline works for SNPs only and uses an external pre-processing tool that does not account for INDELs; INDEL support is deferred to a forthcoming update
- Real-data evaluation relies on an approximated ground truth, lowering precision estimates for both tools
- No software version numbers are stated for EBWT2INDEL or the competitor, a reproducibility gap for the comparison
- Input reads must contain only A/C/G/T: reads containing 'N' must be filtered out or have N replaced/removed before BWT construction

## Evidence

- [results] Two-dataset mode converts both tools' outputs with hg38 alignment: [evidence span withheld — anchor status: not_found]
- [readme] ebwt2InDel is reference-free on BWT input: "to discover SNPs/indels inside one or between two sets of reads (fasta/fastq) *without* aligning them to  a reference genome"
- [methods] filter_snp retains variants with at least m supporting reads: "the tool **filter_snp** can be used to filter the .snp file generated by ebwt2InDel so to keep only SNP/indels covered by at least m reads"
- [results] Reverse-complement augmentation improves sensitivity: "We added the reverse-complement of the reads to each dataset in order to improve the sensitivity of our tool"
- [discussion] Parallel pipeline INDEL limitation: "As a limitation, our parallel pipeline at the moment works for SNPs only"
