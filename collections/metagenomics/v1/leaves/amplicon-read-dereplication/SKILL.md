---
name: amplicon-read-dereplication
description: Use when quality-filtered FASTQ/FASTA amplicon reads must be collapsed
  to unique sequences with abundance vectors before de novo OTU inference, chimera
  filtering, or benchmarking dereplication speed.
when_to_use_negative:
- Input is already a dereplicated unique-sequence set with abundance counts
- The goal is reference-based OTU picking against a curated database rather than de
  novo inference
- Reads have not yet passed expected-error (EE) based quality filtering, since dereplication
  is a step that precedes chimera removal and clustering in the MICCA pipeline
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: MICCA
  role: Pipeline providing the 'otu' command whose dereplication precedes centroid
    selection in de novo greedy OTU picking
  repo: https://github.com/compmetagen/micca
- name: MICCA-FAST
  role: Variant implementing exact string matching dereplication as an alternative
    to the OTUCLUST procedure
  repo: https://github.com/compmetagen/micca
- name: VSEARCH
  role: Wrapped by the MICCA 'otu' command to perform the dereplication step
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:4f44bb7665516452ed3508115225acb687894eeceb2cbf42ce49aaf797c4e3b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/compmetagen/micca
  tools:
  - MICCA
  - MICCA-FAST
  - VSEARCH
  - swarm
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# amplicon-read-dereplication

## Summary

Collapses quality-filtered amplicon reads (e.g. 16S rRNA, ITS) into unique sequences with abundance counts prior to OTU clustering, using either exact string matching (MICCA-FAST) or VSEARCH-backed dereplication as implemented in the MICCA pipeline.

## When to use

Use when quality-filtered FASTQ/FASTA amplicon reads must be collapsed to unique sequences with abundance vectors before de novo OTU inference, chimera filtering, or benchmarking dereplication speed.

## When NOT to use

- Input is already a dereplicated unique-sequence set with abundance counts
- The goal is reference-based OTU picking against a curated database rather than de novo inference
- Reads have not yet passed expected-error (EE) based quality filtering, since dereplication is a step that precedes chimera removal and clustering in the MICCA pipeline

## Inputs

- Quality-filtered amplicon reads in FASTQ or FASTA (e.g. preprocessed 16S-10 with 20% chimeric sequences, or 16S-R with 100000 reads)

## Outputs

- Dereplicated unique-sequence set with per-sequence abundance counts
- Final OTU abundance table (otu_table.csv) with per-cluster read counts
- Centroid/representative sequences per cluster (rep_seqs.fasta)
- Per-method unique-sequence counts and runtimes (dereplication_benchmark.csv)

## How to apply

Load the preprocessed reads (e.g. the 16S-10 or 16S-R datasets from ftp://ftp.fmach.it/metagenomics/micca/scirep/) and dereplicate at 100% sequence identity, counting the abundance of each unique sequence. MICCA-FAST implements this with an exact string matching algorithm, while the standard MICCA 'otu' command wraps VSEARCH for the same step; both must yield the same unique-sequence sets and abundance vectors. Dereplication precedes UCHIME-based de novo chimera removal and greedy centroid clustering that seeds clusters from the most abundant reads individuated after the dereplication step, using k-mer sorted candidate search and Needleman-Wunsch identity (rejecting reads below the threshold corresponding to m=32 mismatches). Record per-method unique-sequence counts and wall-clock runtimes when benchmarking.

## Related tools

- **MICCA** (Pipeline providing the 'otu' command whose dereplication precedes centroid selection in de novo greedy OTU picking) — https://github.com/compmetagen/micca
- **MICCA-FAST** (Variant implementing exact string matching dereplication as an alternative to the OTUCLUST procedure) — https://github.com/compmetagen/micca
- **VSEARCH** (Wrapped by the MICCA 'otu' command to perform the dereplication step)

## Evaluation signals

- Exact-matching dereplicated output is consistent with the standard VSEARCH-based output: identical unique sequences and abundance vectors
- On the 16S-10 dataset (20% chimeric sequences) the pipeline completes with chimeric reads excluded
- Estimated OTU numbers converge to approximately 77% (MICCA) of the real number of OTUs, matching the paper's reported behaviour
- Output table contains expected per-OTU abundance counts and rep_seqs.fasta contains one centroid sequence per cluster
- Wall-clock runtime of each dereplication method is recorded for comparison

## Limitations

- Dereplication is evaluated on simulated data (16S-10, ITS-10, 16S-R) with known OTU composition because no official well-studied benchmark datasets exist, making generalisation to real data uncertain
- MICCA-FAST's exact string matching dereplication converges to approximately 68% of the real number of OTUs on average, lower than standard MICCA's 77%
- Dereplicated abundance ordering drives greedy cluster seeding, so errors in the dereplication step propagate to centroid selection

## Evidence

- [results] MICCA-FAST, with the latter implementing an exact string matching dereplication algorithm: "MICCA-FAST, with the latter implementing an exact string matching dereplication algorithm"
- [discussion] Clustering seeded from post-dereplication abundant reads: "starting the clustering process from the most abundant reads individuated after the dereplication step"
- [results] MICCA and MICCA-FAST OTU convergence: "MICCA and MICCA-FAST achieve the best performances, converging to a number of estimated OTUs that are on average approximately 77% and 68% of the real number"
- [readme] Dataset deposit location: "Dataset download: ftp://ftp.fmach.it/metagenomics/micca/scirep/."
