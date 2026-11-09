---
name: cosine-similarity-metagenome-distance-computation
description: Use when an all-vs-all distance matrix must be computed across many metagenome
  samples from full k-mer abundance vectors (no sketching or presence/absence reduction),
  and scale demands a single-pass, MapReduce-based sweep over an inverted k-mer index.
when_to_use_negative:
- Input is a pre-computed distance matrix or feature table rather than raw metagenome
  reads/indexed k-mer vectors
- The analysis requires a different distance metric explicitly supported by the paper's
  contrast (e.g., Bray-Curtis or Jensen-Shannon were Libra's alternative scoring algorithms,
  not the cosine sweep being reconstructed)
- Data reduction, read count normalization, or presence/absence distance metrics are
  acceptable for the analysis — the article states these greatly diminish resolution
  for large-scale comparisons
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Libra
  role: Performs the all-vs-all metagenome comparison by cosine similarity over k-mer
    content, sweeping the inverted index to accumulate D and M terms
  repo: iychoi/libra
- name: Hadoop
  role: Scalable MapReduce framework in which Libra's MAP-mode sweep of k-mer vectors
    is executed for massive datasets
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:31df8feca73ec09f7d57b540c34abf09a958e92742b08652cfdc4cb6c7920ec9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cosine-similarity-metagenome-distance-computation

## Summary

Reconstructs Libra's all-vs-all pairwise metagenome distance computation, which uses a sweep-line algorithm over an inverted k-mer index to accumulate cosine similarity terms (dot products D and per-sample magnitudes M) for every sample pair in a single scan rather than scanning each vector pair separately.

## When to use

Use when an all-vs-all distance matrix must be computed across many metagenome samples from full k-mer abundance vectors (no sketching or presence/absence reduction), and scale demands a single-pass, MapReduce-based sweep over an inverted k-mer index.

## When NOT to use

- Input is a pre-computed distance matrix or feature table rather than raw metagenome reads/indexed k-mer vectors
- The analysis requires a different distance metric explicitly supported by the paper's contrast (e.g., Bray-Curtis or Jensen-Shannon were Libra's alternative scoring algorithms, not the cosine sweep being reconstructed)
- Data reduction, read count normalization, or presence/absence distance metrics are acceptable for the analysis — the article states these greatly diminish resolution for large-scale comparisons

## Inputs

- Libra k-mer inverted index directory (/index_dir) produced by the preprocess step from FASTA/FASTQ files
- Scoring options: s=COSINESIMILARITY, m=MAP|REDUCE, w=LOGARITHM|BOOLEAN|NATURAL, output directory

## Outputs

- All-vs-all cosine distance matrix written to /score_dir

## How to apply

Build (or load from the upstream HM preprocess step) an inverted index keyed by k-mer that stores each sample's k-mer frequency vector under /index_dir. Run Libra's scoring command `hadoop jar libra-all.jar core -o /score_dir /index_dir` with s=COSINESIMILARITY (the default scoring algorithm) to launch the all-vs-all comparison. Select a weighting scheme via the w option — LOGARITHM (w=1+log f, the default), NATURAL (raw k-mer frequency), or BOOLEAN — noting that the article warns presence/absence metrics diminish resolution. Execute in Hadoop MAP mode (m=MAP, default), which sweeps k-mers once and accumulates the dot-product term D and per-sample magnitude terms M for every sample pair in linear time; fall back to m=REDUCE for large indexes. The scoring step writes the all-vs-all cosine distance matrix into /score_dir.

## Related tools

- **Libra** (Performs the all-vs-all metagenome comparison by cosine similarity over k-mer content, sweeping the inverted index to accumulate D and M terms) — iychoi/libra
- **Hadoop** (Scalable MapReduce framework in which Libra's MAP-mode sweep of k-mer vectors is executed for massive datasets)

## Examples

```
hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- Sanity check on a small test dataset: a sample compared against a duplicate of itself yields a distance below 0.0001
- The output /score_dir contains a complete all-vs-all matrix (n x n) matching the number of samples in the index
- Diagonal of the distance matrix is ~0 for all samples (self-comparison)
- The run used the default m=MAP mode; if index size forced m=REDUCE, confirm the sweep completed with equivalent results
- Re-running with w=LOGARITHM (default) versus w=BOOLEAN should produce distinguishable clustering given the paper's finding that presence/absence metrics reduce resolution

## Limitations

- Sweep requires the full k-mer inverted index; very large indexes may require falling back from MAP mode to REDUCE mode
- Results depend on the preprocessing k-mer size (e.g., -k 20) and the k-mer filter (NONE | STDDEV | STDDEV2 | NOTUNIQUE) chosen upstream
- Cosine Similarity normalizes for sequencing depth and considers both composition and abundance, but the paper validates it on specific benchmarks (HMP 48 samples, CAMI toy PacBio dataset, 43 Tara Ocean Viromes); behavior outside these regimes is uncharacterized in this context

## Evidence

- [abstract] We developed a tool called Libra that performs an all-vs-all comparison of metagenomes for precise clustering based on their k-mer content.: "We developed a tool called Libra that performs an all-vs-all comparison of metagenomes for precise clustering based on their k-mer content."
- [abstract] Cosine Similarity for calculating the distance using sequence composition and abundance while normalizing for sequencing depth: "Cosine Similarity for calculating the distance using sequence composition and abundance while normalizing for sequencing depth"
- [methods] Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON: "Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir: [evidence span withheld — anchor status: external_doc]
- [readme] s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON - m : run mode. MAP (default) | REDUCE - w : weighting algorithm. LOGARITHM (default) | BOOLEAN | NATURAL: "s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON
- m : run mode. MAP (default) | REDUCE
- w : weighting algorithm. LOGARITHM (default) | BOOLEAN | NATURAL"
