---
name: exact-string-matching-sequence-collapsing
description: Use when a quality-filtered FASTQ/FNA amplicon dataset must be dereplicated
  before OTU inference and you need a deterministic exact-match collapse (or a fast
  runtime baseline to benchmark against VSEARCH-style dereplication).
when_to_use_negative:
- Input is already dereplicated or is a unique-sequence/abundance table rather than
  raw reads.
- Reads contain substantial sequencing errors and you need error-tolerant collapsing
  of near-identical reads; exact string matching will not merge single-base variants.
- The goal is similarity-threshold OTU clustering itself (e.g. de-novo greedy clustering)
  rather than the exact-identity dereplication step that precedes centroid selection.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0637
tools:
- name: MICCA-FAST
  role: Implements the exact string matching dereplication algorithm being reproduced/benchmarked.
  repo: github.com/compmetagen/micca
- name: MICCA
  role: Standard pipeline whose 'micca otu' command provides the VSEARCH-based dereplication
    used as the comparison baseline.
  repo: github.com/compmetagen/micca
- name: VSEARCH
  role: Backend wrapped by MICCA's 'otu' command that performs the standard dereplication
    step.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:e169dff6844d9c6bff34b911b68a507f8f9d716123ec6b62c02ca97140e81c25
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MICCA
  - MICCA-FAST
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# exact-string-matching-sequence-collapsing

## Summary

Collapses identical amplicon reads into unique sequences with abundance counts using an exact string-matching dereplication algorithm, as implemented by MICCA-FAST as an alternative to the VSEARCH-based dereplication inside the MICCA OTU inference pipeline.

## When to use

Use when a quality-filtered FASTQ/FNA amplicon dataset must be dereplicated before OTU inference and you need a deterministic exact-match collapse (or a fast runtime baseline to benchmark against VSEARCH-style dereplication).

## When NOT to use

- Input is already dereplicated or is a unique-sequence/abundance table rather than raw reads.
- Reads contain substantial sequencing errors and you need error-tolerant collapsing of near-identical reads; exact string matching will not merge single-base variants.
- The goal is similarity-threshold OTU clustering itself (e.g. de-novo greedy clustering) rather than the exact-identity dereplication step that precedes centroid selection.

## Inputs

- Quality-filtered amplicon reads file (FASTQ/FNA), e.g. the MICCA 16S-R synthetic dataset (500 OTUs, 100000 reads) from ftp://ftp.fmach.it/metagenomics/micca/scirep/

## Outputs

- Unique-sequence set with per-unique-sequence abundance counts (dereplicated reads)
- dereplication_benchmark.csv with per-method unique-sequence counts and wall-clock runtimes for the exact-matching routine and the standard VSEARCH-based MICCA dereplication

## How to apply

Start from preprocessed, quality-filtered amplicon reads (e.g. the MICCA 16S-R synthetic dataset of 100000 reads from ftp://ftp.fmach.it/metagenomics/micca/scirep/). Implement an exact string-matching routine that keys reads by their full sequence string, collapsing identical reads into unique sequences and summing an abundance count per unique sequence; reads differing by even one base are not collapsed. Run the standard MICCA dereplication on the same input via the 'micca otu' command, which wraps VSEARCH, and compare the two outputs on identical reads. Verify the unique-sequence sets and abundance vectors agree between methods, and record wall-clock runtime for each. Write per-method unique-sequence counts and runtimes to dereplication_benchmark.csv. The rationale, per the MICCA paper, is that OTU clustering is seeded from the most abundant reads identified after dereplication, so a correct, order-independent collapse directly determines downstream centroid selection.

## Related tools

- **MICCA-FAST** (Implements the exact string matching dereplication algorithm being reproduced/benchmarked.) — github.com/compmetagen/micca
- **MICCA** (Standard pipeline whose 'micca otu' command provides the VSEARCH-based dereplication used as the comparison baseline.) — github.com/compmetagen/micca
- **VSEARCH** (Backend wrapped by MICCA's 'otu' command that performs the standard dereplication step.)

## Examples

```
python -c "import collections,sys; c=collections.Counter(l.strip() for l in open('reads.fna') if not l.startswith('>') and l.strip()); open('derep.fna','w').write(''.join('>u%d;size=%d\n%s\n'%(i,n,s) for i,(s,n) in enumerate(c.most_common())))"
```

## Evaluation signals

- The exact-matching dereplicated output must contain the same unique sequences and same abundance vector as the standard MICCA/VSEARCH dereplicated output on identical input reads.
- Total abundance summed over unique sequences equals the number of input reads (no reads lost or double-counted).
- Unique-sequence counts for both methods on the 16S-R dataset (100000 reads) are recorded and mutually consistent.
- Wall-clock runtime is recorded for both the exact-matching routine and the VSEARCH-based approach in dereplication_benchmark.csv.
- Downstream sanity check: the most abundant unique sequences become the starting centroids, consistent with MICCA's clustering design.

## Limitations

- Exact string matching cannot collapse reads differing by even a single sequencing error, so unique-sequence counts are sensitive to residual read errors.
- The paper notes MICCA-FAST converges to a number of estimated OTUs that is on average approximately 68% of the real number, versus approximately 77% for standard MICCA, so the FAST variant trades accuracy for speed.
- The paper notes that no official, well-studied benchmark datasets exist, making accuracy and reliability of metagenomic pipelines hard to assess; validation relied on simulated data with known OTU composition (e.g. 16S-R with 500 OTUs).

## Evidence

- [results] MICCA-FAST implements an exact string matching dereplication algorithm offered as an alternative to standard dereplication.: "we show the number of OTUs estimated by MICCA and MICCA-FAST, with the latter implementing an exact string matching dereplication algorithm"
- [discussion] Starting the clustering process from the most abundant reads individuated after the dereplication step (discussion).: "starting the clustering process from the most abundant reads individuated after the dereplication step"
- [readme] VSEARCH is the backend used by MICCA's otu command, which supplies the standard dereplication.: "VSEARCH (doi: 10.7717/peerj.2584) used in ``classify``, ``filter``, ``mergepairs``, ``otu`` and ``msa`` commands"
- [results] The 16S-R synthetic dataset (500 OTUs, 100000 reads) is a suitable benchmark input.: "we have generated 16S-R, a large synthetic sequence dataset including 500 different OTUs and 100000 sequence reads"
- [readme] Preprocessed MICCA datasets are available at the stated FTP deposit for benchmarking.: "Dataset download:
ftp://ftp.fmach.it/metagenomics/micca/scirep/."
- [results] MICCA-FAST's OTU estimates converge to about 68% of the real number versus 77% for standard MICCA.: "MICCA and MICCA-FAST achieve the best performances, converging to a number of estimated OTUs that are on average approximately 77% and 68% of the real number"
