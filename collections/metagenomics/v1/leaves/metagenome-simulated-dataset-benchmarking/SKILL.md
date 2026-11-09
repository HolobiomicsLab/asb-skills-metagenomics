---
name: metagenome-simulated-dataset-benchmarking
description: Use when validating a metagenome distance metric on simulated 454/PacBio
  mock-community datasets that vary sequencing depth, dominant-organism abundance,
  or genetic composition, to compare logarithmic vs natural k-mer weighting against
  Mash and Simka.
when_to_use_negative:
- Input is real environmental data where ground-truth community composition is unknown
  — the stepwise-resolution expectation holds only for simulated mock communities
  with known composition differences
- The goal is raw clustering rather than metric benchmarking — the duplication/depth
  sensitivity checks are benchmark diagnostics, not an analysis endpoint
- Datasets are so large that simulation-based validation is unnecessary; the article's
  motivation is that Libra itself scales via Hadoop MapReduce to massive datasets
  like Tara Ocean Viromes
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Libra
  role: 'Primary tool: builds the k-mer count index via Hadoop MapReduce preprocess
    and produces all-vs-all cosine-similarity distance matrices under NATURAL and
    LOGARITHM weighting'
  repo: https://github.com/iychoi/libra
- name: Mash
  role: 'Benchmark comparator: MinHash-based unique k-mer sketches used to show distance
    compression on the mock 1–4 composition series'
- name: Simka
  role: 'Benchmark comparator: computes Jaccard-ab and Bray-Curtis distance matrices
    that saturate between mocks 3 and 4 and stay flat across the depth series'
- name: Hadoop
  role: Scalable framework underlying Libra's preprocessing and scoring runs (hadoop
    jar libra-all.jar ...)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:49d5b67c58881104c33dc6501aa36a0b6b11677b973991281f140d393dbbdfef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  - Mash
  - Simka
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# metagenome-simulated-dataset-benchmarking

## Summary

Benchmark all-vs-all metagenome comparison metrics (Libra, Mash, Simka) on simulated metagenomes derived from a staggered mock community (HM-277D, mock 1–4) that vary in depth, dominance, and genetic composition, to test whether abundance-aware k-mer distances resolve communities better than MinHash or presence/absence metrics.

## When to use

Use when validating a metagenome distance metric on simulated 454/PacBio mock-community datasets that vary sequencing depth, dominant-organism abundance, or genetic composition, to compare logarithmic vs natural k-mer weighting against Mash and Simka.

## When NOT to use

- Input is real environmental data where ground-truth community composition is unknown — the stepwise-resolution expectation holds only for simulated mock communities with known composition differences
- The goal is raw clustering rather than metric benchmarking — the duplication/depth sensitivity checks are benchmark diagnostics, not an analysis endpoint
- Datasets are so large that simulation-based validation is unnecessary; the article's motivation is that Libra itself scales via Hadoop MapReduce to massive datasets like Tara Ocean Viromes

## Inputs

- Simulated 454 metagenomic read datasets (FASTA/FASTQ) derived from the staggered HM-277D mock community (mock 1–4 series)
- Simulated metagenome datasets varying in dataset size and sequencing depth (e.g. 0.5/1/5/10 million reads)
- Libra k-mer count index directory produced by the preprocess command

## Outputs

- Libra all-vs-all pairwise distance matrices under NATURAL and LOGARITHM weighting (COSINESIMILARITY scoring)
- Mash MinHash-based pairwise distance matrix
- Simka pairwise distance matrices using Jaccard-ab and Bray-Curtis metrics
- Qualitative benchmark result: stepwise distance resolution vs distance compression/saturation across mock 1–4

## How to apply

Build the Libra k-mer count index from simulated 454 mock-community FASTQ/FASTA reads with the preprocess command (e.g. k=20), then run scoring twice on the same index: once with the default COSINESIMILARITY algorithm and NATURAL weighting (-w NATURAL) to obtain raw-abundance distances, and once with LOGARITHM weighting (-w LOGARITHM) to obtain logarithmically weighted distances. In parallel, run Mash on the same samples for MinHash-based unique k-mer distances and Simka for Jaccard-ab and Bray-Curtis matrices. Confirm the expected qualitative pattern: Libra with LOGARITHM weighting resolves mock 1–4 with a stepwise increase in pairwise distance, while Mash distances compress and Simka distances, along with Libra NATURAL weighting, saturate at maximum difference between mocks 3 and 4. Also verify no measurable distance change under read duplication and flat Simka distances across the 0.5/1/5/10 M read-depth series, since compression or saturation indicates loss of resolution.

## Related tools

- **Libra** (Primary tool: builds the k-mer count index via Hadoop MapReduce preprocess and produces all-vs-all cosine-similarity distance matrices under NATURAL and LOGARITHM weighting) — https://github.com/iychoi/libra
- **Mash** (Benchmark comparator: MinHash-based unique k-mer sketches used to show distance compression on the mock 1–4 composition series)
- **Simka** (Benchmark comparator: computes Jaccard-ab and Bray-Curtis distance matrices that saturate between mocks 3 and 4 and stay flat across the depth series)
- **Hadoop** (Scalable framework underlying Libra's preprocessing and scoring runs (hadoop jar libra-all.jar ...))

## Examples

```
hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- Libra with LOGARITHM weighting yields a monotonically stepwise increase in pairwise distance across mock1 → mock4, whereas NATURAL weighting and Simka Bray-Curtis/Jaccard hit maximum difference already between mock3 and mock4
- Mash pairwise distances compress rather than resolve the mock 1–4 composition series
- Duplicating reads produces no measurable distance change (depth-invariance check)
- Simka distances remain flat across the 0.5/1/5/10 million read depth series
- The scoring output is a complete all-vs-all distance matrix in the designated score directory with COSINESIMILARITY as the default algorithm

## Limitations

- Presence/absence and normalized metrics (Mash, Simka Jaccard, Bray-Curtis) diminish resolution, compressing or saturating distances on composition series
- Validation patterns are demonstrated on simulated 454 and CAMI 'toy' PacBio metagenomes; transferability to real wild communities requires additional datasets (HMP, Tara Ocean Viromes)
- Libra requires a Hadoop environment and Java; k-mer size, task count, and k-mer filter (NONE | STDDEV | STDDEV2 | NOTUNIQUE) choices affect index construction

## Evidence

- [intro] We validate this distance metric using simulated metagenomes (from both short- and long-read technologies) to show that Libra has exceptional sensitivity in distinguishing complex mixed microbiomes.: "We validate this distance metric using simulated metagenomes (from both short- and long-read technologies) to show that Libra has exceptional sensitivity"
- [abstract] Data reduction diminishes resolution.: "methods commonly implemented to reduce compute time for large datasets, such as data reduction, read count normalization, and presence/absence distance metrics"
- [abstract] Libra all-vs-all k-mer comparison.: "performs an all-vs-all comparison of metagenomes for precise clustering based on their k-mer content"
- [readme] Scoring algorithm options in README.: "s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] Weighting algorithm options in README.: "w : weighting algorithm. LOGARITHM (default) | BOOLEAN | NATURAL"
