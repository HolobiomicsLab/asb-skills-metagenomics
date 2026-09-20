---
name: large-scale-benchmark-runtime-profiling
description: Use when profiling or validating the scalability of Libra's Hadoop-based
  all-vs-all k-mer comparison on billion-read metagenome datasets, or when reporting
  runtime on a multi-node Hadoop cluster against the paper's ~1.5 h / ~18 h Table
  1 benchmarks.
when_to_use_negative:
- Datasets small enough for sketch-based tools (e.g. Mash/Sourmash), where the ~18
  h Hadoop benchmark profile is irrelevant overhead
- Analyses where data reduction or read-count normalization is acceptable — the article
  states these 'greatly diminish the resolution' and Libra's runtime is the price
  of using all reads
- Tasks whose goal is biological clustering rather than performance measurement —
  in that case reproduce Fig. 5 directly and treat timings as secondary validation
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Libra
  role: Tool whose two Hadoop stages (preprocess/index and core/scoring) are timed
    in this benchmark; run via 'hadoop jar libra-all.jar'
  repo: iychoi/libra
- name: Hadoop
  role: MapReduce framework on which the scalable k-mer indexing and all-vs-all scoring
    stages execute; cluster size (10 nodes) defines the benchmark configuration
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:c10a9fddda2a116bcfd73dcc01fb8d8e3055abd1c4c8d3efa930ca086dd00a45
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# large-scale-benchmark-runtime-profiling

## Summary

Benchmark the wall-clock runtime of Libra's two-stage Hadoop MapReduce pipeline (k-mer indexing + scoring) on massive metagenome collections, using the 43 Tara Ocean Viromes (~4.2 billion reads) all-vs-all comparison as the reference workload and Table 1's reported timings as the performance target.

## When to use

Use when profiling or validating the scalability of Libra's Hadoop-based all-vs-all k-mer comparison on billion-read metagenome datasets, or when reporting runtime on a multi-node Hadoop cluster against the paper's ~1.5 h / ~18 h Table 1 benchmarks.

## When NOT to use

- Datasets small enough for sketch-based tools (e.g. Mash/Sourmash), where the ~18 h Hadoop benchmark profile is irrelevant overhead
- Analyses where data reduction or read-count normalization is acceptable — the article states these 'greatly diminish the resolution' and Libra's runtime is the price of using all reads
- Tasks whose goal is biological clustering rather than performance measurement — in that case reproduce Fig. 5 directly and treat timings as secondary validation

## Inputs

- Quality-controlled FASTA/FASTQ sequencing reads (43 Tara Ocean Viromes, ~4.2 billion reads, ENA 2009-2011 Expedition, Brum et al. QC protocol)
- A Hadoop cluster (reference configuration: 10 nodes)
- Libra index directory (/index_dir) for the scoring-stage benchmark

## Outputs

- K-mer index directory (/index_dir) from the HM indexing stage
- All-vs-all cosine-similarity score directory (/score_dir) for the 43-virome matrix
- Per-stage and end-to-end runtime profile (k-mer matching time, total comparison time) comparable to Table 1

## How to apply

Run the benchmark on a 10-node Hadoop cluster with the QC'd 43 Tara Ocean Viromes (~4.2 billion reads, ENA 2009-2011 Expedition, filtered per Brum et al.) as input. First run the indexing stage (README 'preprocess') with k=20 and parallel tasks (-t 8) to build the k-mer index in /index_dir, then run the scoring stage ('core') in MAP run mode with COSINESIMILARITY and LOGARITHM weighting (defaults) writing to /score_dir. Time each stage separately and end-to-end, then compare against the reported profile: k-mer matching within ~1.5 hours and the full all-vs-all comparison within ~18 hours on a 10-node Hadoop cluster (Table 1). Rationale: Libra deliberately uses all reads (no sketching or data reduction) so its scalability claim is only credible if the full-workload runtime is measured, not extrapolated from subsampled data. Group-size parameters (-s 10GB default, -g 20 groups max) and reducer count (-t) are the knobs that control index parallelism and thus per-stage timing.

## Related tools

- **Libra** (Tool whose two Hadoop stages (preprocess/index and core/scoring) are timed in this benchmark; run via 'hadoop jar libra-all.jar') — iychoi/libra
- **Hadoop** (MapReduce framework on which the scalable k-mer indexing and all-vs-all scoring stages execute; cluster size (10 nodes) defines the benchmark configuration)

## Examples

```
hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- K-mer matching (indexing) stage completes within ~1.5 hours on the 10-node Hadoop cluster, matching Table 1
- End-to-end all-vs-all comparison of the 43 TOVs completes within ~18 hours, matching Table 1
- All 43 x 43 pairwise cosine-similarity scores are produced in /score_dir with no missing pairs
- Clustering the resulting distance matrix reproduces the four temperature-gradient groupings of the 43 viromes (Fig. 5)
- Index stage settings stay within documented defaults (k=20, group size -s 10GB, max -g 20 groups, -t reducers set explicitly for parallelism)

## Limitations

- Runtime targets are tied to the paper's specific configuration (10-node Hadoop cluster, 43 viromes, ~4.2 billion reads); other cluster sizes or datasets will not map directly to the ~1.5 h / ~18 h numbers
- The README defaults to 1 reducer (-t 1), so naive runs under-parallelize and will miss the reported profile unless -t is raised (e.g. -t 8)
- Index grouping is capped by -g (20 groups default); datasets generating more than 20 file groups are combined, which can shift stage timings

## Evidence

- [abstract] Scalability of Libra's all-reads design: "Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset to enable global-scale analyses"
- [intro] Benchmark workload definition: "we show that Libra can scale to massive global-scale datasets by examining viral diversity in 43 Tara Ocean Viromes (TOVs)"
- [readme] Concrete benchmark invocation from README: "hadoop jar libra-all.jar preprocess -k 20 -t 8 -o /index_dir /source_dir"
- [readme] Scoring-stage options to fix before timing: "s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
