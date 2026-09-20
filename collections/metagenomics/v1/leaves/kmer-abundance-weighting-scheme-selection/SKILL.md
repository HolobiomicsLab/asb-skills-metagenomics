---
name: kmer-abundance-weighting-scheme-selection
description: Use when preparing k-mer frequency vectors for Libra all-vs-all metagenome
  scoring and you must decide whether raw abundance, log-scaled abundance, or presence/absence
  reflects the biology you want the distance metric to capture.
when_to_use_negative:
- If the analysis goal only requires presence/absence composition and abundance resolution
  is acceptable to lose — the article notes presence/absence distance metrics greatly
  diminish resolution
- If the input is already a normalized feature/abundance table rather than raw FASTA/FASTQ
  reads requiring Libra's k-mer indexing
- If a non-composition metric such as BRAYCURTIS or JENSENSHANNON is required, since
  the weighting choice here is paired with the COSINESIMILARITY default
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: Libra
  role: Runs the k-mer scoring with the selected weighting scheme via the -w option
    during all-vs-all cosine-similarity comparison
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Execution framework that runs Libra's MapReduce-based scoring over the k-mer
    index
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:f747b4e76f3c2d0bc594a3a44e83d68ae758924a0a8f35d8e8e89727628aebbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# k-mer abundance weighting scheme selection

## Summary

Selects and applies a weighting transform (NATURAL raw k-mer frequency f, LOGARITHM 1+log f, or BOOLEAN presence/absence) to k-mer counts before Libra's cosine-similarity scoring, controlling how sequencing depth and abundance influence the resulting distance matrix.

## When to use

Use when preparing k-mer frequency vectors for Libra all-vs-all metagenome scoring and you must decide whether raw abundance, log-scaled abundance, or presence/absence best reflects the biology you want the distance metric to capture.

## When NOT to use

- If the analysis goal only requires presence/absence composition and abundance resolution is acceptable to lose — the article notes presence/absence distance metrics greatly diminish resolution
- If the input is already a normalized feature/abundance table rather than raw FASTA/FASTQ reads requiring Libra's k-mer indexing
- If a non-composition metric such as BRAYCURTIS or JENSENSHANNON is required, since the weighting choice here is paired with the COSINESIMILARITY default

## Inputs

- Libra k-mer index directory (/index_dir) produced by the preprocess step from FASTA/FASTQ files
- Choice of weighting scheme: NATURAL, LOGARITHM, or BOOLEAN (w option)

## Outputs

- All-vs-all pairwise sample similarity/distance matrix written to /score_dir

## How to apply

After building a k-mer index with Libra's preprocess step (e.g. k=20 with the NOTUNIQUE kmer filter, f option, as default), invoke the scoring command and set the w option explicitly: NATURAL for raw k-mer frequency, LOGARITHM (the default) for 1+log f to dampen dominance by highly abundant k-mers, or BOOLEAN for presence/absence. The rationale is grounded in the article's finding that presence/absence and normalized metrics greatly diminish resolution, so LOGARITHM or NATURAL weighting paired with the COSINESIMILARITY scoring algorithm (s option) preserves both genetic composition and abundance signal while normalizing for sequencing depth. Leave m=MAP (default) for the sweep-line comparison and write scores to /score_dir. Validate by re-scoring a dataset against a duplicate of itself and confirming the distance is below 0.0001.

## Related tools

- **Libra** (Runs the k-mer scoring with the selected weighting scheme via the -w option during all-vs-all cosine-similarity comparison) — https://github.com/iychoi/libra
- **Hadoop** (Execution framework that runs Libra's MapReduce-based scoring over the k-mer index)

## Examples

```
hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- A dataset scored against a duplicate of itself yields a distance below 0.0001
- The /score_dir output contains the complete all-vs-all matrix with entries for every sample pair
- The chosen w value appears in the scoring command and matches the intended scheme (LOGARITHM default vs NATURAL vs BOOLEAN)
- Switching weighting from BOOLEAN to NATURAL or LOGARITHM measurably changes pairwise distances for abundance-varying sample pairs, confirming abundance is contributing

## Limitations

- BOOLEAN presence/absence weighting greatly diminishes resolution compared with abundance-aware weightings
- Very large indexes may require falling back from the MAP run mode to REDUCE mode
- The k-mer index itself is constrained by preprocess parameters (k-mer size, filter such as NOTUNIQUE/STDDEV), which affect the counts being weighted

## Evidence

- [abstract] Distance calculation using Cosine Similarity on sequence composition and abundance, normalized for sequencing depth: "Cosine Similarity for calculating the distance using sequence composition and abundance while normalizing for sequencing depth"
- [readme] Weighting option in scoring: "w : weighting algorithm. LOGARITHM (default) | BOOLEAN | NATURAL"
- [methods] Methods scoring option list: "s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [abstract] Resolution loss caveat: "methods commonly implemented to reduce compute time for large datasets, such as data reduction, read count normalization, and presence/absence distance metrics, greatly diminish the resolution"
