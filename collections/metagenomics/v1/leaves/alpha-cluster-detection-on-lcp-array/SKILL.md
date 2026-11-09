---
name: alpha-cluster-detection-on-lcp-array
description: 'Use when classifying metagenomic reads alignment-free with LiME: you
  have precomputed ebwt, lcp and da files (e.g. built with BCR/egsa and merged with
  eGap) for a collection S of reads plus reference genomes, and you need the α-cluster
  blocks that feed the similarity analysis.'
when_to_use_negative:
- The ebwt, lcp and da data structures for S have not yet been built or merged (use
  BCR/egsa to build and eGap with --lcp and --da to merge first).
- The goal is alignment-based read mapping against a reference database — LiME's α-cluster
  scan is an alignment-free, mapping-free strategy.
- You need the DA-only (EBWT=0) Step 2 similarity computation without re-running cluster
  detection — ClusterLCP output is shared, only ClusterBWT_DA differs.
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: LiME
  role: Provides the ClusterLCP executable that scans lcp(S) and da(S) to detect α-clusters,
    multi-threaded over independent blocks
  repo: https://github.com/veronicaguerrini/LiME
- name: BCR_LCP_GSA
  role: Builds the ebwt and lcp data structures for S that ClusterLCP consumes (preprocessing)
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: egsa
  role: Alternative external-memory builder of ebwt, lcp and da (as a single .gesa
    file converted with EGSAtoBCR)
  repo: https://github.com/felipelouza/egsa
- name: eGap
  role: Merges the ebwts of read set R and genome set G, computing lcp (--lcp or --trlcp
    k) and da (--da) for the merged collection
  repo: https://github.com/felipelouza/egap
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:4a306a5bd258e6d3f507fec8398da69eca694061b68dfd08ae93319323d1db54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BCR_LCP_GSA
  - LiME
  - eGap
  - egsa
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# alpha-cluster-detection-on-lcp-array

## Summary

Detects α-clusters — blocks of ebwt(S) whose associated suffixes share a common context of minimum length α — by sequentially scanning the lcp(S) and da(S) files of a merged read-genome collection, as the first step of LiME's lightweight metagenomic classification.

## When to use

Use when classifying metagenomic reads alignment-free with LiME: you have precomputed ebwt, lcp and da files (e.g. built with BCR/egsa and merged with eGap) for a collection S of reads plus reference genomes, and you need the α-cluster blocks that feed the similarity analysis.

## When NOT to use

- The ebwt, lcp and da data structures for S have not yet been built or merged (use BCR/egsa to build and eGap with --lcp and --da to merge first).
- The goal is alignment-based read mapping against a reference database — LiME's α-cluster scan is an alignment-free, mapping-free strategy.
- You need the DA-only (EBWT=0) Step 2 similarity computation without re-running cluster detection — ClusterLCP output is shared, only ClusterBWT_DA differs.

## Inputs

- fileFasta.lcp (longest common prefix array of collection S)
- fileFasta.da (document array of collection S)
- Merged fasta file of reads R and reference genomes G (e.g. ReadsF+Refs.fasta from setB2_1.fasta, setB2_2.fasta, their reverse complements, and Refs.fasta with 930 genomes)
- Parameters: total number of reads in S, total number of genomes, alpha (minimum cluster context length), number of threads

## Outputs

- Collection Cα of α-clusters (blocks of ebwt(S) delimited by index pairs) passed to ClusterBWT_DA
- Wall-clock runtimes per thread count if used for the thread-scaling evaluation (aggregated into runtime_scaling_table_s4.csv)

## How to apply

Run LiME's ClusterLCP executable with the name of the merged fasta file (e.g. ReadsF+Refs.fasta), the total number of reads in S, the total number of genomes, a minimum context length α (16 in the paper's thread-scaling evaluation on setB2), and a number of threads. ClusterLCP reads fileFasta.lcp and fileFasta.da to build the collection Cα of α-clusters, i.e. blocks of ebwt(S) containing symbols belonging both to reads R and genomes G whose suffixes share a context of at least length α. Because α-cluster analysis is independent across blocks, the tool is multi-threaded and each thread reads distinct parts of the input files through local buffers, so thread count can be increased (e.g. 1, 2, 4, 8) to reduce runtime. The α-clusters produced here are the input required for the subsequent Step 2 similarity analysis by ClusterBWT_DA, so this step must be run on all four merged fasta files (both strands of both paired-end read sets) before classification.

## Related tools

- **LiME** (Provides the ClusterLCP executable that scans lcp(S) and da(S) to detect α-clusters, multi-threaded over independent blocks) — https://github.com/veronicaguerrini/LiME
- **BCR_LCP_GSA** (Builds the ebwt and lcp data structures for S that ClusterLCP consumes (preprocessing)) — https://github.com/giovannarosone/BCR_LCP_GSA
- **egsa** (Alternative external-memory builder of ebwt, lcp and da (as a single .gesa file converted with EGSAtoBCR)) — https://github.com/felipelouza/egsa
- **eGap** (Merges the ebwts of read set R and genome set G, computing lcp (--lcp or --trlcp k) and da (--da) for the merged collection) — https://github.com/felipelouza/egap

## Examples

```
ClusterLCP ReadsF+Refs.fasta numReads numRefs 16 4
```

## Evaluation signals

- ClusterLCP completes on each of the four merged fasta files (forward and reverse-complement strands of both paired-end files) without errors, requiring both fileFasta.lcp and fileFasta.da to be present.
- Downstream ClusterBWT_DA runs identically across all thread-count configurations, and Classify (TaxRank=1, Reference_database.csv, 930 references) yields identical classification results.
- For the thread-scaling validation, runtime decreases monotonically as thread count increases (1, 2, 4, 8), and at every thread count the EBWT=0 (DA-only) runtime is smaller than the EBWT=1 (eBWT) runtime, consistent with Additional file 1 Table S4.
- The chosen α (e.g. 16) matches the value used in the paper's evaluation on setB2 so the α-cluster blocks are comparable.
- The classification result does not depend on the construction algorithm used for ebwt, da, and lcp — swapping BCR/egsa vs. eGap-built structures should not change output classifications.

## Limitations

- The thread-scaling evaluation results are deferred to supplementary Table S4 (Additional file 1) and are not reproduced in the main section.
- Finding the best method for building the data structures (ebwt, da, lcp) is explicitly out of the scope of the paper.
- For paired-end read collections, Step 1 (and Step 2) must be repeated for the data structures of both strands (forward and reverse complement), multiplying runs.
- eGap's --trlcp k option truncates lcp values greater than k to k, which trades precision of the lcp array for faster merging time before cluster detection.

## Evidence

- [methods] In Step 1, inspired by Remark 1, we build the collection Cα of α-clusters, which are blocks of symbols delimited by a pair of indices: [evidence span withheld — anchor status: not_found]
- [intro] multi-threading exploiting parallelization: "we implement a multi-threading version of our tool exploiting the fact that it allows a certain degree of parallelization"
- [readme] ClusterLCP invocation from README: "ClusterLCP ReadsF+Refs.fasta numReads numRefs alpha threads"
- [readme] ClusterLCP requires lcp and da files: "in order to run ClusterLCP we need to have the two data structures fileFasta.lcp and fileFasta.da computed"
- [methods] classification independence from construction algorithm: [evidence span withheld — anchor status: not_found]
- [discussion] Thread scaling deferred to supplementary: "reading it through local buffers (Additional file 1: Table S4)."
